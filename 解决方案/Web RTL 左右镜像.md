## 前言
做过海外业务同学肯定对 RTL（ Right-To-Left）有所了解，阿拉伯语、希伯来语的的文字书写是从右到左的，为了确保页面在所有语种下需要对 RTL 做好适配工作，我们知道通过 `dir` 属性或者 `direction`样式可以让设置文字书写顺序

```html
<!DOCTYPE html>
<html dir="ltr">
  <body style="direction:rtl;">
    This is a text.
  </body>
</html>
```

其实即使不设置这些属性、样式，当网页上包含阿拉伯语文本时，现代浏览器能够根据字符的 Unicode 范围和语言特性自动判断该文本的方向

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723893868306-5d23c028-96ba-41ee-bdaa-31f90a3aebf2.png)

## 现实的困境
其实对于 RTL 习惯的语种，不仅仅是文字从右到左，而是几乎所有的交互表达都需要做左右镜像表达，即使是一个 loading 的图标的旋转方向在 RTL 下也是逆转的

| ![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723954221998-9416f84f-cae4-4e08-9b15-40afb8b502ed.gif) | --> | ![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723954221767-651f0c43-c2cb-41b3-92b8-5b7cd0b8c2d4.gif) |
| --- | :---: | --- |


因此友好的 RTL 体验需要兼顾到

+ 文字书写顺序
+ 横向排布的图标顺序
+ 返回、前进的图标
+ 新页面加载或者页面后退对应的交互方向
+ 横向滚动条滑动方向
+ 横向排列 Tab Item 顺序及左右方向
+ ...

| ![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723894405673-f7801cb6-d488-4b8a-b6c3-ce8416bd864b.gif) | ![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723894282504-7060eb31-020c-4661-aad3-da2ecb2f6f64.gif) |
| :---: | :---: |


仅仅通过 dir 或 direaction 是做不到的上述的交互左右镜像的，除了文字的方向 CSS 中涉及到左右的属性都需要镜像设置才可以

