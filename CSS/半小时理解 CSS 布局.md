CSS 布局是指利用 CSS 属性控制页面元素（element）位置、大小、对齐和显示方式，来对页面进行排版

## 一、inline 元素与 block 元素
布局既然在操作页面的元素，在了解布局技巧之前，先来了解一下 HTML 元素的基本分类，页面常使用元素按照元素本身在文档流中的布局（外部显示类型）可以分为最基础的两类，也就是 display 属性最常见的两个值

+ **inline**：元素宽度和高度通常由内容决定，不可以通过 width、height 属性显式设置。多个 inline 元素会在一行内水平排列，直到一行的空间被填满，然后其内容才会换至下一行显示，常见的有 `span`、`a`、`img`
+ **block**：元素宽度默认是其容器的 100%，高度默认由内容决定，可以通过 width、height 属性设置宽高。block 元素会自动开始于新行，并占满父元素的宽度，即使其内容没有那么宽，常见的有 `p`、`div`



:::info
 其实 display 属性同时影响元素本身在文档流中的布局（外部显示类型），以及它的子元素应该如何排列（内部显示类型）。最典型的值就是 `display: inline-block`，其外部显示为 inline 类型不会独占一行，但内部子元素是 block 类型，可以通过 width、height 属性设置宽高。

inline、block 等相当于内外一致，可以把 `display: inline` 理解为 `display:inline-inline`

:::

## 二、流式布局
流式布局或文档流布局是 CSS 默认的对元素的排版方式，可以把文档想象成一个水缸，页面内的每个元素是一个小木块，按照文档中出现的顺序从水缸底部上浮排列

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700636575809-823f697d-7854-4b62-920c-85566c16a161.png)

```html
<div className="normal-stream">
  <span>inline 元素 1</span>
  <span>inline 元素 2</span>
  <span>inline 元素 3</span>
  <span>inline 元素 4</span>
  <div>block 元素 1</div>
  <span>inline 元素 5</span>
  <span>inline 元素 6</span>
  <div style="width: 150px;">block 元素 2</div>
  <span>inline 元素 7</span>
  <span>inline 元素 8</span>
  <span>inline 元素 9</span>
</div>
```

```css
.normal-stream {
  width: 320px;
}

.normal-stream span,
.normal-stream div {
  padding: 2px;
  color: #fff;
  border: solid 1px #333;
}

.normal-stream span {
  background: orange;
}

.normal-stream div {
  background: green;
}
```

+ 父容器宽度允许的情况下，多个 inline 元素会在同一行内展示
+ 上图中 inline 元素 4 中所示，其内容在一行内不能完整显示，会在多行内衔接显示，而不会整个元素换行。虽然 inline 元素可以设置边框 border，但在布局排版上其占位仅仅是内容部分，不会形成一个独立的矩形空间，所以 inline 元素设置 width、height 无效
+ block 元素即使内容不足以填满文档的一行，也会形成矩形空间独占一行，即使通过 width 属性将其显示内容区域变短（block 元素 2），但仍旧会占据一行的独立空间



这些看起来奇奇怪怪的表现，在了解盒模型的设定后会更容易理解，为了避免引入过多概念，盒模型放到后面介绍

## 三、flex 布局
在 HTML 主要承载图文信息显示的年代，使用 block 元素开辟独立的布局空间，使用 inline 元素填充空间内容，流式布局足以让开发者像排版报纸一样对页面布局。但随着页面信息越来越复杂，多列布局、垂直居中等流式布局越来越力不从心，比如最常见的页面布局

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700637730000-1d3749aa-fcd0-4d8a-b070-8a69a1872107.png)

因为缺少 CSS 官方支持，前端开发者先后探索了 Table 布局、浮动布局、Position 布局等 hack 方案，目前被 CSS3 引入的 Flexbox 布局在浏览器兼容性、功能完备性等方面有明显优势，成为业界布局的主流方案

上面的布局使用 Flexbox 也很容易实现

```html
<div class="flex-container">
  <div class="sidebar">
    Sidebar
  </div>
  <div class="main-content">
    <div class="header">Header</div>
    <div class="content">Content</div>
    <div class="footer">Footer</div>
  </div>
</div>
```

```css
.flex-container {
  display: flex;
  height: 100vh;
}
.sidebar {
  width: 200px;
}
.main-content {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}
.content {
  flex-grow: 1; /* 为了让 Content 区高度充满页面空间 */
}
```

阮一峰老师有一篇特别容易理解的 Flex 布局文章 [Flex 布局教程：语法篇 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)，Flex 的功能极其强大，几乎覆盖了除 Grid 外的所有布局需求，简单介绍几个常用的

