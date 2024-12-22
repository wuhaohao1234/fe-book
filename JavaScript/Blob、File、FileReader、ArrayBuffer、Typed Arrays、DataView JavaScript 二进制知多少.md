JavaScript 通常处理的是文本数据，但在更复杂的应用中处理二进制数据（图像、音频、视频等）变得尤为重要。JavaScript 提供了一系列 API 来创建、操作和处理这些二进制数据：

+ **Blob**：用于代表不可变的二进制数据（如文件）
+ **File**：继承自 Blob，代表用户从文件系统选择的具体文件
+ **FileReader**：用于读取 File 或 Blob 对象的内容
+ **ArrayBuffer**：代表通用、固定长度的原始二进制数据缓冲区
+ **Typed Arrays**：提供了一种操作 ArrayBuffer 中数据的方式，支持多种数据类型（如 Uint8Array、Float32Array 等）
+ **DataView**：允许以任意字节顺序（endianness）读取和写入 ArrayBuffer 中的数据

## Blob
Blob（Binary Large Object）是 JavaScript 提供的一个用于表示不可变的原始二进制数据的对象，它通常用于处理文件、图像、音频、视频等类型的二进制数据，有三个特性

+ **支持二进制数据**：可以存储各种类型的数据，如文本、图像、音频等
+ **不可变性**：创建后 Blob 的数据不能被改变
+ **<font style="color:rgb(51, 51, 51);">分块存储</font>**<font style="color:rgb(51, 51, 51);">：支持通过 Blob 的 slice 方法将一个大的 Blob 对象切分为多个更小的 Blob 对象</font>

### 创建 Blob
可以通过传递字符串或字符串数组来创建 Blob 对象，也可以直接通过二进制数据创建 Blob 对象

```javascript
// 从字符串创建 Blob
const text = "Hello, Blob!";
// 为 Blob 指定正确的 MIME 类型，确保数据在使用时被正确解析和处理
const blob = new Blob([text], { type: 'text/plain' });

console.log(blob.size); // 输出: 12
console.log(blob.type); // 输出: 'text/plain'

// 从二进制数据创建 Blob
const bytes = new Uint8Array([0x48, 0x65, 0x6C, 0x6C, 0x6F]); // 'Hello'
const binaryBlob = new Blob([bytes], { type: 'application/octet-stream' });

console.log(binaryBlob.size); // 输出: 5
console.log(binaryBlob.type); // 输出: 'application/octet-stream'
```

### 使用 Blob
Blob 对象常用于处理文件上传和下载。例如通过 Blob 可以创建一个包含特定数据的文件，然后通过 `URL.createObjectURL` 将其转换为下载链接

```javascript
const data = "Sample data for download";
const blob = new Blob([data], { type: 'text/plain' });
const url = URL.createObjectURL(blob);

const downloadLink = document.createElement('a');
downloadLink.href = url;
downloadLink.download = 'sample.txt';
downloadLink.textContent = 'Download File';
document.body.appendChild(downloadLink);
```

Blob 也经常用于处理和操作图像数据，例如将 `Canvas` 内容转换为图像 Blob，然后显示

```javascript
const canvas = document.getElementById('myCanvas');
canvas.toBlob((blob) => {
  const url = URL.createObjectURL(blob);
  const img = document.createElement('img');
  img.src = url;
  document.body.appendChild(img);
}, 'image/png');
```

> 使用 `URL.revokeObjectURL()` 在不需要时释放对象 URL，防止内存泄漏
>

在进行网络通信时，Blob 可以作为数据传输的容器，尤其是在使用 `fetch` API 时传输二进制数据

```javascript
fetch('https://example.com/api/upload', {
  method: 'POST',
  body: blob,
  headers: {
    'Content-Type': 'text/plain'
  }
}).then(response => {
  return response.json();
}).then(data => {
  console.log(data);
});

```

### blob 方法
+ **slice(start, end, contentType)：**截取 Blob 的部分数据，生成新的 Blob
+ **arrayBuffer()：**将 Blob 转换为 ArrayBuffer，适用于二进制数据操作
+ **text()：**将 Blob 转换为字符串，适用于文本数据读取
+ **stream()：**获取 Blob 的可读流，适用于逐步处理大型数据

