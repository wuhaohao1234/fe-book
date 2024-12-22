[Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) (SSE) 是一种允许服务器端主动向客户端推送消息的技术。它基于 HTTP 协议提供了一种简单、高效的方式，为浏览器中的网页提供实时更新功能。SSE 是一种单向的服务器到客户端的数据流，通常用于需要持续传送更新的应用程序中，如股票行情、天气更新、实时通知等

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732543670415-5ec2d11b-efb6-47f6-9361-feafacfdfa6a.png)

## 特点
1. **单向通信**: SSE 是服务器向客户端推送数据的单向通道。客户端可以接收来自服务器的实时更新，但不会向服务器主动发送消息
2. **简单性和浏览器支持**: SSE 使用简单的文本格式来传输事件，并在大多数现代浏览器中有内置支持。
3. **持久连接**: 使用 SSE 建立的连接是持久的。浏览器会自动保持与服务器的连接，并在中断后尝试重新连接
4. **文本传输**: 传输的数据是纯文本格式，事件流是文本类型，每个消息以一个或多个换行符隔开
5. **自带重连机制**: 如果连接因为任何原因断开，浏览器会自动重新连接

## 使用示例
### 服务器端
在服务器端，标识一个响应为 Server-Sent Events (SSE) 主要是通过设置适当的 HTTP 响应头以及使用正确的格式来发送数据

#### 设置正确的 HTTP 响应头
当服务器准备好发送 SSE 流时，需要先设置一些必要的 HTTP 头来告知客户端这是一个 SSE 流。

1. Content-Type: 必须设置为 text/event-stream，这表明服务器将会提供的是事件流，客户端应将其解释为 SSE 数据
2. Cache-Control: 一般设置为 no-cache，以防止代理或浏览器缓存数据，确保客户端实时接收数据
3. Connection: 设置为 keep-alive 确保连接保持打开状态，因为 SSE 是长期连接

#### 数据格式
数据需要按照 SSE 格式发送，以特定的文本格式结构化数据：

1. 每个事件以一行或多行文本组成，以一对换行符 `\n\n` 结束
2. 使用 data: 前缀来表示要发送的数据行，例如 `data: message\n\n`
3. 可以使用 id、event、retry 等字段来附加更多信息
    1. **id**: 设置事件 ID，如果客户端连接丢失后，这个 ID 可以用于恢复连接时确定从哪里开始
    2. **event**: 自定义事件名称
    3. **retry**: 定义客户端在尝试重新连接前等待的毫秒数

```javascript
const http = require('http');

http.createServer((req, res) => {
  // 设置 HTTP 头以标识 SSE
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  // 定义数据流中的事件 ID 和重试间隔
  let clientId = 0;
  const retryInterval = 5000; // 5秒

  // 定时发送事件
  const eventSender = setInterval(() => {
    clientId++;

    // 构建 SSE 数据，并包含 id、event 和 retry 字段
    const message = `id: ${clientId}\n` + 
                    `event: myCustomEvent\n` + 
                    `retry: ${retryInterval}\n` +
                    `data: This is message number ${clientId}\n\n`;

    res.write(message);
  }, 3000); // 每3秒发送一次事件

  // 响应关闭时清除定时器
  req.on('close', () => {
    clearInterval(eventSender);
  });

}).listen(3000, () => {
  console.log('SSE server running at http://localhost:3000/');
});
```

### 客户端
在客户端中，可以使用 `EventSource` 接口来连接到服务器并接收事件。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SSE Demo</title>
  <script>
    document.addEventListener("DOMContentLoaded", function () {
      const eventSource = new EventSource('http://localhost:3000');

      // 监听事件
      eventSource.addEventListener('myCustomEvent', function(event) {
        console.log('Received myCustomEvent:', event.data);
      });

      // 处理默认消息事件
      eventSource.onmessage = function(event) {
        console.log('General message:', event.data);
      };

      // 错误处理
      eventSource.onerror = function(err) {
        console.error('EventSource failed:', err);
      };
    });
  </script>
</head>
<body>
  <h1>Server-Sent Events Demo</h1>
  <p>Check the console for messages.</p>
</body>
</html>
```

## 与 WebSocket 的比较
+ **协议及用途**: SSE 基于 HTTP 协议，适合于简单的单向更新。WebSocket 是一个更复杂的协议，可实现全双工通信，多用于需要实时、双向信息交换的应用
+ **使用场景**: SSE 更适合于服务器需要频繁发送消息更新而客户端只需接收的场景。而当客户端也需要发送消息，甚至大量实时交互时，WebSocket 是更好的选择
+ **连接管理**: SSE 将自动管理重新连接，WebSocket 则要求手动处理连接的丢失和重新连接