### 基础概念
对元素设置 `display: flex;`后元素会形成 Flex 容器，其直接子元素会自动成为 Flex Item。

### 充满可用空间
flex-grow 定义了 Flex Item 如何分配剩余空间，默认值为 0，表示不拓展任何空间。设置为其它数字表示改该 Flex Item 相对于 Flex 容器内其它 Flex Item 拓展的比例

当一个 Flex 容器内仅有一个 Flex Item flex-grow 设置为 1**，**其它 Flex Item 的 flex-grow 为默认值 0 时，设置为 1 的 Flex Item 将在主轴拓展全部可用空间

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700652447864-44e44731-93e1-4dff-999f-a4837c2f1fe3.png)

```html
<style>
  .flex-container {
    display: flex;
  }

  .sidebar {
    width: 200px;
  }

  .main-content {
    flex-grow: 1;
  }
</style>

<div class="flex-container">
  <div class="sidebar">Sidebar</div>
  <div class="main-content">main content</div>
</div>
```

### 垂直居中
vertical-align 属性名字暗示其可能控制所有类型元素的垂直对齐，但实际上它只影响 inline、inline-block、table-cell 元素的对齐。而且也只是将元素相对于它们所在行的基线、顶部、底部或文本的顶部/底部进行对齐，而非容器的高度，因此它不会将元素在包含它们的父容器中垂直居中

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700653765549-6261bc10-284d-492a-b547-9355cb092039.png)

所以如何做到垂直居中在 Flex 出现之前是一道热门的前端面试题，Flex 之后简单了很多，`align-items: center`一个属性搞定

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700654549489-4264e012-4dc4-4c2f-ae8b-42f7cb4ad2ac.png)

```html
<style>
  .flex-container {
    display: flex;
    align-items: center;
    height: 200px;
  }
  
  .centered-item {
    margin: 10px;
    width: 100px; 
  }
</style>

<div class="flex-container">
  <div class="centered-item" style="height: 50px;">
    居中 Item 1
  </div>
  <div class="centered-item" style="height: 150px;">
    居中 Item 3
  </div>
  <div class="centered-item" style="height: 100px;">
    居中 Item 3
  </div>
</div>
```

当然使用 align-item 不仅仅可以做到垂直居中对齐，还可以

+ flex-start：顶部对齐
+ flex-end：底部对齐
+ baseline：第一行文字的基线对齐
+ stretch：**默认值**，如果项目未设置高度或设为auto，将占满整个容器的高度。说实话有点坑，很多时候用了 flex 布局后发现子元素都被拉伸了就是这个原因

[CSS finally adds vertical centering in 2024 | Blog | build-your-own.org](https://build-your-own.org/blog/20240813_css_vertical_center/)

现在（![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724462416588-90457ad8-54fd-4d76-8df0-23c75fb41d74.png) Chrome: 123 | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724462416611-99c98e22-c87a-4e29-bb99-7719e8bfde87.png) Firefox: 125 | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724462416608-b32e6562-a797-410c-8ee0-fee0e28801cf.png) Safari: 17.4）更简单了，`align-content:center;`一行代码支持

```css
<div style="align-content: center; height: 100px;">
  <code>align-content</code> just works!
</div>
```

### 水平居中 & 间距均匀分布
单个元素水平居中很容易实现，但如果多个元素水平居中，且间距均匀分布就非常麻烦了

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700655952999-2cacd8f4-a241-46f9-969a-1643cfe50e0a.png)

这种时候使用 `justify-content: center` 就可以轻松实现

```html
<style>
  .flex-container {
    display: flex;
    justify-content: center;
  }
  
  .centered-item {
    margin: 10px;
    width: 100px; 
  }
</style>

<div class="flex-container">
  <div class="centered-item"></div>
  <div class="centered-item"></div>
  <div class="centered-item"></div>
</div>
```

[阮一峰老师的图示](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)可以生动说明 justify-content 不同值的区别，space-between 和 space-around 在很多时候有奇效

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700656330263-23ee0df5-7047-4a27-97a9-7f1b1a73c528.png)

## 四、元素的尺寸与位置
掌握流式布局和 Flex 布局基本就能应对常见的布局需求了，但为了精准控制元素的尺寸、位置还需要学习大名鼎鼎的盒模型和元素定位

### 盒模型
在默认情况下（`box-sizing:content-box`）block 元素的占位空间由几部分组成

+ 内容区 width、height
+ 内边距 padding
+ 边框 border
+ 外边距 margin

