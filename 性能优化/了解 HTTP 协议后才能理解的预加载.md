在性能优化过程中，开发者通常会集中精力在以下几个方面：服务器响应时间（RT）优化、服务端渲染（SSR）与客户端渲染优化、以及静态资源体积的减少。然而，对于许多用户进入网站的第一个页面（如首页），网络开销也是一个不容忽视的问题

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724490742126-03eb2423-c84c-458f-97f2-c3205f2aae36.png)

由于新用户可能从未与网站建立连接，从DNS查询到TCP连接，再到下载服务器返回的内容，这些步骤的耗时通常远远超过服务器的响应时间。而多数情况下开发者无法通过代码优化来减少这部分时间消耗。

为了解决新用户访问网站时可能遇到的网络开销问题，我们可以借助多种预加载技术在用户实际需要之前提前加载资源，从而减少等待时间，实现更流畅的用户体验。接下来本文将详细探讨几种常见的预加载方法，并在 `prefetch`、`preload`等基础上，结合流式渲染、HTTP Early Hints、HTTP/2 push 等技术，对预加载技术灵活运用，从而在用户到达网站的瞬间就提供无缝、快速的访问体验

## CDN 动态加速
在开始介绍预加载之前，其实开发者可以通过 CDN 动态加速优化用户与服务器的建连、内容传输时间。CDN 通常被用来加速静态资源的传输，比如图像、JavaScript 和 CSS，这个大部分开发者非常熟悉，但现代的 CDN 技术已经不仅仅局限于静态内容的优化，大部分 CDN 厂商可以利用其全球广泛分布的边缘节点服务器为网站提供动态内容的访问加速

用户访问网站动态内容需要通过互联网连接到源站服务器，这个过程中数据需要经过多个网络节点和长距离传输，容易受到各种网络拥塞和延迟的影响

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724494242380-4245f356-710a-41b5-bab7-d0b0eac32db6.png)



使用 CDN 动态加速时，CDN 通过在全球分布的边缘节点缓存和处理用户请求，显著缩短了从用户到服务器的物理距离，减少了传输延迟。同时 CDN 服务商会实时监控全球的网络状态，通过智能路由技术选择当前最优的路径传输数据，这避免了网络中的拥塞和瓶颈，确保数据以最快的速度传输到用户端

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724494491161-0ee0a0fc-97b3-47b2-82c4-3ae4217f0631.png)



当然如果使用了 CDN 提供的边缘计算能力，可以让用户直接从 CDN 边缘节点获取动态内容，进一步加速动态内容的访问

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724492520788-90eecfbc-e4b8-48ab-9da8-c42c6f346c51.png)

## dns-prefetch DNS 预解析
当浏览器需要访问特定域名时，必须先将先将域名解析为 IP 地址，这一步骤就是 DNS 解析。`dns -prefetch` 可以让浏览器提前在后台完成这一解析工作，避免用户在实际请求资源时等待 DNS 解析的时间

在 HTML 顶部通过`<link>`标签来指示浏览器对接下来要是用的静态资源、动态接口等域名提前进行 DNS 解析

```html
<link rel="dns-prefetch" href="//example.com">
```

## preconenct 域名预建连
当浏览器解析了域名后，接下来需要通过TCP协议和服务器建立连接，并在使用 HTTPS 的情况下进行 TLS 握手，这些步骤通常需要较多往返时间（RTT）。`preconnect` 通过提前完成这些连接步骤，可以减少用户真正需要请求资源时的等待时间。

可以在HTML中通过`<link>`标签来指示浏览器进行预连接，使用 preconnect 之后浏览器不仅会解析域名的 DNS，还会提前与服务器建立 TCP 连接，并完成 TLS 握手

```html
<link rel="preconnect" href="//example.com">
```

## preload 与 prefetch 预加载
除了对域名进行解析、建连，还可以通过 preload 和 prefetch 对页面将要使用的资源提前下载

`preload` 是一种声明式资源引入方式，用来强制浏览器在合适的时机加载指定资源，通常用于关键资源（如字体、脚本、样式表等）的预加载，以确保这些资源能够尽快被使用

