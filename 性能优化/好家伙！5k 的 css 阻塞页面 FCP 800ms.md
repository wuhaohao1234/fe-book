## TTFB 和 FCP 之间为什么这么大差距
在优化 Alibaba.com PC 新用户首页时候发现一个尴尬的问题，页面经过多轮优化 TTFB 已经相对而言可以接受，海外核心国家 90 分位数可以到 480ms 以内

理论上一个开启流式渲染、结构良好、 SSR 的页面，TTFB 和 FCP 之间的差距是可以控制在 2000ms 以内，而奇怪的是页面的 TTFB 和 FCP 的差距竟然达到 3000ms

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1716803798326-93413f7f-a272-4eb0-9025-9e1071910cf7.png)

也就是 HTML 推送到浏览器后没有第一时间上屏

## 看似没有什么问题的 HTML
那么问题一定是处在页面结构上了，HTML 源码简化之后大概是这样的

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Alibaba.com</title>
  <link rel='preconnect' href='https://s.alicdn.com' />
  <link rel="preload" href="https://s.alicdn.com/LCP图片.jpg" as="image" />
  <link rel="stylesheet" type="text/css" href="https://s.alicdn.com/5k-页面通用样式.css" />
  <link rel="stylesheet" type="text/css" href="https://s.alicdn.com/30k-页面主体内容.css" />
</head>
<body>
  <header>这里是一些没有图片的可视 DOM 元素</header>
  <div class="content">
    ...
    <img src="https://s.alicdn.com/LCP图片.jpg" />
    ...
  </div>
  <footer>...</footer>
  <script src="https://s.alicdn.com/页面通用脚本.js"></script>
</body>
</html>
```

这样的 HTML 结构从性能角度来看是不是没有什么大问题？

## 加个打点直呼好家伙
关键是在国内网络情况下访问直接秒开，无法通过调试和看加载瀑布流直观发现问题，[WebPageTest 测试数据](https://www.webpagetest.org/result/240515_AiDcM2_NV/)也说得过去

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1716809332236-aabfd8c4-9236-44f4-ada1-7eb15c0acdf5.png)

百思不得其解后只能对关键时间进行打点上报，看看能不能从数据上发现什么异常

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel='preconnect' href='https://s.alicdn.com' />
  <link rel="preload" href="https://s.alicdn.com/LCP图片.jpg" as="image" />
  <script>window._timing.css1_start = Date.now();</script>
  <link rel="stylesheet" type="text/css" href="https://s.alicdn.com/5k-页面通用样式.css" />
  <script>window._timing = {};window._timing.css1_end = Date.now();</script>
  <link rel="stylesheet" type="text/css" href="https://s.alicdn.com/30k-页面主体内容.css" />
  <script>window._timing.css2_end = Date.now();</script>
</head>
<body>
  <header>这里是一些没有图片的可视 DOM 元素</header>
  <div class="content">
    ...
    <img src="https://s.alicdn.com/LCP图片.jpg" />
    ...
  </div>
  <footer>...</footer>
  <script src="https://s.alicdn.com/页面通用脚本.js"></script>
  <script>
    // 数据上报伪代码
    report('css1', window._timing.css1_end - window._timing.css1_start);
    report('css2', window._timing.css2_end - window._timing.css1_end);
</script>
</body>
</html>
```

观测数据后瞠目结舌，虽然`5k-页面通用样式.css`size 很小，但下载、解析时间均值（不是 90 分位数）达到 650ms，而`30k-页面主体内容.css`只需要 200ms 以内，两者加载顺序对掉后 css1、css2 的时间没有明显变化

```html
<script>window._timing.css1_start = Date.now();</script>
<link rel="stylesheet" type="text/css" href="https://s.alicdn.com/30k-页面主体内容.css" />
<script>window._timing = {};window._timing.css1_end = Date.now();</script>
<link rel="stylesheet" type="text/css" href="https://s.alicdn.com/5k-页面通用样式.css" />
<script>window._timing.css2_end = Date.now();</script>
```

说明主要时间用在了**首次**和 s.alicdn.com 建连，而不是下载。CSS 的解析会阻塞后面的 DOM 渲染，因此 FCP 元素不能上屏

## 首屏 CSS 内联
了解了原因后对页面进行分析，发现首屏真正依赖的 CSS 代码在 4k 以内，为了避免 CSS 下载、解析对 DOM 渲染的阻塞，决定直接对首屏 CSS 内联，非首屏 CSS 下移到 DOM 之后

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Alibaba.com</title>
  <link rel='preconnect' href='https://s.alicdn.com' />
  <link rel="preload" href="https://s.alicdn.com/LCP图片.jpg" as="image" />
  <style>/*首屏需要的css*/</style>
</head>
<body>
  <header>这里是一些没有图片的可视 DOM 元素</header>
  <div class="content">
    ...
    <img src="https://s.alicdn.com/LCP图片.jpg" />
    ...
  </div>
  <footer>...</footer>
  <link rel="stylesheet" type="text/css" href="https://非首屏css.css" />
  <script src="https://s.alicdn.com/页面通用脚本.js"></script>
</body>
</html>
```

修改上线之后效果显著，在 TTFB 没有变化的情况下海外核心国家 90 分位数  FCP 直接下降 800+ms

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1716811844217-82d40268-862f-485f-b6d8-8a7e6e188d28.png)

## 限定条件
当然因为 Alibaba.com 新用户首页有特殊的情况才能达到如此明显的效果

+ 网速较慢用户第一次访问网站，没有和页面静态资源域名有过历史建连
+ 首屏依赖的 css 要足够小

> [https://web.dev/articles/extract-critical-css?hl=zh-cn](https://web.dev/articles/extract-critical-css?hl=zh-cn)
>
> 为了最大限度地减少首次呈现的往返次数，应力求将首屏内容保持在 14 KB（压缩后）以内。[TCP](https://hpbn.co/building-blocks-of-tcp/) 连接无法立即使用客户端和服务器之间的全部可用带宽，它们都会进行[慢启动](https://hpbn.co/building-blocks-of-tcp/#slow-start)，以避免连接过载，导致连接的数据量超出其可传输量。在此过程中，服务器会使用少量数据开始传输，如果它以完美状态到达客户端，则会在下次往返时将传输量加倍。对于大多数服务器，在第一次往返中可传输的最大数据包为 10 个数据包（约为 14 KB）
>

## 一些感悟
+ 对用户 90 分位数的性能优化策略和 Google CWV 建议的 75 分位数完全不同，很多优化方向需要对 HTTP 协议和浏览器渲染有深入理解才会灵光一现
+ 灵光一现是不靠谱的，性能优化没有银弹，很难根据经验提出通用解对所有场景生效，一定要依赖精准的数据分析来发现盲点
+ 认真对待每一行代码，极致的性能优化要求下 1/3 时间在分析数据，1/3 时间在重构代码，为之前的“差不多”还债



