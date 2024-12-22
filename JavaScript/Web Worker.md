JavaScript 的执行环境通常是单线程的，这意味着所有的任务包括用户界面更新和复杂的计算，都是在一个线程上顺序执行的。如果某个任务耗时过长，就会阻塞其他任务的执行，从而导致用户界面的卡顿和响应迟缓。

Web Worker 提供了一种在后台运行 JavaScript 的机制，这些 JavaScript 脚本在独立的线程中执行，不会干扰主线程的运行，通过这种方式 Web Worker 能够显著提升现代 Web 应用的性能和响应能力

## Web worker 特点
+ **并行处理**：Web Worker 允许 JavaScript 在多个线程中并行执行，从而使得 CPU 密集型任务可以在后台进行
+ **独立环境**：Web Worker 运行在独立的上下文中，不能直接访问主线程的 DOM，但可以通过消息传递与主线程通信
+ **非阻塞**：由于工作线程与 UI 线程分离，通过在后台运行耗时任务（如复杂计算或压缩解压缩等），让主线程保持响应性，用户交互不受影响

## 基本用法
### 创建独立的 worker 脚本文件
要创建一个 Web Worker，首先需要有一个独立的 JavaScript 文件，这是因为 Worker 在自己的上下文中运行，并且不能直接访问主线程的 DOM

创建一个命名为 `worker.js` 的文件，其中包含需要在 Worker 中运行的代码。

```javascript
// worker.js

self.onmessage = function(event) {
  // Worker 接收到的数据
  const data = event.data;
  console.log('Data received from main thread:', data);

  // 执行一些计算或处理
  const result = data * 2; // 简单相乘操作

  // 将结果发送回主线程
  self.postMessage(result);
};
```

在 Web worker 中，`self` 对象是一个重要的全局对象，类似于主线程中的 `window` 对象。`self` 代表了 Worker 自己的执行环境

### 初始化 worker
在主 JavaScript 文件中创建 Worker 实例，指向 `worker.js`

```javascript
// main.js

// 创建一个新的 Worker 实例
const myWorker = new Worker('worker.js');

// 向 Worker 发送数据
myWorker.postMessage(5);

// 监听来自 Worker 的消息
myWorker.onmessage = function(event) {
  console.log('Result received from worker:', event.data);
};

// 错误处理
myWorker.onerror = function(error) {
  console.error('Error in worker:', error.message);
};
```

### 使用 postMessage 通信
如上面示例展示，主线程和 Worker 都可以使用`postMessage`发送消息给对方，使用 `onmessage` 事件监听从另一端接收的消息

### 使用 MessageChannel 通信
MessageChannel 是一种用于在不同浏览上下文（如两个不同的窗口、多个 iframe 或 Web Worker 和主线程之间）进行双向通信的机制。MessageChannel 的机制很简单：创建一个实例，该实例具有两个可以发送消息的 MessagePort，这些端口可以被分发到不同的上下文中，并通过 postMessage 方法发送消息，onmessage 事件接收消息

主线程

```javascript
const worker = new Worker('worker.js');

// 创建一个 MessageChannel 实例
const messageChannel = new MessageChannel();

// 发送第二个端口给 Worker
worker.postMessage({ port: messageChannel.port2 });

// 监听 port1 收到的消息
messageChannel.port1.onmessage = (event) => {
  console.log('Received from worker:', event.data);
};

// 发送消息到 Worker 通过 port1
messageChannel.port1.postMessage('Hello, worker!');
```

worker.js

```javascript
self.onmessage = (event) => {
  // 获取从主线程传递过来的端口
  const port = event.data.port;
  
  // 监听在这个端口上收到的消息
  port.onmessage = (event) => {
    console.log('Received from main thread:', event.data);
    // 回复消息回主线程
    port.postMessage('Hello, main thread!');
  };
};
```

### 终止 Worker
当不再需要 Worker 时，可以通过调用 `terminate` 方法来停止 Worker 并释放其资源。

```javascript
myWorker.terminate();
```

## Web worker 限制
+ **需要不同的脚本文件**：Worker 代码需要在一个与主脚本分开的文件中编写，加载模式和安全策略可能会有不同的要求
+ **无法直接访问 DOM**：Worker 运行在一个独立的线程中，不能直接操作页面的 DOM。这意味着如果需要更改 DOM，必须通过消息传递将信息发送回主线程进行操作
+ **同源限制**：Worker 脚本必须与发起它的页面在同一个源上，或允许通过 CORS 配置进行跨源加载

## 使用 Blob URLs 创建 Worker 实例
因为 Web worker 要求脚本地址和页面文档地址同源，很多时候两者使用不同域名，这时候除了在页面文档服务器设置静态资源转发规则，还可以将 Worker 脚本加载后，以 Blob URL 的形式创建一个新的 Worker 实例

```javascript
fetch('https://static.example.com/worker.js')
  .then(response => response.text())
  .then(workerCode => {
    const blob = new Blob([workerCode], {type: 'application/javascript'});
    
    // 生成一个指向此 Blob 的临时 URL
    const workerUrl = URL.createObjectURL(blob);
    const myWorker = new Worker(workerUrl);
  
    // 使用 Worker
  });
```

这种方式实际上是将逻辑脚本内容转换为了 Blob 对象，并从该对象创建了数据 URI，使用这种方案要注意两个问题

1. Worker 脚本不具备正常脚本解析和调试能力
2. 确保远程脚本的安全性和可信度，防止引入恶意代码

## 小试牛刀
很多时候需要在用户上传图像时对图像进行压缩或滤镜处理，这样的耗时任务可以放在 Web Worker 中处理，防止对主线程的阻塞

```javascript
// image-worker.js

self.onmessage = function(event) {
  const imageData = event.data;
  // 进行基本图像处理，例如灰度滤镜
  for (let i = 0; i < imageData.data.length; i += 4) {
    let avg = (imageData.data[i] + imageData.data[i + 1] + imageData.data[i + 2]) / 3;
    imageData.data[i] = avg;        // Red channel
    imageData.data[i + 1] = avg;    // Green channel
    imageData.data[i + 2] = avg;    // Blue channel
    // Alpha channel remains unchanged
  }
  self.postMessage(imageData);
};
```

```javascript
/ main.js

const worker = new Worker('image-worker.js');

worker.onmessage = function(event) {
  const imageData = event.data;
  // 将处理过的 imageData 绘制到主线程的 canvas 上
  const canvas = document.getElementById('resultCanvas');
  const context = canvas.getContext('2d');
  context.putImageData(imageData, 0, 0);
};

// 加载图片并在 canvas 上进行绘制
const canvas = document.getElementById('sourceCanvas');
const context = canvas.getContext('2d');
const imageData = context.getImageData(0, 0, canvas.width, canvas.height);

// 将图像数据发送到 Worker 处理
worker.postMessage(imageData);
```

