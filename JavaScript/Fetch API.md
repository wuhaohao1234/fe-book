JavaScript 的 fetch API 是一种现代的基于 Promise 的机制，用于在浏览器中执行网络请求的 API，它提供了一种更加强大和灵活的方式来替代 XMLHttpRequest

## 发起 GET 请求
```javascript
fetch('https://api.example.com/data')
  .then(response => {
    if (!response.ok) {
      throw new Error('Network response was not ok ' + response.statusText);
    }
    return response.json(); // 将响应转换为 JSON 对象
  })
  .then(data => {
    console.log(data); // 处理成功响应的数据
  })
  .catch(error => {
    console.error('There has been a problem with your fetch operation:', error);
  });
```

## 处理响应
fetch 返回一个 Promise，该 Promise resolve 为一个 Response 对象，Response 对象包含了响应的状态信息和 body 数据，常用的方法包括：

+ `response.json()`: 解析响应为 JSON
+ `response.text()`: 将响应解析为文本
+ `response.blob()`: 将响应解析为 Blob
+ `response.arrayBuffer()`: 将响应解析为 ArrayBuffer

## async/await 风格
因为 Fetch API 基础 Promise，因此天然支持 async/await

```javascript
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error('Network response was not ok ' + response.statusText);
    }
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('There has been a problem with your fetch operation:', error);
  }
}

fetchData();
```

## 发起 POST 请求
fetch 函数接受两个参数：

1. **url**: 请求的资源地址
2. **options**: 可以指定请求的方法、头部信息（headers）、请求体（body）等

```javascript
const data = { username: 'example' };

fetch('https://api.example.com/submit', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(data),
})
  .then(response => response.json())
  .then(data => console.log('Success:', data))
  .catch(error => console.error('Error:', error));
```

## 中止请求
使用 AbortController 可以中止进行中的请求

```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch(url, { signal })
 .then(response => response.json())
 .catch(err => {
   if (err.name === 'AbortError') {
     console.log('Fetch aborted');
   } else {
     console.error('Uh oh, an error!', err);
   }
 });

// 中止请求
controller.abort();
```

## 跨域请求
Fetch API 在执行跨域请求时候默认不发送 cookies，防止潜在的 CSRF（跨站请求伪造）攻击，可以通过设置 `credentials`属性修改其行为

+ `'same-origin'`: 在同源请求中包含凭证信息
+ `'include'`: 总是包含凭证信息，不论跨域与否
+ `'omit'`: 从不发送凭证信息

```javascript
fetch('https://example.com/data', {
  method: 'GET',
  credentials: 'include'
})
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);">除了客户端配置，CORS 请求成功还需要服务器的响应头正确配置。例如，服务器需要设置以下 HTTP 响应头，以便客户端接受并发送 cookies。</font>

+ `Access-Control-Allow-Origin`：需要明确指定允许的请求来源。不能使用通配符 *，因为发送凭证时不支持通配符
+ `Access-Control-Allow-Credentials`：需要设置为 true 以允许 cookies 被传递

  