```html
<link rel="preload" href="styles.css" as="style">
<link rel="preload" href="main.js" as="script">
<link rel="preload" href="image.jpg" as="image">
```

`prefetch` 同样是一种声明式资源请求方式，用于提示浏览器在空闲时下载未来可能用到的资源，适合作为页面未来使用的资源或者当前页面下一跳页面要使用的资源预加载

```html
<link rel="prefetch" href="next-page-image.jpg">
<link rel="prefetch" href="next-page-script.js">
```

两个标签在优先级上有一定的区别

+ `preload`：具有高优先级，浏览器会立即加载这些资源
+ `prefetch`：具有较低优先级，只有在浏览器空闲时才会加载这些资源，确保不妨碍当前页面的正常加载

两者在浏览器支持上各有千秋

| preload | prefetch |
| --- | --- |
| ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724494590902-fabeba38-a49f-494a-95ca-50c7d63dbfa4.png) | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724494616592-c9b12fde-9bd4-404e-a8e2-2d7d0ae1be9f.png) |


## prerender 预渲染
使用`prerender` 可以将目标页面上近乎所有资源（HTML、CSS、JavaScript、图像等）和内容在后台提前下载并渲染，浏览器在用户首次访问该页面之前已经完全准备好了该页面的视图。这样当用户跳转到该页面时，使用户在实际跳转到这个页面时能够立即呈现，不需要再等待加载和渲染的时间

```html
<link rel="prerender" href="https://example.com/next-page">
```

听起来 prerender 是预加载的终极方案了，但在实际性能优化方案中却很少被使用，使用 preload 有几个弊端

+ **不能命中时候资源开销过大**：因为 prerender 会对页面进行资源下载和渲染，当页面没有被用户访问时候造成的资源浪费过大
+ **影响页面数据统计**：大部分页面在执行时候会对页面进行数据上报用作后续的页面效果分析，部分页面会有展示广告等行为，如果 prerender 后用户没有访问页面，会造成数据统计上的混乱
+ **浏览器兼容性问题**：不同的浏览器对于 prerender 的实现细节可能有所不同。例如，一些浏览器可能出于性能或安全考虑，会对预渲染的资源类型进行某些限制

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724496112205-9eb6b70f-a1e5-4a15-9df3-29ab690b8ee3.png)

## 根据用户行为 prefetch 下一跳页面
无脑对页面进行 prefetch 会造成巨大的资源浪费，但很多时候我们可以根据用户行为更精准的预测用户接下来的动作，再进行 prefetch 可以很大程度上减少资源浪费

举个例子，在 PC 页面当用户鼠标悬停在某个商品图片上时候，我们可以大胆预测用户及大概率要点击页面，这时候可以对页面进行 prefetch。如果希望进一步细化，用户点击鼠标的动作会依次触发 `mousedown`、`mouseup`、`click`事件，我们可以在 mousedown 事件中对页面进行预载，这样可以节省人点击鼠标的 200ms 左右

```jsx
function App() {
  return (
    <div className="App">
      <h1>Product List</h1>
      <div className="product-list">
        <Product id="1" name="Product 1" imageUrl="https://via.placeholder.com/150" prefetchUrl="/next-page-1" />
        <Product id="2" name="Product 2" imageUrl="https://via.placeholder.com/150" prefetchUrl="/next-page-2" />
        <Product id="3" name="Product 3" imageUrl="https://via.placeholder.com/150" prefetchUrl="/next-page-3" />
      </div>
    </div>
  );
}

const Product = ({ id, name, imageUrl, prefetchUrl, delay=200 }) => {
  const [prefetchTimeout, setPrefetchTimeout] = useState(null);

  const handleMouseOver = () => {
    const timeout = setTimeout(() => {
      const link = document.createElement('link');
      link.rel = 'prefetch';
      link.href = prefetchUrl;
      link.credentials = 'include';
      document.head.appendChild(link);
    }, delay);

    // 防止用户快速
    setPrefetchTimeout(timeout);
  };

  const handleMouseOut = () => {
    // 如果过度发 prefetch 请求
    clearTimeout(prefetchTimeout);
  };

  return (
    <div className="product"
         onMouseOver={handleMouseOver}
         onMouseOut={handleMouseOut}>
      <img src={imageUrl} alt={name} loading="lazy" />
      <p>{name}</p>
    </div>
  );
}
```

