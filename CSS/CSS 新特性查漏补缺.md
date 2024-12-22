这些属性其实并没有多么新，但受限于固有的浏览器兼容思维，可能在日常开发中用的不多，随着 IE11 都已经是小众浏览器后，可以大胆用起来了

## scope
[@scope - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@scope)

随着 Chrome 118、Safari 17.4 的支持，有作用域的 CSS 真的来了

```css
@scope [(<scope-start>)]? [to (<scope-end>)]? {
  <rule-list>
}
```

+ 作用域根(<scope-start>)： 定义样式开始应用的节点
+ 作用域限制(<scope-end>)（可选）： 定义不受作用域影响的节点，叫 scope-exclude 更好理解
+ 样式规则（<rule-list>）： @scope 内定义的样式

 在 CSS 内

```css
/* 仅对 .container 元素下元素生效 */
@scope (.container) {
  a { color: blue; }
  img {
    border: 5px solid black;
    background-color: goldenrod;
  }
}
```

在 HTML 标签内使用

```css
<div class="container">
  <style>
    @scope {
      a { color: blue; }
      img {
        border: 5px solid black;
        background-color: goldenrod;
      }
    }
  </style>
</div>
```

## variables
[CSS 变量教程 —— 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2017/05/css-variables.html)

+ var 方法有第二个参数，可以设置默认值 `color: var(---mainColor, #f2f3f7)`
+ 如果变量值是一个字符串，可以与其他字符串拼接 `--foo: '#'var(--num)`
+ 如果变量值带有单位，不能写成字符串 `--foo: 20px;`，而不能写成 `--foo: '20px';`
+ 同一个 CSS 变量，可以在多个选择器内声明。读取的时候，优先级最高的声明生效。这与 CSS 的"层叠"（cascade）规则是一致的
+ CSS 变量结合 media query 可以实现响应式布局

## :has
[:has() - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:has)

:has 选择器让 CSS 有了对父（祖先）元素的筛选能力

```css
div:has(> a) {
  background-color: yellow;
}
```

在这个示例中，所有直接含有<a>作为子元素的<div>会被选中并且背景色被设置为黄色。

## size container queries
[CSS 容器查询 - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_containment/Container_queries)

Container Queries 称为容器查询，作用类似于媒体查询（Media Queries），但它们的作用范围不是 Viewport，而是容器元素。通过容器查询可以基于元素的大小来应用不同的样式

```css
@container (min-width: 200px) {
  .my-component {
    /* 在容器宽度至少为200px时应用的样式 */
  }
}
```

凡是写在 @container 规则中的CSS语句，都会寻找最近的容器元素，并进行匹配。上述代码表示 “当 `.my-component`的宽度达到200像素或更多时，应用括号内的样式”。

除了绝对单位，Container Queries 引入了两个相对单位 cqw（container query width，1cqw等于容器宽度的1%）、cqh（container query height），类似于 vw、wh

## image-set
[image-set() - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/image/image-set)

image-set 允许开发者能够根据用户设备的屏幕分辨率（比如普通屏幕、Retina高清屏幕等）定义不同分辨率的图片资源，以便浏览器选择合适的图片显示。这对于建立响应式图片策略非常有用，可以确保在不同设备上加载适合其屏幕分辨率的图片。

```css
background-image: url("example-image-1x.jpg"); /* 兜底设置 */
background-image: image-set(
  "example-image-1x.jpg" 1x,
  "example-image-2x.jpg" 2x,
  "example-image-3x.jpg" 3x
);
```

在上述例子中，background-image 属性使用了 image-set 函数，它包含三个图片资源，分别用于1倍、2倍和3倍像素密度的屏幕。浏览器会根据设备的像素密度自动选择最合适的图像资源。

## object-fit
[object-fit - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-fit)

对于背景图可以通过 background-size、background-position、background-repeat 等属性，控制图像的布局和展现方式。在很多时候（比如性能优化渲染优先级考虑、商品图 SEO）会有使用 img 标签完成类似效果，object-fit 可以让图片不通过定位和 div 容器直接实现类似效果

```html
<style>
  img {
    width: 300px;
    height: 200px;
    object-fit: cover; /* 图像覆盖整个区域，多余的部分被裁剪 */
  }
</style>

<img src="image.jpg">
```

属性object-fit接受以下值：

+ fill: 这是默认值。替换内容将被拉伸以填充元素的内容框。图像的长宽比可能会改变。
+ contain: 替换内容会按照其原始长宽比例缩放来适应元素的内容框，确保替换内容完全展示在元素内部，不被裁剪。这会导致容器未被完全填充。
+ cover: 替换内容会按照其原始长宽比例被缩放并填充整个内容框，必要时内容也会被裁剪以确保填充整个内容框。
+ none: 替换内容不被拉伸或缩放，会保持原始大小。
+ scale-down: 替换内容的大小是none或contain中较小的那个，意味着内容的尺寸显示不会超过原始大小。

## scroll-behavior
[scroll-behavior - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/scroll-behavior)

scroll-behavior CSS属性指定了当触发滚动时页面如何滚动。它可以应用到整个文档上或者一个滚动容器元素上。这个属性主要接受两个值