```css
.element {
  margin: 30px;
  border-width: 4px;
  padding: 20px;
  width: 170px;
  height: 170px;
}
```

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700658134659-b8749ad4-6f08-475d-8bf0-f52aaf92e4bc.png)

几个概念都很好理解，但有些反直觉的是 width 和 height 控制的是元素内容区的大小，padding、border 不属于 width 和 height

:::info
不包含 margin 很好理解，外边距声明的是其它元素需要和该元素保持的间距

:::

这意味着如果你给一个元素设置了 width: 100px 和 height: 100px，然后又添加了 border: 10px 和 padding: 20px，实际的元素会占据 160x160 的空间。这种落差感造成了很多开发着的困扰，尤其是在早期 IE 的盒模型实现没有遵从 W3C 定义，而是按照大家预期的方式实现了

为了解决这种困扰，CSS3 引入了 box-sizing 属性，用于指定使用哪种盒模型

+ content-box：默认值，width、height 不包含 border 和 padding
+ border-box：使用早期 IE 盒模型的计算方式，height、width 包含了 border、padding



明显 border-box 的设置更符合 CSS 初学者的认知，所以在很多流行的框架（比如 Antd）中多全局覆盖了 box-sizing 的值为 border-box

```css
.element {
  box-sizing: border-box;
  margin: 30px;
  border-width: 4px;
  padding: 20px;
  width: 170px;
  height: 170px;
}
```

对应的盒模型是这样的表现，宽度变成了 170 - 2*2 - 4*2 = 122

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700658069012-fb3c61ff-a8ba-499d-82ee-a45a75ed4a17.png)

### <font style="color:rgb(51, 51, 51);">position</font>
在默认情况下 HTML 中的元素会按照书写顺序从左到右、从上到下排列，块级元素自动换行，元素在文档流 _**占位空间 **_和 _**视觉显示位置 **_是一致的。通过 position 属性和位移属性（top、right、bottom、left）可以修改元素的在文档流中的占位空间及显示位置，构建更复杂的页面布局

#### 默认行为 static
position 属性的默认值是 static，这时候 top、right、bottom、left 属性不生效，元素文档流中占位空间和其视觉显示空间是一致的

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700719614984-25a4a079-b1a7-4d06-b644-80394c17c287.png)

#### 相对自身定位 relative
设置为 relative 的元素仍然保留在普通文档流中，视觉显示位置可以相对于其在文档流中的初始位置进行偏移，偏移量由 top、right、bottom、left 指定。元素虽然视觉上移动了位置，但其在文档流中占位空间和 static 时候一致，其它元素在布局排版时候仍然认为该元素在原来的位置上

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700720181519-d37359ea-c56a-401c-969d-c55ba32d797a.png)

#### 相对于已定位祖先元素位移 absolute
元素会脱离普通文档流，其视觉显示位置将相对于最近的已定位的祖先元素（非static）进行定位。absolute 定位的元素不占据空间，其它元素在布局排版时会表现得如同它不存在一样

:::info
已定位父元素：元素父元素层层上溯，直到发现祖先元素设置了 position，并且值不是 static 为止，如果没有已定位父元素，会相对于 html 定位

:::

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700722526637-b6810204-5760-4a3c-b6d2-dc2319e821ea.png)

#### 相对于浏览器窗口定位 fixed
元素会脱离普通文档流，并相对于浏览器窗口进行定位显示。无论网页如何滚动它的位置固定不动。fixed 元素定位的元素同样不占据空间，其它元素在布局排版时会表现得如同它不存在一样

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700722548932-836ce34b-7c35-4e62-b357-19443fd568ac.png)

:::info
吸顶横向导航、吸底行动按钮等效果，在 CSS 支持之前类似效果通过 JavaScript 实现，现在可以 `position:sticky;`实现，日常布局使用不多，可以在 [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky) 详细了解

:::

## 五、页面布局思路 —— 从整体到局部
很多同学在第一次开发前端页面时候会遇到一个窘境：学了这么多知识，但依旧做不了复杂的布局。网页布局是一个熟能生巧的过程，但有一个最基本的原则——从整体到局部，这样复杂页面布局就像是画马一样简单了



![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700723431219-0ce26c72-26f1-4a23-9871-23b1e4a5e29b.png)



![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700723426892-2ec1bba6-34a0-40c4-8aed-265df9a5d7bc.png)

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700723445483-5201f887-2a7e-4c90-8d90-5ba3c32eb3a7.png)

来个现实点的例子

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700724164144-0f83d6b4-ca63-48b8-9a7b-02b022352c65.png)

布局过程可以是这样的

:::info
布局拆解的过程和 React 组件抽象极其类似

:::