## 方案一：使用 CSS 逻辑属性实现 RTL
CSS 逻辑属性（Logical Properties）和逻辑值（Logical Values）是为了在不同[书写模式](https://developer.mozilla.org/zh-CN/docs/Web/CSS/writing-mode)（writing mode）之间提供一致的布局和样式控制。通过使用逻辑属性，你可以创建更具适应性的布局，无需根据书写方向手动调整

+ **物理属性**: 传统的 CSS 属性如 `margin-left`, `padding-right`, `top` 等，它们直接指定页面的物理方向（如左、右、上、下）
+ **逻辑属性**: 根据书写模式动态地映射到物理方向，它们提供一种更抽象的方式来定义布局，从而在不同的文本方向下自动适应

### 逻辑属性分类
#### Block 和 Inline 尺寸属性
+ `block-size`: 设置元素在块级方向（通常是垂直方向）的尺寸
+ `inline-size`: 设置元素在内联方向（通常是水平方向）的尺寸

这两个属性在 writing-mode 不设置为垂直书写时对 RTL 无影响，本文不做展开

```css
.element {
  block-size: 50%; /* 在 LTR 模式下等同于 height: 50%; */
  inline-size: 100px; /* 在 LTR 模式下等同于 width: 100px; */
}
```

#### 边距（Margin）、填充（Padding）和边框（Border）
+ `margin-block-start`/`margin-block-end`: 设置元素在块级方向上的起始或结束边距
+ `padding-block-start`/`padding-block-end`: 设置元素在块级方向上的起始或结束填充
+ `border-block-start`/`border-block-end`: 设置元素在块级方向上的起始或结束边框
+ `margin-inline-start`/`margin-inline-end`: 设置元素在内联方向上的起始或结束边距
+ `padding-inline-start`/`padding-inline-end`: 设置元素在内联方向上的起始或结束填充
+ `border-inline-start`/`border-inline-end`: 设置元素在内联方向上的起始或结束边框

常规写法的样式

```css
.element {
  margin-left: 10px;
  padding-right: 20px;
  border-right: 2px solid black;
}
```

使用逻辑属性改写后，当父元素设置 `dir="rtl"`后可以自动 RTL

```css
.element {
  margin-inline-start: 10px;
  padding-inline-end: 20px;
  border-inline-end: 2px solid black;
}
```

#### 边框圆角
+ `border-start-start-radius`: 设置块级起始方向和内联起始方向交汇处的圆角半径（在 LTR 模式下，通常对应左上角）
+ `border-start-end-radius`: 设置块级起始方向和内联结束方向交汇处的圆角半径（在 LTR 模式下，通常对应右上角）
+ `border-end-start-radius`: 设置块级结束方向和内联起始方向交汇处的圆角半径（在 LTR 模式下，通常对应左下角）
+ `border-end-end-radius`: 设置块级结束方向和内联结束方向交汇处的圆角半径（在 LTR 模式下，通常对应右下角）

```css
.element {
  /* LTR 时候左上角为圆角，RTL 时候自动变为右上角为圆角 */
  border-start-start-radius: 50%;
}
```

#### 位置属性
+ `inset-block-start`/`inset-block-end`: 指定元素相对于其包含块在块级方向上的起始或结束位置
+ `inset-inline-start`/`inset-inline-end`: 指定元素相对于其包含块在内联方向上的起始或结束位置

```css
.element {
  position: fixed;
  /* LTR 时候在 ViewPort 右下角，RTL 时候在 ViewPort 左下角 */
  inset-block-end: 10px;
  inset-inline-end: 20px;
}
```

### 问题
看起来使用 CSS 逻辑属性看起来是可行的，不过业界没人用，主要有几个原因

1. 浏览器兼容性：根据 [caniuse](https://caniuse.com/css-logical-props) IE 全部不支持 CSS 逻辑属性，不过考虑到 IE 即将退役问题其实不算大

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723969961881-07826e40-fff0-4076-af08-acc48e85f87d.png)

2. 代码可读性：在项目中混合使用物理属性和逻辑属性，可能会让代码变得不易读和不易维护，很多开发甚至从来没有接触过 CSS 逻辑属性
3. 不支持 flex、grid、transform 等属性方向属性：逻辑属性不能直接控制 `flex-direction`、`align-items`、`transform` 等属性的行为，在这些布局场景下仍需要依赖传统的物理属性手动调整特定样式

第三点最为致命，几乎导致了方案的不可用

## 方案二：transform: scaleX(-1)
说到完全的镜像翻转貌似通过`transform: scaleX(-1)`就可以实现了，几乎没有任何成本，拿网页的一块区域做个测试

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723980334622-e0cafefb-bc34-4a0a-afc7-a1e615d253df.png)

额，明明是英语却一个字也不认识了，因为文案也被镜像反转了，这不是我们预期的效果，可以在文案的父元素上添加一个类反弹翻转

```css
.contains-text {
  transform: scaleX(-1);
}
```

这样文字的问题解决了

| ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723980180143-5a93cc63-a497-4f8b-9fbb-f34ddf2161f3.png) | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723980151485-875709ee-0a1b-4601-aa96-66e8dcc09a2d.png) |
| --- | --- |


左右箭头被翻转符合预期，可是仔细观察却发现怪怪的，调整时间的旋钮位置和表盘内的数字，也就是实际商品图片也被翻转了，这就不对劲了，总不能为了 RTL 扭曲现实

| ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723980639677-33570e97-5c43-44b9-a95b-f9b7fd14d977.png) | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723980659838-625615c8-9431-4ca1-94b6-cc5aaf0f0455.png) |
| ---: | :--- |


实际交互起来问题更大，使用 `scaleX(-1)` 仅仅是视觉上的翻转，元素的实际点击区域和交互行为没有发生改变。总结而言使用 transform 方案有几个问题

