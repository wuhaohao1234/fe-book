[CSS Grid 网格布局教程 —— 阮一峰的网络日志](http://ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)

CSS Grid 布局是一种强大的二维布局系统，能够处理复杂的布局需求。相比于 Flexbox 更适合一维布局（行或列），Grid 布局则能够同时处理行和列的布局问题

### 三列布局，第一列固定宽度
```html
<style>
  .grid-container {
    display: grid;
    /* 第一列固定200px，其他两列平分剩余空间 */
    grid-template-columns: 200px 1fr 1fr;
    gap: 10px;
  }
  
  .item {
    background-color: #4caf50;
    color: white;
    padding: 20px;
    text-align: center;
  }
</style>

<div class="grid-container">
  <div class="item item1">1</div>
  <div class="item item2">2</div>
  <div class="item item3">3</div>
</div>
```

### 响应式卡片布局
卡片宽度在 200px ～ 独占一列之间自适应

```html
<style>
  .grid-container {
    display: grid;
    /* 自适应列宽 */
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); 
    gap: 10px;
  }
  
  .item {
    background-color: #4caf50;
    color: white;
    padding: 20px;
    text-align: center;
  }
</style>

<div class="grid-container">
  <div class="item">Card 1</div>
  <div class="item">Card 2</div>
  <div class="item">Card 3</div>
  <div class="item">Card 4</div>
</div>
```

auto-fill 和 auto-fit 都可以在 repeat() 函数中使用，当网格容器的大小发生变化时，根据可用的空间自动调整网格单元的数量，两者行为有一定的差异

+ `**auto-fill**` 会用空格子填满剩余宽度
+ `**auto-fit**` 则会尽量扩大单元格的宽度

### 带区域命名的复杂布局
![](https://cdn.nlark.com/yuque/0/2024/png/87727/1721548674790-60ddfda0-a2de-4f36-a4ef-ee40eae89c74.png)

```html
<div class="grid-container">
  <header class="header">Header</header>
  <nav class="nav">Navigation</nav>
  <main class="main">Main Content</main>
  <aside class="aside">Sidebar</aside>
  <footer class="footer">Footer</footer>
</div>
```

```css
.grid-container {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav main aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  gap: 10px;
  height: 100vh;
}

.header {
  grid-area: header;
  background-color: #4caf50;
  padding: 20px;
  text-align: center;
}

.nav {
  grid-area: nav;
  background-color: #ff9800;
  padding: 20px;
}

.main {
  grid-area: main;
  background-color: #8bc34a;
  padding: 20px;
}

.aside {
  grid-area: aside;
  background-color: #00bcd4;
  padding: 20px;
}

.footer {
  grid-area: footer;
  background-color: #3f51b5;
  padding: 20px;
  text-align: center;
}
```

## 复杂混合布局
Grid 在实现复杂布局时候无需多层嵌套即可实现

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1721548841597-becc7a43-93b6-4265-bed6-d83f581c37dd.png)

```html
<div class="grid-container">
    <div class="item item1">1</div>
    <div class="item item2">2</div>
    <div class="item item3">3</div>
    <div class="item item4">4</div>
    <div class="item item5">5</div>
  </div>
```

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 四个等宽列 */
  grid-template-rows: repeat(3, 100px); /* 三行，每行100px */
  gap: 10px; /* 单元格间距 */
}

.item {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #4CAF50;
  color: white;
  font-size: 24px;
}

.item1 {
  background-color: #d32f2f; /* 红色 */
  grid-column: 1 / 4; /* 占据第1到第3列 */
  grid-row: 1 / 3; /* 占据第1到第2行 */
}

.item2 {
  background-color: #00796b; /* 绿色 */
  grid-column: 4 / 5; /* 占据第4列 */
  grid-row: 1 / 2; /* 占据第1行 */
}

.item3 {
  grid-column: 1 / 2; /* 占据第1列 */
  grid-row: 3 / 4; /* 占据第3行 */
}

.item4 {
  background-color: #7b1fa2; /* 紫色 */
  grid-column: 4 / 5; /* 占据第4列 */
  grid-row: 2 / 4; /* 占据第2到第3行 */
}

.item5 {
  background-color: #f57c00; /* 橙色 */
  grid-column: 2 / 4; /* 占据第2到第3列 */
  grid-row: 3 / 4; /* 占据第3行 */
}
```

