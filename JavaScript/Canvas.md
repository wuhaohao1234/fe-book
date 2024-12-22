Canvas 是 HTML5 引入的一个用于绘制图形的元素，通过 JavaScript 可以动态地在网页上绘制和操作图像、图形、动画等。Canvas 提供了一块可编程的位图区域，开发者可以通过绘图上下文对其进行绘制

## 基础概念
html 提供了 `canvas`元素，这是一个空白的矩形区域，可以使用 JavaScript 在上面绘制图形、图像和文本

```html
<canvas id="myCanvas" width="600" height="400"></canvas>
```

<font style="color:rgb(51, 51, 51);">要在 Canvas 上绘图首先需要获取绘图上下文，绘图上下文是用于在 Canvas 上绘制图形的环境，常见的类型有：</font>

+ **<font style="color:rgb(51, 51, 51);">2D 上下文</font>**<font style="color:rgb(51, 51, 51);">：用于二维图形绘制，提供了丰富的绘图方法，如绘制路径、填充、描边、绘制图像和文本等</font>
+ **<font style="color:rgb(51, 51, 51);">WebGL 上下文</font>**<font style="color:rgb(51, 51, 51);">：用于三维图形绘制，基于 OpenGL ES 3D 图形库，适用于复杂的 3D 渲染和高性能图形应用</font>

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// 绘制描边矩形
ctx.strokeRect(220, 50, 150, 100); // (x, y, width, height)
```

## 基本绘图操作
**绘制矩形**

```javascript
ctx.fillRect(x, y, width, height);  // 填充矩形
ctx.strokeRect(x, y, width, height);  // 矩形轮廓
```

**绘制路径**

```javascript
// 开始路径
ctx.beginPath();
ctx.moveTo(300, 300); // 起点

ctx.lineTo(350, 250); // 第一条线
ctx.lineTo(400, 300); // 第二条线
ctx.lineTo(350, 350); // 第三条线
ctx.closePath();
ctx.stroke();
```

**绘制圆形**

```javascript
ctx.beginPath();
ctx.arc(x, y, radius, startAngle, endAngle);
ctx.stroke();  // 或 ctx.fill()
```

**填充与描边**

```javascript
// 填充
ctx.fillStyle = 'rgba(0, 128, 255, 0.5)';
ctx.fill();

// 描边
ctx.strokeStyle = '#000000';
ctx.lineWidth = 3;
ctx.stroke();
```

## 图像绘制
Canvas 可以绘制图片，通过 `drawImage` 方法将图片绘制到 Canvas 上

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const img = new Image();

img.onload = function() {
  // 绘制原始大小
  ctx.drawImage(img, 0, 0);

  // 绘制缩放后的图片
  ctx.drawImage(img, 300, 200, 150, 100); // (image, x, y, width, height)
};

img.src = 'https://www.alibaba.com/favicon.ico'; // 替换为实际图像 URL
```

## 文本绘制
通过 `fillText` 和 `strokeText` 方法，可以在 Canvas 上绘制文本。

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// 设置字体
ctx.font = '48px Arial';
ctx.fillStyle = '#FF0000';
ctx.fillText('Hello, Canvas!', 50, 100);

// 设置描边样式
ctx.strokeStyle = '#000000';
ctx.lineWidth = 2;
ctx.strokeText('Hello, Canvas!', 50, 200);
```

## 简单动画
在 requestAnimationFrame 周期内，通过清除先前的 Canvas 内容并绘制新内容，可以实现流畅的动画效果

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
let posX = 0;
const posY = 150;

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height); // 清除画布

  // 绘制移动的方块
  ctx.fillStyle = '#FF5733';
  ctx.fillRect(posX, posY, 50, 50);

  posX += 2; // 更新位置

  if (posX > canvas.width) posX = -50; // 重置位置

  requestAnimationFrame(draw); // 循环调用
}

draw(); // 启动动画
```

## 性能优化
在使用 Canvas 进行绘图和动画时，性能优化尤为重要，尤其是在处理复杂图形和高频率动画时

### 减少重绘次数
尽量减少 Canvas 的重绘次数，只在必要时更新绘图内容。使用 `requestAnimationFrame` 来协调动画更新，避免不必要的渲染。

```javascript
function draw() {
  // 仅在需要时重绘
  if (needsRedraw) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    // 执行绘图操作
    needsRedraw = false;
  }
  requestAnimationFrame(draw);
}
```

### 使用离屏 Canvas
利用离屏 Canvas 进行预绘，然后将结果绘制到主 Canvas 上，减少主线程的计算压力，提升渲染效率

```javascript
// 创建离屏 Canvas
const offscreenCanvas = document.createElement('canvas');
offscreenCanvas.width = 200;
offscreenCanvas.height = 200;
const offscreenCtx = offscreenCanvas.getContext('2d');

// 在离屏 Canvas 上绘制
offscreenCtx.fillStyle = 'red';
offscreenCtx.fillRect(0, 0, 200, 200);

// 将离屏 Canvas 的内容绘制到主 Canvas
const mainCanvas = document.getElementById('myCanvas');
const mainCtx = mainCanvas.getContext('2d');
mainCtx.drawImage(offscreenCanvas, 50, 50);
```