### 最外层拆解
![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700724391314-ee1d3e9a-dd1c-467b-baed-ff72005527f8.png)

首先确定一个根容器，包含有两个子容器

+ Filter 筛选区
+ List 信息区

因为仅仅是上下排列，使用简单的流式布局就可以

```html
<div class="container">
  <div class="filter"></div>
  <div class="list"></div>
</div>
```

### 对 Filter 区域进行细化
![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700724848816-fe1f28c6-9f79-448c-8277-947542e202ff.png)

```html
<div class="filter">
  <div class="label">
    <span>商机类型</span>
  </div>
  <div class="tag-container">
    <div class="tag">筛选标1</div> 
    <div class="tag">筛选标2</div> 
    <div class="tag">...</div> 
  </div>
</div>
```

lebel 和 tag-container 是左右双栏布局，并且 label 宽度固定，tag-container 宽度充满剩余空间，这是前面提到的 flex 布局中 flex-grow 的用法

而 tag-container 中的多个 tag 横向排列，这种可以简单使用流式布局就可以，为了方便控制边距、高度等可以设置为 inline-block，当然出于垂直居中等考虑也可以应用上文中提到的技巧，继续使用 flex 布局和 align-items

```css
.filter {
  display: flex;
}

.tag-container {
  flex: 1;
}

.tag {
  display: inline-block;
  margin-right: 10px;
}
```

这样 Filter 区域的主体布局就完成了，只剩下一些调整边距和对齐的“细节”

### ListItem 抽离
![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700726085004-ddb48129-1871-4fe7-9c9b-63942093301c.png)

List 区域是由 ListItem 循环组成，而 ListItem 可以从左到右拆分为四个主要区域，其中图片区、联系人、行动点宽度固定，核心信息区充满剩余的空间，又是 flex 布局的典型应用

:::info
当然根据设计稿核心信息区也是固定宽度，如果考虑到响应式其宽度是会变化的

:::

```html
<div class="list">
  <div class="list-item">
    <div class="photo"></div>
    <div class="information"></div>
    <div class="contact"></div>
    <div class="action"></div>
  </div>
</div>
```

```css
.list-item {
  display: flex;
}

.infomation {
  flex: 1;
}
```

### 进一步拆解核心信息区
![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700728271154-a1d15645-a49e-430e-8c52-58d68e313846.png)

有了上面的经验，核心信息区就比较简单了

```html
<div class="information">
  <div class="main"></div>
  <div class="decription"></div>
  <div class="extra">
    <div class="quantity">
      <span>采购数量:</span>
      <span class="bold">100</span>
      <span>Pieces</span>
    </div>
    <div class="location">
      <span>发布地点:</span>
      <span class="bold">England</span>
    </div>
    <div class="datetime">
      <span>发布地点:</span>
      <span class="bold">17 小时前</span>
    </div>
  </div>
  <div class="remain"></div>
</div>
```

### 最终效果
![](https://cdn.nlark.com/yuque/0/2023/jpeg/87727/1700732041091-03880157-552c-47f5-9f10-3ecc0b8575a0.jpeg)

代码做到了整体的布局，没有做对齐、间距等控制

```html
<style>
  .filter {
    display: flex;
  }

  .tag-container {
    flex: 1;
  }

  .tag {
    display: inline-block;
    margin-right: 10px;
  }

  .list-item {
    display: flex;
  }

  .infomation {
    flex: 1;
  }

  .extra {
    display: flex;
  }
</style>

<div class="container">
  <div class="filter">
    <div class="label">
      <span>商机类型</span>
    </div>
    <div class="tag-container">
      <!-- 实际代码中这块是根据数据动态生成的 -->
      <div class="tag">筛选标1</div>
      <div class="tag">筛选标2</div>
      <div class="tag">...</div>
    </div>
  </div>
  <div class="list">
    <div class="list-item">
      <div class="photo"></div>
      <div class="information">
        <div class="main"></div>
        <div class="decription"></div>
        <div class="extra">
          <div class="quantity">
            <span>采购数量:</span>
            <span class="bold">100</span>
            <span>Pieces</span>
          </div>
          <div class="location">
            <span>发布地点:</span>
            <span class="bold">England</span>
          </div>
          <div class="datetime">
            <span>发布地点:</span>
            <span class="bold">17 小时前</span>
          </div>
        </div>
        <div class="remain"></div>
      </div>
      <div class="contact"></div>
      <div class="action"></div>
    </div>
  </div>
</div>
```



再加一些细节就可以完成页面的布局了

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1700733160375-ea180f45-27f3-4d7d-a686-efb670ccdea6.png)