### 添加 credentials 属性，携带 cookie
安全原因 prefetch 请求默认不携带 cookie，为了让 prefetch 请求携带 cookie， 可以在 prefetch 的 link 标签中添加 credentials 属性，并将其设置为 "include"

```jsx
<link rel="prefetch" href="..." as="script" credentials="include">
```

> 如果使用 js 生成 link 标签需要使用 `link.crossOrigin='use-credentials';`
>

### 服务器设置缓存
因为大部分动态页面为了给用户传输动态内容是禁用客户端缓存的，所以即使发了 prefetch 请求也无法做到用户真实点击的时候复用 prefetch 请求，反而会重新发请求造成资源浪费

因此需要在服务端识别 prefetch 请求，设置短时间的客户端缓存，当用户很快真实访问 prefetch 的页面后可以复用缓存

浏览器发送的 prefetch 请求会携带 HTTP Header `[Sec-Purpose: prefetch](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Purpose)`或`Purpose: prefetch`，服务端根据这个属性识别 prefetch 请求

```jsx
app.get('/next-page', (req, res) => {
  const purposeHeader = req.headers['purpose'] || req.headers['sec-purpose'];
  if (purposeHeader === 'prefetch') {
    res.set('Cache-Control', 'max-age=10'); // 设置缓存策略
    console.log('Prefetch request detected, setting cache.');
  } else {
    console.log('Regular request detected, no cache.');
  }
  res.send(`
    <h1>Next Page Content</h1>
    <p>This is the next page that was prefetched.</p>`
  );
});
```

## 页面与首屏请求并行加载
上述方案在 SSR 页面效果显著，但在 CSR 页面可能优化效果有限，主要原因是 CSR 页面内容存储在 CDN 甚至客户端本地缓存，本身加载很快，页面的渲染主要依赖动态接口的返回

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724499730005-523cc1d4-a7b4-4841-a14c-b7cf466bfcd6.png)

如果我们可以知道页面首屏渲染需要发起的请求，其实可以利用和上面类似的原理，在用户点击页面的瞬间同时发起异步请求，当解析执行 JavaScript 脚本发送异步请求时可以判断本地已经有缓存，直接使用结果

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724500141302-5538946e-801c-4f35-81fd-45bfa1a09121.png)

原理非常类似，不再代码演示，核心还是请求

+ 设置`credentials`请求可以携带 cookie
+ 服务端识别 prefetch 请求，对接口设置短时间的缓存

这样的方案最大程度利用了浏览器的特性实现起来比较简单，对 Service Worker 熟悉的话可以利用 Service 做更复杂的控制

## Speculation Rules API 
Speculation Rules API 是一个新的 Web API，提供一种声明式的方法来指示浏览器应该对哪些链接进行预取操作，通过这个 API，开发者可以更精确地指示浏览器在何时和如何预取资源，从而显著提升网页性能和用户体验

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Speculation Rules API</title>
    <!-- Speculation rules -->
    <script type="speculationrules">
      {
        "prefetch": []
      }
    </script>
  </head>
  <body>
    <a href="/page1.html" data-prefetch-url="/page1.html">Go to Page 1</a>
    <a href="/page2.html" data-prefetch-url="/page2.html">Go to Page 2</a>
    <a href="/page3.html" data-prefetch-url="/page3.html">Go to Page 3</a>

    <script>
      function addPrefetchRule(url) {
        const speculationRulesScript = document.querySelector('script[type="speculationrules"]');
        const rules = JSON.parse(speculationRulesScript.textContent);

        // 检查 rule 是不是已经设置
        if (!rules.prefetch.some(rule => rule.urls.includes(url))) {
          rules.prefetch.push({
            "source": "list",
            "urls": [url]
          });

          // 更新 speculation rules
          speculationRulesScript.textContent = JSON.stringify(rules);
          console.log(`Prefetch rule added for: ${url}`);
        }
      }

      // 鼠标 hover 时候添加
      document.querySelectorAll('a[data-prefetch-url]').forEach(link => {
        link.addEventListener('mouseover', () => {
          const url = link.getAttribute('data-prefetch-url');
          addPrefetchRule(url);
        });
      });
    </script>
  </body>