1. auto: 这是默认值，元素会立即跳转到目标位置。
2. smooth: 使用这个值时，元素会平滑过渡到目标位置，而不是立即跳转。

```css
html {
  scroll-behavior: smooth;
}
```

在上面的代码中，将整个文档的滚动行为设置为平滑。这意味着当用户点击一个带有锚点的链接或者当使用 JavaScript 的scrollTo方法时，浏览器会平滑地滚动到目标位置，而不是立即跳到该位置。

## css scroll snap
[https://cloud.tencent.com/developer/article/2023709](https://cloud.tencent.com/developer/article/2023709)

CSS Scroll Snap允许开发者创建具有强制性定位停靠点的滚动容器和元素。当用户滚动时滚动位置将自动调整到某个元素的特定点上，就像滚动页面中的轮播图或幻灯片一样。

```css
/* 滚动容器 */
.scroll-container {
  scroll-snap-type: y mandatory; /* 在垂直方向上启用滚动捕捉 */
  overflow-y: scroll; /* 启用竖直滚动条 */
  height: 100vh; /* 容器高度设置为浏览器视口的100% */
}

/* 子元素，它们是滚动容器的直接子元素 */
.snap-element {
  height: 100vh; /* 每个子元素的高度也设置为浏览器视口的100% */
  scroll-snap-align: start; /* 指定每个子元素的起始点作为捕捉对齐点 */
}
```

.scroll-container是一个具有滚动条的容器，它被定义为在垂直方向上有强制滚动捕捉行为。意思是，当用户停止滚动时，滚动容器一定会捕捉到其中一个子元素的起始点（在这个例子中，起始点即为每个子元素的顶部）。每个.snap-element被设置为与视口一样高，并且它的顶部会自动捕捉到滚动容器的边缘。

[使用 CSS Scroll Snap 优化滚动，提升用户体验！-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2023709)

## content-visibility
[https://web.dev/articles/content-visibility?hl=zh-cn](https://web.dev/articles/content-visibility?hl=zh-cn)

content-visibility 是一个CSS属性，其设计用意在于改善长网页的性能和加载速度。它允许浏览器跳过不在屏幕上的元素的渲染工作，直至它们快要进入Viewport 时才开始渲染，从而节省了资源，加快了初次渲染时间。

content-visibility属性接受以下几个值：

1. visible：这是默认值，浏览器会正常渲染元素，就像没有content-visibility属性一样。
2. hidden：当设置为这个值时，元素（以及它的子孙元素）不会被渲染。它的一个关键特性是，如果元素不可见（例如位于视图之外），已经使用的布局空间会被保方留下。
3. auto：这是性能改善最明显的设置。如果元素不在视口中，浏览器会跳过渲染该元素及其内容。一旦元素即将进入视口或其他变化导致元素需要被渲染时，它的内容就会被渲染。这可以大大减少页面的初始加载时间和滚动期间的性能消耗。

content-visibility经常与contain-intrinsic-size属性一起使用；contain-intrinsic-size可以设置在content-visibility为auto或hidden时保留的占位空间的大小。这有助于防止滚动布局发生显著的改变。

```css
.element {
  content-visibility: auto;
  contain-intrinsic-size: 500px 300px; /* 占位空间的宽500px，高300px */
}
```

在上面的代码片段中，.element类的元素将只会在视口内时渲染。当content-visibility设置为auto且该元素不在视图内时，它会保留一个500x300像素的占位空间，避免文档布局发生大幅变动。  


对性能优化感兴趣可以顺便了解一下

+ [loadding="lazy"](https://developer.mozilla.org/zh-CN/docs/Web/Performance/Lazy_loading#%E5%9B%BE%E7%89%87%E5%92%8C_iframe)：浏览器原生支持的懒加载
+ [decoding="async"](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLImageElement/decoding)：异步解码图像，允许在解码完成前渲染其它内容

## flex 和 grid
当然这两个特性一点都不小众、更不新，但在面试环节中会惊讶的发现很多同学无法熟练掌握

> Grid 布局与 Flex 布局有一定的相似性，都可以指定容器内部多个项目的位置。但是，它们也存在重大区别。
>
> Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是一维布局。Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是二维布局。Grid 布局远比 Flex 布局强大。
>

[Flex 布局教程：语法篇 —— 阮一峰的网络日志](http://ruanyifeng.com/blog/2015/07/flex-grammar.html)

[CSS Grid 网格布局教程 —— 阮一峰的网络日志](http://ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)

一行代码实现响应式布局

```css
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); 
```

## baseline 2023
如果留心在 caniuse 或者 MDN 可以看到一些 [baseline 2023](https://web.dev/blog/baseline2023?hl=zh-cn) 的图标

> [Baseline](https://web.dev/blog/baseline-definition-update?hl=zh-cn) 的原始定义是，在所有主流浏览器（Chrome、Edge、Firefox 和 Safari）的现行版本和以往版本中均支持相应功能后，这些功能就会纳入 Baseline。
>

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1716521114578-7b703703-cc70-4ac4-ba78-82337137f127.png)



Google I/O 2024 What's new in the web [https://io.google/2024/explore/6d6b2a35-ae3a-4f73-b40d-9dc4a8cf0df4/](https://io.google/2024/explore/6d6b2a35-ae3a-4f73-b40d-9dc4a8cf0df4/)



