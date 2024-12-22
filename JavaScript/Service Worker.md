Service Worker 是一种运行在 Web 应用后台的脚本，可以拦截和控制网络请求，充当 Web 应用与浏览器网络的代理服务器，可以实现 Web 应用离线访问、资源预载、智能缓存、资源更新、后台同步和推送通知等。这些特性使 Web 应用即便在网络不稳定或离线状态下，也能提供更快速和可靠的用户体验

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732772396208-be38024a-d9b2-4180-986f-3574a464a596.png)

使用 Service Worker 首先需要在页面主 JavaScript 文件中，通过 navigator.serviceWorker.register 方法来注册 Service Worker，通常这个步骤在页面加载时候进行

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
      .then(registration => {
        console.log('Service Worker registered with scope:', registration.scope);
      }).catch(error => {
        console.error('Service Worker registration failed:', error);
      });
  });
}
```

接下来就可以创建 `service-worker.js` 文件，在文件中处理后续安装、激活以及 fetch 事件等步骤

## Service Worker 生命周期
Service Worker 的生命周期由安装、激活、控制、更新等多个阶段组成

### 安装阶段
当浏览器检测到 Service Worker 要求注册新版本时，会进入安装流程，并触发`install`事件

+ 事件处理程序中可以使用 event.waitUntil() 方法来确保在安装过程中完成异步任务（如资源缓存）
+ 只有在 event.waitUntil() 中的所有任务都成功完成后，Service Worker 的安装才会成功

```javascript
self.addEventListener('install', event => {
  console.log('Service Worker installing...');
  event.waitUntil(
    caches.open('v1').then(cache => {
      console.log('Caching resources during install');
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/script.js'
      ]);
    })
  );
});
```

> `self` 在 Service Worker 中是一个全局对象，类似于在浏览器主线程中的 window 对象，用于指代 Service Worker 自身
>

### 激活阶段
安装完成后进入激活阶段，并触发`activate`事件，主要用于设置新版本的 Service Worker，当一个新的 Service Worker 成功激活时，旧版本的 Service Worker 将被停止，新的版本将接管控制权，新激活的 Service Worker 可以在 `activate` 事件中执行清理操作，删除不再需要的旧缓存数据

```javascript
self.addEventListener('activate', event => {
  // 定义一个缓存白名单，其中包含需要保留的缓存版本名
  const cacheWhitelist = ['v1'];

  // 在激活阶段，使用 waitUntil() 阻止 Service Worker 激活完成，直到内部的 Promise 被解决
  event.waitUntil(
    // 获取所有缓存的名称（即现有的缓存版本）
    caches.keys().then(cacheNames => {
      // 使用 Promise.all() 处理可能要删除的多个缓存
      return Promise.all(
        // 遍历所有的缓存名称
        cacheNames.map(cacheName => {
          // 检查每个缓存名称是否在白名单中
          if (!cacheWhitelist.includes(cacheName)) {
            // 如果不在白名单中，删除这个缓存
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

### 控制阶段
一旦激活完成 Service Worker 就会进入控制阶段，服务所在作用域内的所有页面，这时它可以拦截网络请求进行处理，比如处理 `fetch` 事件，决定资源从网络还是缓存中获取

`event.respondWith` 用于拦截和处理浏览器请求。它允许开发者控制请求的响应，提供自定义响应结果，通常用于返回缓存的资源、重定向请求、或根据特定条件选择不同的响应方案

```javascript
self.addEventListener('fetch', event => {
  // 使用事件对象的 respondWith() 方法以自定义响应
  event.respondWith(
    // 尝试从缓存中匹配请求
    caches.match(event.request).then(response => {
      // 缓存中有匹配的响应则返回，否则进行网络请求
      return response || fetch(event.request).then(networkResponse => {
        // 打开缓存进行存储
        return caches.open('v1').then(cache => {
          // 将请求 URL 与网络响应一同缓存
          // 使用 clone() 是因为 response 是 Stream，只能消费一次
          cache.put(event.request, networkResponse.clone());
          // 返回网络响应给页面使用
          return networkResponse;
        });
      });
    })
  );
});
```

### 更新阶段
Service Worker 的更新阶段负责在检测到新的 Service Worker 时，替换旧版本并在合适的时机进行激活<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);"></font>

1. **检查更新**：<font style="color:rgb(51, 51, 51);">每次页面加载且在调用 </font>`<font style="color:rgb(51, 51, 51);">navigator.serviceWorker.register()</font>`<font style="color:rgb(51, 51, 51);"> 时，浏览器会自动检查 Service Worker 文件的变更。如果检测到文件内容发生了变化，浏览器会认为这是一个新版本，并开始下载</font>
2. **下载新版本**：<font style="color:rgb(51, 51, 51);">新的 Service Worker 文件被下载到浏览器中，下载成功后新的 Service Worker 会进入到“等待中”（Waiting）状态</font>
3. **等待激活**：<font style="color:rgb(51, 51, 51);">默认情况下新版本 Service Worker 不会立即取代正在使用的旧版本，它会在不影响当前运行状态的前提下，等到所有旧版本的页面和控制范围内的客户端关闭后才激活</font>
4. **激活新版本**：<font style="color:rgb(51, 51, 51);">新版本进入激活阶段后，被设置为活跃的控制者，旧版本的 Service Worker 则会被终止，进入激活阶段的 Service Worker 可以通过 </font>`<font style="color:rgb(51, 51, 51);">activate</font>`<font style="color:rgb(51, 51, 51);"> 事件为新控制的范围做清理旧缓存等准备工作</font>

<font style="color:rgb(51, 51, 51);"></font>

在更新阶段，为了确保新版本的 Service Worker 快速生效，开发者通常会使用 `skipWaiting()` 和 `clients.claim()`跳过等待

```javascript
self.addEventListener('install', event => {
  console.log('New Service Worker installing...');
  // 强制等待中的 Service Worker 立即生效
  self.skipWaiting();
});

self.addEventListener('activate', event => {
  console.log('Service Worker activating...');
  // 立即接管所有客户端
  event.waitUntil(
    clients.claim()
  );
});
```

## Service Worker 和主线程通信
### postMessage
主线程

```javascript
// 注册 Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js').then(async registration => {
    // 等待 Service Worker 准备好
    const readySW = await navigator.serviceWorker.ready;
    const activeWorker = readySW.active;

      // 发送消息到 Service Worker
      activeWorker.postMessage({ 
        type: 'INIT', 
        message: 'Hello from main thread!' 
      });

      // 监听来自 Service Worker 的消息
      navigator.serviceWorker.addEventListener('message', event => {
        console.log('Received message from Service Worker:', event.data);
      });
  });
}
```

service-worker.js

```javascript
// 监听来自主线程的消息
self.addEventListener('message', event => {
  console.log('Received message from main thread:', event.data);

  // 根据接收到的消息处理业务逻辑
  if (event.data && event.data.type === 'INIT') {
    // 执行一些逻辑，例如更新缓存、初始化状态等

    // 回复消息回主线程
    event.source.postMessage({ type: 'REPLY', message: 'Hello from Service Worker!' });
  }
});
```

### MessageChannel
主线程

```javascript
// 注册 Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js').then(async registration => {
    console.log('Service Worker registered with scope:', registration.scope);

    // 确保 Service Worker 准备好
    const readySW = await navigator.serviceWorker.ready;

    // 创建一个 MessageChannel 实例
    const messageChannel = new MessageChannel();

    // 监听 port1 收到的消息
    messageChannel.port1.onmessage = event => {
      console.log('Received from Service Worker:', event.data);
    };

    // 发送一个端口到 Service Worker
    if (readySW.active) {
      readySW.active.postMessage({ type: 'INIT' }, [messageChannel.port2]);
    }

    // 在本地通过 port1 发送消息
    messageChannel.port1.postMessage('Hello, Service Worker!');
  }).catch(error => {
    console.error('Service Worker registration failed:', error);
  });
}

```

service-worker.js

```javascript
self.addEventListener('message', event => {
  // 通过传递的端口进行通信
  const port = event.data.port || event.ports[0];

  if (event.data && event.data.type === 'INIT' && port) {
    // 监听在这个端口上收到的消息
    port.onmessage = event => {
      console.log('Received from main thread:', event.data);

      // 回复消息回主线程
      port.postMessage('Hello, main thread!');
    };

    // 可以继续通过 port 发送消息
    port.postMessage('Service Worker is ready to communicate!');
  }
});
```

## Service Worker 存储机制
除了 IndexDB 在 Service Worker 中可以使用 Service Worker 提供的一种 Cache Storage 异步存储机制，用于缓存请求和响应对象。相对于传统的浏览器缓存 Cache Storage 对开发者更友好，因为它提供了完善的 API 来管理缓存内容

### 添加到缓存
`caches.open()` 方法用于在 Cache Storage 中打开一个缓存对象，并返回一个 Promise，该 Promise 解析为一个 Cache 对象，对 Cache 操作首先要打开缓存

**cache.put**

```javascript
self.addEventListener('fetch', event => {
  // 首先尝试通过网络请求获取资源
  event.respondWith(
    fetch(event.request).then(networkResponse => {
      // 打开名为 'dynamic-cache' 的缓存
      return caches.open('dynamic-cache').then(cache => {
        // 确保响应是成功的才能放入缓存
        cache.put(event.request, networkResponse.clone());
        return networkResponse;
      });
    })
  );
});
```

**chache.addAll**

`cache.addAll` 用于将一批请求添加到缓存中，这个方法简单高效地处理多个请求，减少了开发者手动获取和存储每个响应的繁琐

```javascript
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('my-cache').then(cache => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/script.js',
        '/image.png'
      ]);
    })
  );
});
```

### 从缓存中读取
`cache.match` 方法用于在特定的 Cache 对象中查找与指定请求匹配的响应。

```javascript
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.open('my-cache').then(cache => {
      return cache.match(event.request).then(cachedResponse => {
        // 如果有缓存响应，返回它；否则继续通过网络请求获取
        if (cachedResponse) {
          return cachedResponse;
        }

        // 如果没有缓存响应，执行网络请求
        return fetch(event.request).then(networkResponse => {
          // 将新获取的响应对象存储到缓存中
          if (networkResponse && networkResponse.status === 200) {
            cache.put(event.request, networkResponse.clone());
          }
          return networkResponse;
        });
      });
    })
  );
});
```

### 更新缓存
```javascript
caches.open('my-cache').then(cache => {
  cache.put('/dynamic-content', new Response('Updated content'));
});
```

### 删除缓存
```javascript
caches.keys().then(cacheNames => {
  return Promise.all(
    cacheNames.filter(cacheName => {
      return cacheName !== 'my-cache';
    }).map(cacheName => {
      return caches.delete(cacheName);
    })
  );
});
```

## Service Worker 使用限制
### 必须在HTTPS环境下工作
Service Worker 可以拦截和修改网络请求，会涉及敏感数据的传输和处理。为了防止中间人攻击和其他安全威胁，浏览器要求 Service Worker 只能在 HTTPS 环境下注册和运行

```javascript
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js')
    .then(function(registration) {
      console.log('Registration done with scope: ', registration.scope);
    })
    .catch(function(error) {
      console.log('Registration failed:', error);
    });
}
```

### 只能控制与脚本同域的请求
Service Worker 的作用域是基于其注册脚本的位置来决定的。这意味着Service Worker只能控制与其注册脚本在相同路径或子路径下的页面和请求。

+ Service Worker 注册在 `/sw.js`，则只能控制 `/` 和其子路径下的页面
+ 如果注册在 `/blog/sw.js`，则只能控制 `/blog/` 路径及其子路径下的页面

### 无法访问 DOM、window 对象
Service Worker 运行在独立的 Worker 线程中，不直接访问 DOM、window 对象。这是为了隔离服务工作线程与主页面线程，增强安全性和性能，通过 Message Channel（消息通道）与主页面通信，发送和接收数据更新页面。

```javascript
// 在Service Worker中
self.addEventListener('message', event => {
  console.log('Received message:', event.data);
  event.ports[0].postMessage('Got your message!');
});

// 在主页面中
navigator.serviceWorker.ready.then(registration => {
  const messageChannel = new MessageChannel();
  messageChannel.port1.onmessage = event => {
    console.log('Received reply:', event.data);
  };
  navigator.serviceWorker.controller.postMessage('Hello, Service Worker!', [messageChannel.port2]);
});
```