</html>
```

Speculation Rules API 目前还处于早期阶段，未来可能会看到更多的浏览器开始支持这一 API，并且 API 本身也可能会引入更多的功能和配置选项

## 页面部分内容提前返回
### 流式渲染简介
流式渲染（Streaming Rendering）是指在服务器上生成页面内容时，逐步将已准备好的部分内容立刻发送到客户端，而不是等待页面所有内容全部生成才开始发送，使客户端可以更快的接收数据渲染页面，而不必等待整个页面的内容完全下载，从而实现快速的页面加载和用户可视化体验。这个过程像是水管中的水一样流动起来源源不断，因此被称为流式渲染

流式渲染实际上一个非常古老的技术，早在 HTTP 1.1 规范中就已经引入了 `Transfer-Encoding: chunked` 头字段，允许服务器将响应内容分批返回给客户端。服务器可以在生成响应内容的同时，将其分成小块，逐步传输给客户端，而不是等待所有内容生成完成后再返回

在浏览器端，早期的浏览器（如 Netscape Navigator 和 IE）就已经支持对部分 HTML 内容进行解析和执行。当浏览器接收到服务器返回的部分 HTML 内容时，它可以立即开始解析和执行该内容，而不需要等待所有内容加载完成

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723376892245-3cd5f559-ba57-45cb-af6a-2c3ee968dd80.gif)

```javascript
const http = require('http');

http
  .createServer((req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    });
    function renderChunk(chunk) {
      res.write(`<div>${chunk}</div>`);
    }

    renderChunk('Loading...');

    setTimeout(() => {
      renderChunk('Chunk 1');
    }, 1000);

    setTimeout(() => {
      renderChunk('Chunk 2');
    }, 2000);

    setTimeout(() => {
      renderChunk('Chunk 3');
    }, 3000);

    setTimeout(() => {
      renderChunk('done!');
      res.write('</body></html>');
      res.end();
    }, 4000);
  })
  .listen(3000, () => {
    console.log('Server listening on port 3000');
  });
```

### 开启流式渲染后的新思路
页面支持流式渲染之后我们可以利用等待服务器计算生成动态内容的空档，提前返回页面部分内容，在浏览器完成关键域名的预建连、核心资源的预加载，严格来讲下面讲的很多内容其实是 HTTP 协议实现的，但思路上和流式渲染原理一致，所以放在一块来讨论

### 提前返回 preconnenct、preload 标签
页面可以对静态部分做缓存，接收到用户请求后流式渲染直接返回（其实这种最适合利用 CDN 边缘渲染）

页面静态部分 public/static.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Optimized Page</title>
  <!-- Preconnect to an external domain -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  
  <!-- Preload a critical CSS file -->
  <link rel="preload" href="/styles/main.css" as="style">
  
  <!-- Preload an important image -->
  <link rel="preload" href="/images/hero-banner.jpg" as="image">
</head>
<body>
```

如果服务器 RT 过长，甚至可以反直觉的在页面顶部预载 JavaScript 文件，但不执行

server.js

```jsx
const http = require('http');
const path = require('path');
const fs = require('fs');

const filePath = path.join(__dirname, 'public', 'static.html');

http
  .createServer((req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    });

    function renderChunk(chunk) {
      res.write(`<div>${chunk}</div>`);
    }

    fs.readFile(filePath, 'utf8', (err, firstFragment) => {
      // 返回静态部分，浏览器提前建连、加载
      renderChunk(firstFragment);
    });

    renderChunk('Loading...');

    setTimeout(() => {
      // 复杂的服务端计算
      renderChunk('done!');
      res.write('</body></html>');
      res.end();
    }, 4000);
  })
  .listen(3000, () => {
    console.log('Server listening on port 3000');
  });
```

### 根据数据生成 preload html 片段
其实我们还可以把服务器 RT 部分细分，🙋‍♀️🌰 页面取数部分实际非常复杂，而恰好首屏呈现的部分内容取数很快，后续取数或 SSR 很慢

