## LCP 元素竟然是 Title 区域
一般情况下页面 LCP 元素是首屏最大的图片，即使托管在了 CDN、并且 preload 图片下载仍需要一定时间，但最近分析团队负责的某个页面时候竟然发现 20% 左右的页面访问 LCP 竟然是一个没有任何图片的文本区域，仅次于页面首屏最大图片

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1702022348549-5ad19d78-eb33-4f20-8f0f-06bb89e1532f.png)

在 webpagetest 测试看 Filmstrip View，确实可以看到文案区域空白时候，下面 DOM 已经被渲染

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1702022901762-1f736e64-7522-49c0-a97c-2eca629db4fd.png)

是什么阻止了页面文案的渲染？

## web font 渲染时机
既然是文本渲染问题很自然联想到字体问题，为了保证品牌一致性和跨平台一致性，网站使用了 web font。web font 有多种格式，在以往是个复杂的选择问题，自从 WOFF2 格式出现后就不用纠结了，WOFF2 使用了 Brotli，因此其压缩效果比 WOFF 高出 30%，从而可以减少下载数据量，从而提升性能，同时 WOFF2 浏览器支持也特别好

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1702025096771-de5adc37-7ceb-47ca-8e8e-f6641a1ae2c5.png)

可以使用 `@font-face` 在页面引入 web font

```css
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 100;
  src: url(https://fonts.gstatic.com/s/roboto/v30/KFOkCnqEu92Fr1MmgVxFIzIXKMnyrYk.woff2) format('woff2');
}
```

可能在大部分人想象中使用 @font-face 后文本的渲染过程是这样的

1. 下载 web font
2. 在下载期间先用备选字体渲染
3. web font 字体下载完成后使用该字体替换

但实际这只是 @font-face 渲染方式的一种，而且不是默认的，@font-face 由 `font-display`属性决定 web font 在下载时间和可用时间是如何展示的

1. **auto**：浏览器使用默认的字体加载策略，不同浏览器处理方式不同，但一般是<font style="color:rgb(51, 51, 51);">有一个非常短暂的阻塞期，如果字体在这段时间内加载完成则使用该字体，否则它将使用备用字体，并且在字体文件加载完成后切换到定义的字体</font>
2. **block**：在字体请求的阶段内文本不可见，只有在字体文件加载完成后文本才会显示。如果在这个阶段（大约为3秒）字体文件没有加载完成，将使用备用字体渲染，并且在加载完成后切换到请求的字体
3. **swap**：在字体文件加载过程中浏览器会立即使用备用字体渲染，一旦字体文件加载完成，浏览器会将文本切换到请求的字体。这能有效避免文本在加载字体文件时不可见，但可能会导致文本样式突然改变
4. **fallback**：结合了block和swap两种情况。在字体加载的前100ms（或者更长，取决于浏览器）内，浏览器会阻塞文本的渲染。之后如果字体尚未加载，将使用备用字体。但与swap不同的是，在备用字体显示的一段时间后，如果请求的字体仍未加载完成，浏览器就不会再执行字体切换，以避免文本样式的突变
5. **optional**：与fallback类似，但它给予浏览器更大的自由度来决定是否下载字体文件。如果字体文件在一定时间内没有加载完成，浏览器可能选择不再加载它。这个值对于用户体验和性能都有可能带来提升，因为它减轻了网络负担，但可能会有字体不一致的风险

在字体下载比较慢的情况下

+ 如果认为字体切换带来的闪烁不是问题，可以设置 `font-display:swap;`提前渲染文本
+ 如果认为不一定要保障使用 web font 来渲染，那么无疑 `font-display:optional;`是最优选

## web font 下载提速
但在大部分网站无论是字体闪烁、还是不使用规定的 web font 渲染都是问题，因此我们的主要思路需要集中在如何提升 web font 下载速度

### 提前加载 web font
令人意外的是浏览器解析到 @font-face 声明时候并不会下载对应的字体，只有解析到<font style="color:rgb(18, 18, 18);">使用了该 font-face 中定义字体的页面元素时，才会下载对应的字体</font>

<font style="color:rgb(18, 18, 18);">这也就意味着把 @font-face 声明写在页面顶部是没有作用的，可以使用 preconnect 或者 preload 提前让浏览器提前建连和下载字体文件，尤其是在页面应用了流式渲染时候有奇效</font>

```html
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preload" href="https://fonts.gstatic.com/s/roboto/v30/KFOkCnqEu92Fr1MmgVxFIzIXKMnyrYk.woff2" as="font" type="font/woff2" crossorigin>
```

### 字体子集化缩减 web font 体积
字体子集化是一种减小字体文件大小的技术，它通过移除字体文件中不需要的字符来实现。由于许多字体文件包含了成千上万的字符，这些字符包括各种语言的字母、符号、标点、数字、空格、换行符以及装饰性和特殊字符。然而一个特定的网站或应用可能不需要这么多字符

在 @font-face 内部可以使用 `unicode-range` 定义字体应用到的 Unicode 字符范围。这样可以将字体文件拆分成多个，当字符在定义的范围内时，浏览器才会下载并使用对应的字体文件

<font style="color:rgb(51, 51, 51);">例如网站主要使用英文，但偶尔包含一些拉丁扩展字符，可以用 unicode-range 来指定只加载这些字符的字体：</font>

```css
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2');
  unicode-range: U+00-FF; /* 基本拉丁字母及扩展 */
}
```

通过将字体文件限制为仅包含网站或应用所需的字符，可以减少文件的大小，得益于上面提到的字体当被使用到才会下载的机制，从而减少下载时间和带宽使用，提高网站的加载速度和性能

Google Roboto 字体的定义 [https://fonts.googleapis.com/css2?family=Roboto](https://fonts.googleapis.com/css2?family=Roboto)

```css
/* cyrillic-ext */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: url(https://fonts.gstatic.com/s/roboto/v30/KFOmCnqEu92Fr1Mu72xKKTU1Kvnz.woff2) format('woff2');
  unicode-range: U+0460-052F, U+1C80-1C88, U+20B4, U+2DE0-2DFF, U+A640-A69F, U+FE2E-FE2F;
}
/* cyrillic */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: url(https://fonts.gstatic.com/s/roboto/v30/KFOmCnqEu92Fr1Mu5mxKKTU1Kvnz.woff2) format('woff2');
  unicode-range: U+0301, U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116;
}
/* greek-ext */
@font-face {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  src: url(https://fonts.gstatic.com/s/roboto/v30/KFOmCnqEu92Fr1Mu7mxKKTU1Kvnz.woff2) format('woff2');
  unicode-range: U+1F00-1FFF;
}
...
```

## 优化效果
为了保障用户体验，选择了对字体文件 preload

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1702034411612-91736494-8c26-4607-a265-b3671d82d36e.png)

这样文本部分终于第一时间加载

![](https://cdn.nlark.com/yuque/0/2023/png/87727/1702034297457-7574d2ba-20cb-486a-9976-d36c83c16a1d.png)

