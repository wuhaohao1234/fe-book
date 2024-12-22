[Flex 布局教程：语法篇 —— 阮一峰的网络日志](http://ruanyifeng.com/blog/2015/07/flex-grammar.html)

## 水平和垂直居中
```html
<style>
  .container {
    display: flex;
    justify-content: center; /* 水平居中 */
    align-items: center; /* 垂直居中 */
    height: 100vh; /* 整个视口高度 */
    background-color: #f0f0f0;
  }
  
  .box {
    padding: 20px;
    width: 200px;
    text-align: center;
    background-color: #4caf50;
    color: white;
    border-radius: 5px;
  }
</style>
<div class="container">
  <div class="box">居中内容</div>
</div>
```

## 列表卡片等高
Flexbox 布局容器默认使用 `align-items: stretch`，这使得所有子元素在交叉轴（即垂直方向，这里的高度）上尽可能拉伸到父容器的最大高度

```html
<style>
  .card-container {
    display: flex;
  }
  
  .card {
    flex: 1;
    margin: 10px;
    padding: 20px;
    background-color: #e1e1e1;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
</style>
<div class="card-container">
  <div class="card">卡片1内容</div>
  <div class="card">卡片2内容</div>
  <div class="card">卡片3内容</div>
</div>
```

## 左右双栏布局
```html
<style>
  .container {
    display: flex;
    min-height: 100vh;  /* 使容器占满整个视口高度 */
  }
  
  .sidebar {
    width: 200px;  /* 设置左侧栏固定宽度 200px */
    background-color: #f4f4f4;
    padding: 20px;
    box-sizing: border-box;
  }
  
  .main-content {
    flex: 1;  /* 右侧内容区宽度自适应 */
    background-color: #ffffff;
    padding: 20px;
    box-sizing: border-box;
  }
</style>
<div class="container">
  <div class="sidebar">左侧栏</div>
  <div class="main-content">
    <h1>右侧内容区域</h1>
    <p>这里是主内容区域，宽度自适应。</p>
  </div>
</div>
```

## header、content、footer 布局
```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body, html {
  height: 100%;
}

.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh; /* 使容器占满整个视口高度 */
}

.header {
  background-color: #4CAF50;
  padding: 20px;
  text-align: center;
  color: white;
}

.content {
  flex: 1;
  background-color: #f4f4f4;
  padding: 20px;
}

.footer {
  background-color: #333;
  padding: 10px;
  text-align: center;
  color: white;
}
```

```html
<div class="container">
  <header class="header">Header</header>
  <main class="content">
    <h1>Content</h1>
    <p>这里是主内容区域，可以包含各种内容。</p>
  </main>
  <footer class="footer">Footer</footer>
</div>
```

## 多列响应布局
```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.item {
  flex: 1 1 calc(25% - 20px);
  margin-bottom: 20px;
  padding: 20px;
  background-color: #e1e1e1;
  box-sizing: border-box;
}

@media (max-width: 1200px) {
  .item {
    flex: 1 1 calc(33.333% - 20px);
  }
}

@media (max-width: 768px) {
  .item {
    flex: 1 1 calc(50% - 20px);
  }
}

@media (max-width: 480px) {
  .item {
    flex: 1 1 100%;
  }
}
```



`flex: 1 1 calc(25% - 20px)`理解，flex 是一个符合属性，由三个基本值组成：

+ **flex-grow: 1** 定义项目的放大比例，`1` 表示项目在需要额外的空间时可以放大，占据更多的空间
+ **flex-shrink: 1** 定义项目的缩小比例，`1` 表示项目在需要缩小空间时可以缩小，以适应容器的空间
+ **flex-basis: calc(25% - 20px)** 定义项目在没有额外放大或缩小时的基准大小



```html
<div class="container">
  <div class="item">内容1</div>
  <div class="item">内容2</div>
  <div class="item">内容3</div>
  <div class="item">内容4</div>
  <!-- 更多内容项 -->
</div>
```