+ **文本和图像的误读**：`scaleX(-1)` 会翻转元素的所有内容，包括文本、图像、图标等，虽然这在某些情况下可以有效地翻转布局，但会导致文本和图像也被翻转，这通常不是预期的效果
+ **悬停和焦点效果的错位**：由于元素的实际位置没有变化，悬停、焦点等效果可能会出现位置错位或行为异常
+ **鼠标点击区域的问题**：点击区域仍然位于元素原始的物理位置，而不是翻转后视觉上的位置，这可能导致用户点击无效或者点击不准确
+ **难以扩展和适应变化**：在扩展复杂功能或调整布局时，需要不停的翻转再翻转，需要处理更多的异常情况或逻辑分支，增加代码的复杂性
+ **SEO 影响**：使用 `scaleX(-1)` 的翻转内容可能会影响搜索引擎的索引和排名，因为代码只是改变了视觉显示位置，在 HTML 代码中优先级高的 HTML 出现在了 靠后的位置，对搜索引擎形成误导

## 方案三：为 RTL 适配代码
启示我们很早就知道 RTL 的答案——为所有元素适配一套样式代码就可以了

```css
.element {
  display: inline-block;
  padding-left: 10px;
}

[dir="rtl"] .element {
  display: inline-block;
  padding-right: 10px;
}
```

只不过这样做成本太高了，而且开发过程中很容易忘记某个元素的处理，被我们第一时间拒绝了

### rtlcss 构建时自动生成
由于 RTL css 的样式非常有规则，所以可以在构建代码时使用工具处理，[rtlcss](https://rtlcss.com/index.html) 就是这类工具中的佼佼者，rtlcss 配备了一个 [playground](https://rtlcss.com/playground/) 方便开发者预览器生成的 RTL css

> Instead of authoring two sets of CSS files, one for each language direction. Now you can author the LTR version and RTLCSS will automatically create the RTL counterpart for you!
>

可能各位前端都没意识到有这么多 css 属性和左右方向相关

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723983016843-d6e67f8c-a1a5-44e5-b340-ca711246d7e4.png)

### 通过 webpack 接入
使用 [postcss-rtlcss](postcss-rtlcss) 可以很方便的把 rtlcss 集成到项目中

```javascript
rules: [
    {
        test: /\.css$/,
        use: [
            { loader: 'style-loader' },
            { loader: 'css-loader' },
            {
                loader: 'postcss-loader',
                options: {
                    postcssOptions: {
                        plugins: [
                            postcssRTLCSS(options)
                        ]
                    }
                }
            }
        ]
    }
]
```

### mode
在 options 中各种能力可以对生成代码风格进行调整，其中影响最大的是  `mode`

```css
.test1, .test2 {
    background-color: #FFF;
    background-position: 10px 20px;
    border-radius: 0 2px 0 8px;
    color: #666;
    padding-right: 20px;
    text-align: left;
    transform: translate(-50%, 50%);
    width: 100%;
}
```

**combine**：为 LTR 和 RTL 生成两份独立、完整的代码，这是官方推荐的方式，虽然生成的代码量比较大，但非常安全

```css
.test1, .test2 {
    background-color: #FFF;
    background-position: 10px 20px;
    color: #666;
    width: 100%;
}

[dir="ltr"] .test1, [dir="ltr"] .test2 {
    border-radius: 0 2px 0 8px;
    padding-right: 20px;
    text-align: left;
    transform: translate(-50%, 50%);
}

[dir="rtl"] .test1, [dir="rtl"] .test2 {
    border-radius: 2px 0 8px 0;
    padding-left: 20px;
    text-align: right;
    transform: translate(50%, 50%);
}
```

**override**：只对 RTL 相关的代码提取出来后做 override，代码量较少

```css
.test1, .test2 {
    background-color: #FFF;
    background-position: 10px 20px;
    border-radius: 0 2px 0 8px;
    color: #666;
    padding-right: 20px;
    text-align: left;
    transform: translate(-50%, 50%);
    width: 100%;
}

[dir="rtl"] .test1, [dir="rtl"] .test2 {
    border-radius: 2px 0 8px 0;
    padding-right: 0;
    padding-left: 20px;
    text-align: right;
    transform: translate(50%, 50%);
}
```