+ 服务器首屏取数
+ 服务器取数 2
+ 服务器取数 3
+ 调用页面 SSR
+ 返回服务武器渲染部分

这种时候可以在调用 ssr 之前，解析首屏数据生成，如果包含图片，可以生成 preload 标签，提前返回到浏览器，设置对首屏部分调用占位的 SSR

```javascript
const http = require('http');

http
  .createServer((req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    });

    function renderChunk(chunk) {
      res.write(`<div>${chunk}</div>`);
    }
    
   setTimeout(async () => {
      const firstData = await getFirstScreenData();
      renderChunk(`
        <link rel="preload" href="${firstData.imgSrc}" as="image">
      `);
    }, 1000);

    setTimeout(() => {
      // 复杂的服务端计算
      renderChunk('done!');
      res.write('</body></html>');
      res.end();
    }, 4000);
  })
  .listen(3000, () => {
    console.log('Server listening on port 3000');
  });
```

### http header 返回后续域名 preconenct
除了在 HTML 中通过 link 标签支持 preconnect，足够了解 HTTP 协议后我们还可以更快一些，在 HTTP header 中设置 preconenct，这就是 [HTTP Link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link)

```plain
HTTP/2 200 OK
Content-Type: text/html
Link: <https://example.com>; rel=preconnect, <https://fonts.googleapis.com>; rel=preconnect
```



server.js

```javascript
app.get('/', (req, res) => {
  // Set Link headers for preconnect
  res.set('Link', [
    '<https://example.com>; rel=preconnect',
    '<https://fonts.googleapis.com>; rel=preconnect'
  ].join(', '));
  
  // Flush headers to send them immediately
  res.flushHeaders();

  // Stream the HTML file
  const readStream = fs.createReadStream('content');

  readStream.pipe(res);
});
```

## HTTP/2 push
HTTP/2 Push 是 HTTP/2 协议中的一种功能，允许服务器在响应客户端请求时，主动将多个资源推送给客户端，而无需客户端明确请求这些资源

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724505077605-727b435d-8d93-46bb-b3cc-996d6e6e122b.png)

在 NGINX 中，http2_push 指令用于启用或禁用 HTTP/2 push

```nginx
server {
    listen 443 ssl http2;
    server_name localhost;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    location / {
        root /path/to/your/web/content;
        index index.html;
       
        # 启用 HTTP/2 推送
        http2_push_preload on;
    
        http2_push banner.jpg;
    }
}
```



看起来怎么这么熟悉，没错，HTTP/2 push 在很多时候就是利用 HTTP Link 特性实现的

```nginx
server {
    listen 443 ssl http2;
    server_name localhost;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    location / {
        root /path/to/your/web/content;

        # 启用 HTTP/2 推送
        http2_push_preload on;

        # 发送 HTML 文档的同时，告诉客户端推送资源
        add_header Link "<banner.jpg.css>; as=image; rel=preload";
    }
}
```



不过通过 Nginx 配置来完成这个工作过于不灵活，大部分时候是通过上面讲的在服务代码中实现

## Early Hints
[https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/103)

在 HTTP 1xx 的状态码用来告示客户端继续进行请求或等待更详细的响应，比如在 WebSocket 交换协议期间返回的 101

```plain
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```



还有一个专门用于服务器希望发送最终响应头部之前，提供一些消息头，客户端可以开始预加载资源的 103 —— Early Hints，其作用和前面提到的 http link 非常类似

```plain
HTTP/1.1 103 Early Hints
Link: </style.css>; rel=preload; as=style
```



server.js

```javascript
app.get('/', (req, res) => {
  // Send 103 Early Hints response to client
  res.status(103).set({
    Link: [
      '</styles/main.css>; rel=preload; as=style',
      '</images/example.jpg>; rel=preload; as=image',
    ].join(', ')
  }).end();

  // Set up final response
  const readStream = fs.createReadStream('content');
  res.set('Content-Type', 'text/html');

  // Stream the final response content
  readStream.pipe(res);
});
```



Early hints  preconnect 已经在主流浏览器都得到普遍支持，preload Safari 还没有支持

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724504241416-36a94a04-f071-4867-a127-7fdd34e13fa6.png)