```javascript
const blob = new Blob(["Hello, Stream!"], { type: 'text/plain' });
const readableStream = blob.stream();
const reader = readableStream.getReader();

reader.read().then(({ done, value }) => {
  if (!done) {
    // 创建一个字符串
    const text = new TextDecoder().decode(value);
    console.log(text); // 输出: "Hello, Stream!"
  }
});
```

## File
File 对象是 Blob 对象的子类，表示来自用户文件系统的文件，因此 File 对象不仅包含了二进制数据，还包含了文件的元数据，如文件名、大小和类型

### 使用 File 对象
最常见的 File 对象创建方式是通过用户在网页中上传文件。这通常通过 `<input type="file">` 元素或拖放操作实现

```html
<input type="file" id="fileInput" />
<script>
  const fileInput = document.getElementById('fileInput');
  
  fileInput.addEventListener('change', (event) => {
    const file = event.target.files[0];
    console.log(file.name); // 'example.txt'
    console.log(file.size); // 文件大小，单位为字节
    console.log(file.type); // 'text/plain'
  });
</script>

```

### File 对象属性
File 对象继承自 Blob，因此拥有 Blob 的所有属性和方法，并附加了一些新的属性

| **<font style="color:rgb(51, 51, 51);">属性</font>** | **<font style="color:rgb(51, 51, 51);">描述</font>** |
| :--- | :--- |
| <font style="color:rgb(51, 51, 51);">name</font> | <font style="color:rgb(51, 51, 51);">文件的名称，包含扩展名</font> |
| <font style="color:rgb(51, 51, 51);">lastModified</font> | <font style="color:rgb(51, 51, 51);">文件的最后修改时间，以时间戳形式表示</font> |
| <font style="color:rgb(51, 51, 51);">lastModifiedDate</font> | <font style="color:rgb(51, 51, 51);">lastModified 的 Date对象</font> |
| <font style="color:rgb(51, 51, 51);">webkitRelativePath</font> | <font style="color:rgb(51, 51, 51);">用户选择文件时的相对路径</font> |


```javascript
const fileInput = document.getElementById('fileInput');
const fileInfo = document.getElementById('fileInfo');

fileInput.addEventListener('change', (event) => {
  const file = event.target.files[0];
  if (file) {
    fileInfo.textContent = `
      文件名: ${file.name}
      文件大小: ${file.size} 字节
      文件类型: ${file.type}
      最后修改时间: ${new Date(file.lastModified)}
      相对路径: ${file.webkitRelativePath || '无'}`;
  } else {
    fileInfo.textContent = '未选择任何文件。';
  }
});
```

## FileReader
FileReader 允许 Web 应用程序异步读取 File 或 Blob 对象的内容，支持多种读取方法，如读取为文本、数据 URL 或 ArrayBuffer

FileReader 对象可以通过调用其构造函数直接创建

```javascript
const reader = new FileReader();
```

使用 FileReader.abort() 方法可以取消正在进行的读取操作

### 主要方法
+ **readAsArrayBuffer(blob)**：读取 Blob 并返回 ArrayBuffer
+ **readAsDataURL(blob)**：读取 Blob 并返回 data URL（base64 编码的字符串）
+ **readAsText(blob, [encoding])**：读取 Blob 并返回文本字符串

当调用上述方法后会触发 FileReader 的事件

| **<font style="color:rgb(51, 51, 51);">事件</font>** | **<font style="color:rgb(51, 51, 51);">描述</font>** |
| :--- | :--- |
| <font style="color:rgb(51, 51, 51);">onloadstart</font> | <font style="color:rgb(51, 51, 51);">开始读取操作时触发</font> |
| <font style="color:rgb(51, 51, 51);">onprogress</font> | <font style="color:rgb(51, 51, 51);">读取过程中定期触发，可用于显示进度</font> |
| <font style="color:rgb(51, 51, 51);">onabort</font> | <font style="color:rgb(51, 51, 51);">读取操作被中止时触发</font> |
| <font style="color:rgb(51, 51, 51);">onerror</font> | <font style="color:rgb(51, 51, 51);">读取过程中发生错误时触发</font> |
| <font style="color:rgb(51, 51, 51);">onload</font> | <font style="color:rgb(51, 51, 51);">读取操作成功完成时触发</font> |
| <font style="color:rgb(51, 51, 51);">onloadend</font> | <font style="color:rgb(51, 51, 51);">读取操作完成（无论成功或失败）时触发</font> |