我们可以注意到对于源码中的 `padding-right: 20px;` RTL 需要生成两句代码才能做到正确的 override

```css
[dir="rtl"] .test1, [dir="rtl"] .test2 {
  padding-right: 0;
  padding-left: 20px;
}
```

### **<font style="color:rgb(33, 37, 41);">processUrls </font>**自带左右的图片地址处理
使用 rtlcss 不会让图片误翻转，但类似前进、后退的箭头需要左右翻转，这时候可以通过图片地址和 rtlcss 做配合实现

```css
.test1, .test2 {
    background-image: url("./folder/subfolder/icons/ltr/chevron-left.png");
    left: 10px;
}
```

开启 process Urls

```javascript
const options = { processUrls: true };
```

这样就可以在生成的代码中自动修改图片地址中的 left、right

```css
[dir="ltr"] .test1, [dir="ltr"] .test2 {
    background-image: url("./folder/subfolder/icons/ltr/chevron-left.png");
    left: 10px;
}

[dir="rtl"] .test1, [dir="rtl"] .test2 {
    background-image: url("./folder/subfolder/icons/rtl/chevron-right.png");
    right: 10px;
}
```

### 逃生出口
即使使用 rtlcss 在部分场景开发者可以通过注释禁用或者特殊处理源码，介绍几个常用的，完整使用参考[这里](https://www.npmjs.com/package/postcss-rtlcss)

```css
/*rtl:ignore*/      /* 选择器内全部禁用 */
.test1, .test2 {
    text-align: left;
    left: 10px;
}

.test3, .test4 {
    text-align: left;
    /*rtl:ignore*/    /* 下一行禁用 */
    left: 10px;
}

/* 指定 RTL 时候生成的 value */
.test1, .test2 {
    font-family: Arial, Helvetica/*rtl:"Droid Arabic Kufi"*/;
    left: 10px;
}
```

### 内联样式处理
rtlcss 不支持内联样式，或者使用 JavaScript 生成的动态样式，尽量避免在 css 文件外设置方向性相关的样式，可以使用 JavaScript 动态添加 className

### direction 设置
让 RTL 相关的 CSS 生效，需要在 html 节点添加`dir="rtl"`属性，为了让 RTL 用户的页面不发生抖动，添加的时间越早越好

1. **根据用户设置的 cookie 或 App setting**

一般支持多语言的网站都会允许用户选择偏好的语种，信息可以存储在 cookie 中，这样在可以在服务器端识别并设置 dir 属性

```javascript
app.get('/', (req, res) => {
  // 从会话或 cookie 获取用户偏好
  const isRtl = req.session.isRtl || req.cookies.isRtl;
  res.render('index', { isRtl });
});
```

2. **根据 HTTP request header Accept-Language**

现代浏览器会在 HTTP request header 通过 `Accept-Language`携带用户浏览器设置的偏好语种，一定程度上也可以代表用户自身的偏好，在用户第一次访问网站时候可以根据此信息决定 dir 属性的 value

```javascript
app.get('/', (req, res) => {
  // 获取用户的语言设置
  const userLang = req.headers['accept-language'] || 'en';
  const rtlLanguages = ['ar', 'he', 'fa', 'ur']; // 波斯语和乌尔都语也是 RTL 的

  // 检查语言是否属于 RTL 语言
  const isRtl = rtlLanguages.some(lang => userLang.includes(lang));

  // 根据 isRtl 决定使用的模板或样式
  res.render('index', { isRtl });
});
```

3. **在页面顶部执行的 JavaScript**

部分页面可能缓存在 CDN，根据用户请求设置 RTL 属性，那么需要让设置 dir 属性的 JavaScript 尽量在页面内容渲染之前执行

```javascript
const userLang = navigator.language || navigator.userLanguage;
const rtlLanguages = ['ar', 'he', 'fa', 'ur'];
const isRtl = rtlLanguages.some(lang => userLang.includes(lang));

if (isRtl) {
  document.documentElement.setAttribute('dir', 'rtl');
}
```

## 三方库适配
一些广泛使用的优秀 UI 组件已经自己实现了 RTL，在项目中需要对其 exclude

### Tailwind 支持 RTL
#### 带前缀的类名
Tailwind 在 3.0 版本[支持了 RTL](about:blank)，支持的方式也颇为落后

```html
<html dir="rtl">
  <body>
    <div class="group flex items-center">
      <div class="ltr:ml-3 rtl:mr-3">
        <p class="text-sm font-medium text-slate-300">...</p>
        <p class="text-sm font-medium text-slate-500">...</p>
      </div>
    </div>    
  </body>
</html>
```

没错，在源码中把 LTR 和 RTL class 都写上，通过 dir 决定哪个生效，本来就很多 class 的 html 有雪上加霜了。。。

#### CSS 逻辑属性
Tailwind 3.3 支持 CSS 逻辑属性来支持 RTL，添加了很多相关的类名，例如

+ `ms-0`: margin-inline-start: 0px;
+ `me-0`: margin-inline-end: 0px;
+ `ps-0`: padding-inline-start: 0px;
+ `pe-0`: padding-inline-end: 0px;

CSS 逻辑属性虽然本身不好推广，不过在 Tailwind 中难度可以小很多，反正所有类名都是新的，不过局限性依然存在

#### 插件简化
可以通过`[@tailwindcss/rtl](https://www.npmjs.com/package/tailwindcss-rtl)`简化 Tailwind RTL支持，其作用和 rtlcss 类似，将类名下的内容做左右镜像处理，不过它会尽量使用 CSS 逻辑属性

源代码

```css
.container {
  margin-left: 20px;
  padding-right: 30px;
  justify-content: flex-start;
  text-align: left;
}

.button {
  margin-right: 10px;
  padding-left: 20px;
}

.navbar {
  float: left;
}
```

处理后追加内容

```css
.container {
  margin-inline-start: 20px;
  padding-inline-end: 30px;
  text-align: start;
}

[dir="rtl"].container {
  justify-content: flex-end;
}

.button {
  margin-inline-end: 10px;
  padding-inline-start: 20px;
}

.navbar {
  float: inline-start;
}
```

### antd 支持 RTL
相对 Tailwind 而言 antd RTL 对开发者友好了很多，antd 提供了一个全局配置 `ConfigProvider`，通过在应用的根组件中使用 `ConfigProvider`，可以很容易地切换整个应用的 RTL 模式。

```jsx
import { ConfigProvider } from 'antd';
import arEG from 'antd/es/locale/ar_EG'; // 阿拉伯语

<ConfigProvider locale={arEG} direction="rtl">
  {/* 你的组件 */}
</ConfigProvider>
```

在 antd 内部使用多种技术混合实现自动 RTL

+ **CSS-in-JS 运行期替换类名**：antd 有一套和 LTR 对应的 RTL 类名，如果 Provider 传入了 `direction="rtl"`，在运行期这些类会自动以不同的方式添加前缀或后缀，这是 antd 支持 RTL 的主要方式
+ **CSS-in-JS 运行期动态覆盖**：antd 利用 CSS-in-JS 解决方案，允许在组件级别动态生成样式。启用 RTL 模式后 antd 会生成特定的 CSS 规则覆盖默认的 LTR样式，例如`margin-left` 和 `margin-right` 等属性被交换
+ **CSS 逻辑属性**：antd 也会利用 CSS 逻辑属性，例如 `margin-inline-start` 和 `margin-inline-end`等

## 更多特殊处理
[RTL Styling 101 - An extensive guide on how to style for RTL in CSS](https://rtlstyling.com/posts/rtl-styling)