### 优化像素操作
在进行像素级操作（如获取和修改图像数据）时，尽量减少对 `ImageData` 的频繁访问。一次性获取和修改数据，然后一次性写回

```javascript
// 获取像素数据
let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
let data = imageData.data;

// 修改像素数据
for (let i = 0; i < data.length; i += 4) {
  data[i] = 255 - data[i];     // 红色通道
  data[i + 1] = 255 - data[i + 1]; // 绿色通道
  data[i + 2] = 255 - data[i + 2]; // 蓝色通道
  // data[i + 3] 是 alpha 通道
}

// 写回像素数据
ctx.putImageData(imageData, 0, 0);
```

### 合理使用涂层
将复杂的静态背景和动态元素分层绘制，可以减少需要频繁更新的绘图区域，提升整体渲染效率

```html
<div style="position: relative;">
  <!-- 静态背景 Canvas -->
  <canvas id="backgroundCanvas" width="600" height="400" style="position: absolute; top: 0; left: 0; z-index: 0;"></canvas>

  <!-- 动态元素 Canvas -->
  <canvas id="foregroundCanvas" width="600" height="400" style="position: absolute; top: 0; left: 0; z-index: 1;"></canvas>
</div>

<script>
  const bgCanvas = document.getElementById('backgroundCanvas');
  const bgCtx = bgCanvas.getContext('2d');

  // 绘制静态背景
  bgCtx.fillStyle = '#EEEEEE';
  bgCtx.fillRect(0, 0, bgCanvas.width, bgCanvas.height);

  const fgCanvas = document.getElementById('foregroundCanvas');
  const fgCtx = fgCanvas.getContext('2d');
  let posX = 0;
  const posY = 200;

  function animate() {
    fgCtx.clearRect(0, 0, fgCanvas.width, fgCanvas.height);
    fgCtx.fillStyle = '#FF0000';
    fgCtx.fillRect(posX, posY, 50, 50);
    posX += 2;
    if (posX > fgCanvas.width) posX = -50;
    requestAnimationFrame(animate);
  }

  animate();
</script>
```

## Canvas VS SVG
SVG（Scalable Vector Graphics）是一种用来描述**二维矢量图形**的XML格式。它是一种基于文本的图像格式，支持交互和动画，广泛用于网页设计和开发中。在选择 Canvas 或 SVG 技术进行项目开发时，需要考虑到它们各自的特性、适用场景和项目需求

+ **Canvas** 是基于像素的技术，适用于生成即时图形，在处理大量对象（如游戏中的图形）时，Canvas 通常表现得更好，因为它直接操作位图。但每次绘制都是对整个 Canvas 的再绘制，无法直接对单个元素进行操作
+ **SVG** 是矢量图形技术，缩放时不会失去清晰度，适合高保真度的图像展示，每个元素都是独立可操作的 DOM 元素，可以添加事件、样式和脚本，但对于复杂图形，DOM 节点数可能较大

| **<font style="color:rgb(51, 51, 51);">特性</font>** | **<font style="color:rgb(51, 51, 51);">Canvas</font>** | **<font style="color:rgb(51, 51, 51);">SVG</font>** |
| :--- | :--- | :--- |
| **<font style="color:rgb(51, 51, 51);">绘制方式</font>** | <font style="color:rgb(51, 51, 51);">基于像素的位图绘制</font> | <font style="color:rgb(51, 51, 51);">基于矢量的图形绘制</font> |
| **<font style="color:rgb(51, 51, 51);">适用场景</font>** | <font style="color:rgb(51, 51, 51);">高性能实时渲染、大量动态图形、游戏等</font> | <font style="color:rgb(51, 51, 51);">需要可缩放、交互性高的静态图形、复杂的布局与样式</font> |
| **<font style="color:rgb(51, 51, 51);">DOM 复杂性</font>** | <font style="color:rgb(51, 51, 51);">仅一个 </font>`<font style="color:rgb(51, 51, 51);"><canvas></font>`<font style="color:rgb(51, 51, 51);">元素，绘图内容不在 DOM 中</font> | <font style="color:rgb(51, 51, 51);">图形元素作为 DOM 节点存在，易于操作和样式化</font> |
| **<font style="color:rgb(51, 51, 51);">性能</font>** | <font style="color:rgb(51, 51, 51);">适合大量图形和高频率更新，性能较高</font> | <font style="color:rgb(51, 51, 51);">动态元素较多时性能可能下降，特别是复杂的 SVG 图形</font> |
| **<font style="color:rgb(51, 51, 51);">可访问性</font>** | <font style="color:rgb(51, 51, 51);">需要额外处理，默认不可访问</font> | <font style="color:rgb(51, 51, 51);">元素可被屏幕阅读器等辅助技术识别，具备更好的可访问性</font> |
| **<font style="color:rgb(51, 51, 51);">动画与交互</font>** | <font style="color:rgb(51, 51, 51);">需要手动实现动画和交互逻辑</font> | <font style="color:rgb(51, 51, 51);">支持 CSS 动画、SVG 动画和事件处理</font> |