### 文件上传与读取
在用户上传文件后，使用 FileReader 可以提前读取文件内容，进行预览、验证或处理。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>读取文本文件示例</title>
  </head>
  <body>
    <input type="file" id="textFileInput" accept=".txt, .md" />
    <pre id="fileContent"></pre>

    <script>
      const textFileInput = document.getElementById('textFileInput');
      const fileContent = document.getElementById('fileContent');

      textFileInput.addEventListener('change', (event) => {
        const file = event.target.files[0];
        if (file) {
          const reader = new FileReader();

          reader.onload = (e) => {
            fileContent.textContent = e.target.result;
          };

          reader.onerror = () => {
            fileContent.textContent = '读取文件时发生错误。';
          };

          reader.readAsText(file);
        } else {
          fileContent.textContent = '未选择任何文件。';
        }
      });
    </script>
  </body>
</html>
```

使用 readAsDataURL 方法，可以将用户上传的图像文件转换为可以直接在网页中显示的 URL

```html
<!DOCTYPE html>
<html>
<head>
  <title>图像预览示例</title>
</head>
<body>
  <input type="file" id="imageInput" accept="image/*" />
  <img id="imagePreview" src="" alt="图像预览" style="max-width: 300px;" />

  <script>
    const imageInput = document.getElementById('imageInput');
    const imagePreview = document.getElementById('imagePreview');

    imageInput.addEventListener('change', (event) => {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();

        reader.onload = (e) => {
          imagePreview.src = e.target.result;
        };

        reader.onerror = () => {
          console.error('读取图像文件时发生错误。');
        };

        reader.readAsDataURL(file);
      } else {
        imagePreview.src = '';
      }
    });
  </script>
</body>
</html>
```

### 分块读取
对于非常大的文件，可以利用 `Blob.slice` 方法分块读取，避免占用过多内存。

```javascript
const chunkSize = 1024 * 1024; // 1MB
let offset = 0;

function readChunk(file) {
  if (offset >= file.size) return;

  const slice = file.slice(offset, offset + chunkSize);
  const reader = new FileReader();

  reader.onload = (event) => {
    const text = event.target.result;
    console.log(`读取第 ${offset / chunkSize + 1} 块:`, text);
    offset += chunkSize;
    readChunk(file);
  };

  reader.onerror = () => {
    console.error('读取块时发生错误。');
  };

  reader.readAsText(slice);
}

readChunk(file);
```

## ArrayBuffer 与 Typed Arrays、DataView
ArrayBuffer 是一种用于表示通用、固定长度的原始二进制数据缓冲区，它是处理二进制数据的基础，可以用于存储各种类型的数据，主要在以下领域使用

+ **网络通信**：WebSockets 和 Fetch API: 在与服务器进行低级网络通信时，ArrayBuffer 可以用于发送和接收二进制数据
+ **科学计算和数据分析**：在需要高效率大量数据运算的应用中（如图像处理、音频处理、科学计算），ArrayBuffer 配合 TypedArray 可实现更高效的数据处理
+ **图像处理**：可以在 Canvas 元素中使用 ArrayBuffer 来高效处理图像数据
+ **大数据处理**：ArrayBuffer 提供了对内存中二进制数据的直接访问权限，这在处理大数据集时尤其有用

```javascript
// 创建一个 8 字节的 ArrayBuffer
const buffer = new ArrayBuffer(8);

console.log(buffer.byteLength); // 输出: 8
```

ArrayBuffer 本身不能直接读取或写入数据，必须通过 Typed Arrays 或 DataView 对象来操作

### 使用 Typed Arrays 操作 ArrayBuffer
Typed Arrays 提供了一种操作 ArrayBuffer 中数据的方式，允许以特定的字节序和数据类型读写二进制数据

JavaScript 提供了多种类型的 Typed Arrays，每种对应不同的数据类型和字节长度，它们为不同的数据类型（如整数、浮点数）提供了高效的操作接口

| 类型 | 描述 | 字节长度 |
| --- | --- | --- |
| Int8Array | 8 位有符号整数 | 1 |
| Uint8Array | 8 位无符号整数 | 1 |
| Uint8ClampedArray | 8 位无符号整数，值被夹断到 [0, 255] | 1 |
| Int16Array | 16 位有符号整数 | 2 |
| Uint16Array | 16 位无符号整数 | 2 |
| Int32Array | 32 位有符号整数 | 4 |
| Uint32Array | 32 位无符号整数 | 4 |
| Float32Array | 32 位浮点数 | 4 |
| Float64Array | 64 位浮点数 | 8 |
| BigInt64Array | 64 位有符号大整数（ES2020 引入） | 8 |
| BigUint64Array | 64 位无符号大整数（ES2020 引入） | 8 |


Typed Arrays 特别适用于科学计算、图像处理、音频处理等需要高性能的数据操作场景

```javascript
const buffer = new ArrayBuffer(16); // 16 字节的缓冲区

// 创建一个 Float32Array 视图
const floatView = new Float32Array(buffer);

floatView[0] = 3.14;
floatView[1] = 2.718;

console.log(floatView); // 输出: Float32Array [ 3.14, 2.718, 0, 0 ]
```

### 使用 DataView 操作 ArrayBuffer
DataView 提供了一种更灵活的方式来操作 ArrayBuffer 中的数据，允许以任意的字节序（大端或小端）读写不同类型的数据。相比 Typed Arrays，DataView 更加通用，适用于需要处理复杂数据结构的场景

```javascript
const buffer = new ArrayBuffer(16); // 16 字节的缓冲区
const dataView = new DataView(buffer);

// 设置不同类型的数据
dataView.setInt8(0, 127);          // 第一个字节设置为有符号 8 位整数
dataView.setUint16(1, 65000, true); // 从第 1 个字节开始，设置无符号 16 位整数，字节序为小端
dataView.setFloat32(3, 3.14, false); // 从第 3 个字节开始，设置 32 位浮点数，字节序为大端

// 读取数据
console.log(dataView.getInt8(0));      // 输出: 127
console.log(dataView.getUint16(1, true)); // 输出: 65000
console.log(dataView.getFloat32(3, false)); // 输出: 3.140000104904175
```

### Typed Arrays 和 DateView 选择
+ 如果需要处理同一类型的数据，且不需要控制字节序，使用 Typed Arrays 更为方便和高效
+ 如果需要处理复杂的数据结构，或需要在数据中混合多种类型，且需要控制字节序，使用 DataView 更为合适

### 和 Blob 区别
ArrayBuffer 和 Blob 在 JavaScript 中都是用于处理二进制数据的对象，在部分场景两者都能胜任，但擅长的场景有所区别

+  ArrayBuffer 适用于需要高效处理和操作二进制数据的场景，如计算、数据分析、网络通信等，因为它允许对数据执行低级别的读写操作
+ Blob 主要用于文件处理，如文件上传/下载、在浏览器中生成文件等

### 图像颜色反转
假设我们从服务器获取了一张灰度图像的二进制数据，我们需要对其进行处理，比如反转颜色。

```javascript
// 模拟从服务器获取的图像数组（二进制数据）
function fetchImageData() {
  // 假设我们有一个 2x2 的灰度图像示例数据（4 个像素，每个像素 8 位灰度值）
  return new Uint8Array([100, 150, 200, 250]); // 灰度值
}

function processImageData() {
  // 获取图像数据
  let imageData = fetchImageData();

  // 创建一个 ArrayBuffer，长度与 imageData 相同
  let buffer = new ArrayBuffer(imageData.length);

  // 创建一个视图来操作这个缓冲区
  let view = new Uint8Array(buffer);

  // 复制数据到缓冲区
  for (let i = 0; i < imageData.length; i++) {
    view[i] = imageData[i];
  }

  // 处理图像数据（颜色反转）
  for (let i = 0; i < view.length; i++) {
    view[i] = 255 - view[i]; // 反转灰度值
  }

  // 输出处理后的数据
  console.log('Processed Image Data:', view);
}

processImageData();
```

  
  


