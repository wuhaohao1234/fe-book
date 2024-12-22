说到性能优化前端同学肯定对[雅虎军规](https://juejin.cn/post/6844903657318645767)特别熟悉，虽然随着网络技术的发展有些规则已经不合时宜，但其核心思想仍然在指导大部分 Web 开发者对页面进行性能优化

+ 尽快建连（使用 CDN、预加载、减少 DNS 查询等）
+ 减少传输内容
+ 复用缓存
+ 静态资源加载顺序调优
+ JavaScript、CSS 语法层面的一些优化

基础优化工作完成后性能优化进入深水区，我们需要对页面加载的去哪过程逐段拆解，一个网页的加载大概有下图过程

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723371759869-b1b260e1-ecf1-46ca-b64b-837d54160355.png)

本文主要使用 Next.js 打造同构 + 流式渲染 + Selective Hydration 的通用 React SPA 性能优化解决方案，加速 RT + 下载 + DOM 渲染时长

## 基础概念
为了实现最终的方案需要先了解一下 demo 中用到的基础概念

### <font style="color:rgb(51, 51, 51);">Google Core web vitals 性能指标</font>
想要从 0 到 1 打造极致性能的 Web 页面，首先需要了解如何衡量页面的性能，什么样的页面算是性能优秀，笔者所在的团队经历过好几个阶段的探索，比如页面 TTFB、主接口达到时间、页面第一个图片被渲染时间等，总会出现指标上很快，但人的体感并不快的情况

这次我们来了解一下 Google 定义的一套以人为本的性能衡量指标—— [Core Web Vitals](https://web.developers.google.cn/explore/learn-core-web-vitals?hl=zh-cn)，也被简称为 CWV，Core Web Vitals 主要关注四个性能指标：

1. **FCP**：First Content Paint，首次内容绘制指标测量页面从开始加载到页面内容的任何部分在屏幕上完成渲染的时间。对于该指标内容指的是文本、图像（包括背景图像）、<svg>元素或非白色的<canvas>元素
2. **LCP**：Largest Content Paint，最大内容绘制指标会根据页面首次开始加载的时间点来报告可视区域内可见的最大图像或文本块完成渲染的相对时间
3. **CLS**：Cumulative Layout Shift，累计布局偏移指标是测量整个页面生命周期内发生的所有意外布局偏移中最大一连串的布局偏移分数（简单理解就是页面是否有布局不稳定导致的抖动）
4. **INP**：Interaction to Next Paint，下次绘制交互指标观察用户与页面进行的所有交互的延迟，并报告所有（或几乎所有）延迟中的最大值，低 INP 意味着页面始终能够快速响应绝大多数用户交互，INP 已在 2024 年 3 月取代首次输入延迟 (FID)，防止页面只是首次相应用户操作快，后续加载延迟高

Core Web Vitals 专注于直接影响用户体验的性能指标，而不仅仅是技术指标。这意味着这些指标是基于用户在浏览网页时的实际感受，而非单纯的载入时间或网络速度等数据，因此上面才说这是一套以人为本的性能衡量指标。而文章接下来要介绍的性能优化手段都是围绕这个目标进行的

### SSR 与同构
**SSR** 全称 Server-Side Rendering，是指在服务器端生成 HTML 内容并将其发送到客户端的过程。在 JSP、PHP 年代是没有 SSR 的概念的，因为所有页面都是 SSR 的

但随着互联网的发展，用户对动态内容和交互体验的需求不断增加，Web 技术从以静态内容为主的 Web 1.0 向交互性更强的 Web 2.0 过渡，Ajax 技术的引入和 JavaScript 能力的增强，开发者开始选择在浏览器中使用 JavaScript 操控 DOM，从而达到改变页面内容而无需重新加载，这种被称为 Client-Side Rendering，也就是 **CSR**，催生了纯在前端渲染的解决方案 SPA（Single Page Application）

但随着对 JavaScript 的依赖增强， JavaScript 脚本的体积日益膨胀， 页面首次加载时用户需要等待 JavaScript 被下载和执行，这可能导致首屏渲染时间较长；同时搜索引擎爬虫在处理 CSR 应用时可能出现无法正确的抓取动态生成内容，因此 CSR 应用的 SEO 效果可能不佳

于是可以把 SSR 的首屏内容直出、SEO 友好和 CSR 流畅的用户操作体验结合起来的**同构**（Isomorphic）技术应运而生。同构是指在客户端和服务器端可以共享相同的 JavaScript 代码，在服务器上使用 JavaScript 渲染 HTML，然后在客户端加载该 HTML，并在客户端继续使用 JavaScript 处理交互，这样页面首次渲染通过 SSR 提供更快的首屏体验，而后续交互则通过客户端渲染，提高了应用的响应性

同构应用主要有三个步骤

1. **初始渲染**: 服务器通过 Node.js 运行 JavaScript，生成 HTML 内容并发送到客户端，用户可以快速看到页面内容
2. **下载 JavaScript**: 在客户端浏览器下载并执行 JavaScript 文件
3. **绑定事件**: 客户端的 JavaScript 会将事件监听器等功能绑定到已渲染的 HTML 元素上，使其变得可交互

服务端返回的 HTML 虽然已经让用户看到其内容，但仍然是静态 HTML，无法响应用户的交互，如同干燥的土壤，而在客户端为 DOM 绑定事件使其可交互如同未干燥的土壤补充水分，使其可以生机勃勃，这个过程有一个非常形象的专业术语——**Hydration**（注水），使静态的服务器渲染内容变得动态和可交互，让页面具有了“生气”

Hydration 的过程不是简单把服务器渲染的内容在客户端重新渲染，这样会造成页面内容的闪烁。`ReactDOM.hydrate` 会遍历这些在服务端生成的 HTML 以确保它们与客户端的 React 组件的输出一致

+ 如果内容一致，React 会使用现有 DOM，并且不会重新渲染该部分
+ 如果不一致，React 会重新渲染那些不一致的部分

一旦 Hydration 完成，React 就会将事件处理函数（如点击事件、输入事件等）绑定到这些组件上，使得内容变得可交互

```jsx
import React from 'eact';
import ReactDOM from 'eact-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(<App />);
```

很多同学把 SSR 等同于同构是不准确的，同构相当于加强版的 SSR

+ SSR 强调的是如何在服务器上生成页面内容，同构是强调共享代码和用户体验在服务器与客户端之间的无缝转变
+ SSR 的关注点更多是如何有效生成初次页面，同构应用不仅关注初始渲染，更重视如何让页面在客户端变得动态和可交互，提供更好的性能和用户体验

### 流式渲染
流式渲染（Streaming Rendering）是指在服务器上生成页面内容时，逐步将已准备好的部分内容立刻发送到客户端，而不是等待页面所有内容全部生成才开始发送，使客户端可以更快的接收数据渲染页面，而不必等待整个页面的内容完全下载，从而实现快速的页面加载和用户可视化体验。这个过程像是水管中的水一样流动起来源源不断，因此被称为流式渲染

流式渲染实际上一个非常古老的技术，早在 HTTP 1.1 规范中就已经引入了 `Transfer-Encoding: chunked` 头字段，允许服务器将响应内容分批返回给客户端。服务器可以在生成响应内容的同时，将其分成小块，逐步传输给客户端，而不是等待所有内容生成完成后再返回

在浏览器端，早期的浏览器（如 Netscape Navigator 和 IE）就已经支持对部分 HTML 内容进行解析和执行。当浏览器接收到服务器返回的部分 HTML 内容时，它可以立即开始解析和执行该内容，而不需要等待所有内容加载完成

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723376892245-3cd5f559-ba57-45cb-af6a-2c3ee968dd80.gif)

```javascript
const http = require('http');

http
  .createServer((req, res) => {
    res.writeHead(200, {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    });
    function renderChunk(chunk) {
      res.write(`<div>${chunk}</div>`);
    }

    renderChunk('Loading...');

    setTimeout(() => {
      renderChunk('Chunk 1');
    }, 1000);

    setTimeout(() => {
      renderChunk('Chunk 2');
    }, 2000);

    setTimeout(() => {
      renderChunk('Chunk 3');
    }, 3000);

    setTimeout(() => {
      renderChunk('done!');
      res.write('</body></html>');
      res.end();
    }, 4000);
  })
  .listen(3000, () => {
    console.log('Server listening on port 3000');
  });
```

##  流式渲染的最后拼图
### SPA 页面的困境
流式渲染可以给页面带来显著的性能优化，虽然是成熟的技术，但在同构的 SPA 页面实现起来却并不简单

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723378838352-204b7496-95df-4c1e-bd9c-1f7a4e8127f6.png)

 demo 简单示意一下

```jsx
function App(props) {
  const { sideBarData, ContentData } = props;
  return (
    <div>
      <Header />
      <div>
        {/* Sidebar 和 Content 需要取数渲染 */}
        <Sidebar /> 
        <Content />
      </div>
      <Footer />
    </div>
  );
}
```

对于需要数据才能被正确渲染的组件在不考虑同构时候，一般通过 useEffect 在组件内部发起异步请求，获取到数据后触发组件 re-render。但在同构场景服务器并不执行 useEffect hook，如果希望在服务端渲染出有最终内容的 HTML 需要做到组件在 render 时候已经获取到了数据

+ 组件通过 props 接收数据
+ 服务端提前获取子组件需要的数据，通过组件 props 传入

```jsx
// server.js
const express = require('express');
const React = require('react');
const { renderToPipeableStream } = require('react-dom/server');
const App = require('./App'); // 引入主组件 App

const app = express();

app.get('*', async (req, res) => {
  res.setHeader('Content-Type', 'text/html');

  const sideBarData = await getSideBarData();
  const ContentData = await getContentData();

  const { pipe, abort } = renderToPipeableStream(
    <App sideBarData={sideBarData} ContentData={ContentData} />, {
    onShellReady() {
      res.statusCode = 200;
      pipe(res);
    }
  });
});

app.listen(PORT, () => {
  console.log(`Server is running on port 3000`);
});
```

这样的实现有两个明显的弊端

1. 组件的数据获取逻辑与组件本身的功能分离，导致二者之间的耦合度较高，可维护性差
2. 整个应用需要数据获取完成后一起 SSR，页面所有内容生成后一块返回给客户端，页面性能取决于取数最慢的接口，流式渲染实际已经失效

### 子组件自取数，并发渲染
想解决上述问题需要

+ 子组件有维护在内部、在服务端可以独立发请求的能力
+ 服务端可以对子组件独立渲染，渲染完成后立刻发送到客户端，不依赖整个应用渲染完成

如果不考虑到服务端因素，应用内子组件独立取数，在等待期间展示 fallback UI 占位、数据完成后触发重新渲染，这不就是 Suspense 嘛（[Suspense 支持异步取数](https://juejin.cn/post/7401042923489345536)）

使用 Suspense 后，组件取数维持在内部

```jsx
// 模拟异步取数
function getSidebar() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(Array.from({ length: 5 }, (_, index) => `item ${index + 1}`));
    }, Math.floor(Math.random() * 5000));
  });
}

const Sidebar = async () => {
  // Next.js 支持这种写法，无需使用 use(Promise) 形式
  const list = await getSidebar();
  return (
    <aside>
      <h2 className="font-semibold">Sidebar</h2>
      <ul>
        {list.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </aside>
  );
};
```

对需要异步取数的子组件在最外层包裹 Suspense，使用 fallback 设置占位符

```jsx
export default function App() {
  return (
    <div className="flex flex-col min-h-screen">
      <Header />
      <div className="flex flex-grow">
        <div className="w-1/6 bg-gray-200 p-4">
          <Suspense fallback={<Loading />}>
            <Sidebar />
          </Suspense>
        </div>
        <div className="flex-grow p-4">
          <Suspense fallback={<Loading />}>
            <Content />
          </Suspense>
        </div>
      </div>
      <div className="flex justify-center h-[60px] bg-slate-500 text-white p-4">
        <Footer />
      </div>
    </div>
  );
}
```

### Selective hydration
做完上面的步骤后服务器在对整个应用渲染时无需在外部取数，需要异步取数的子组件 Suspense 拦截返回 fallback UI 占位，完成取数后触发组件重新渲染，返回真实的 HTML

流式渲染得到完美实现，但因为占位的 fallback UI 已经返回给客户端，等依赖异步的子组件重新渲染返回后在 HTML 的底部，而且因为异步取数不能保证服务器子组件返回的顺序

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1724237871513-34adbd26-7e69-402f-83b6-dc5d183f0346.png)

这时候就需要最后一块拼图 ——  React Selective Dydration 登场了，使用 Suspense 包裹后服务器返回的 fallback UI 带有特殊标记，子组件整体使用注释节点包裹

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723436571764-5b4f700b-f507-417e-a6ff-82477e556bae.png)

这个注释节点有些门道，`<!--$?-->` 表示组件加载中，`<!--$-->`表示加载完成。在每个子组件完成异步取数、渲染后，服务器流式返回的内容由三部分组成

+ script 标签包裹的组件 hydration 需要的数据
+ 组件渲染结果的 HTML，设置为 hidden，并且带有特殊 id
+ 一段自执行 script，用于替换 fallback UI 的 DOM，并对组件注水

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723437028033-1132ecca-c780-4fe1-aa66-dfc191bdd22a.png)

这样无论子组件返回顺序是什么样的，都可以对对应位置的 fallback UI 完成替换和注水，而无需对整个应用重新注水，这就是使用 Suspense 带来的  Selective Hydration

## 方案对 CWV 性能指标的优化
了解 Core Web Vitals 4 个核心指标后，就能发现应用分模块独立取数渲染的重要性了

+ **FCP**：页面使用流式渲染，当不依赖数据的 Header 部分完成渲染后第一时间返回给客户端，让页面可视内容尽早上屏
+ **CLS**：依赖异步数据的模块渲染 fallback UI，在页面展示占位符，真实内容返回后页面不发生布局的抖动
+ **INP**：在客户端启用 Selective Hydration，每个部分渲染完成后独立 hydration，避免整个应用一起 hydra 带来的长任务阻塞用户操作
+ **LCP**：一般而言页面 LCP 元素在主内容区，使用流式渲染不仅可以优化 FCP 指标，原本等待主内荣的客户端空闲时间可以做预载工作
    - 首先在流式渲染页面中，在等待服务武器生成 LCP 所在 HTML 部分时候，前面的 HTML 已经被优先返回，节省了客户端后续解析、执行 LCP HTML的时间
    - 在页面第一段输出设置 dns-prefetch、peconnect 等 meta 标签，对关键域名提前建连
    - 在页面第一段输出中提前返回 LCP 渲染需要的图片、CSS 甚至是 JavaScript 资源，在 LCP 所在的 HTML 被服务器返回后可以第一时间上屏

## Next.js 实现同构 + 流式渲染 + Selective Hydration
了解了借助概念和方案是如何影响性能指标之后终于可以开始写 demo 了，接下来要展示的方案有以下特征，来实现极致性能

+ Header 不依赖首屏取数，首屏 SSR 直出
+ Sidebar 和 Content 依赖首屏取数，服务器首先返回 loading 占位，等数据 ready 后流式输出 SSR 内容，独立注水，杜绝长任务
+ Footer 不依赖服务器取数，但服务器未取数、渲染，在客户端懒加载组件代码和数据，进一步释放性能

### Next.js 简介
[Next.js](https://nextjs.org/) 是一个流行的 React  Web 应用框架，由 Vercel 开发，旨在提高开发者在构建现代 Web 应用时的效率和性能。Next.js 对 RSC（React Server Components）支持的比较完善不仅仅是因为其致力于 React Web 应用，还因为多个 React 核心开发者已跳槽到了 Next.js 母公司 Vercel，这无疑加强了 Vercel 与 React 生态系统的优势

### 初始化项目
```bash
npx create-next-app@latest
```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723345072980-3391c26d-bada-4ab0-9723-f9cbadda29ae.png)

### 创建目录
```bash
├── app
│   ├── components
│   │   ├── Content.jsx
│   │   ├── Footer.jsx
│   │   ├── Header.jsx
│   │   ├── loading.css
│   │   ├── Loading.jsx
│   │   └── Sidebar.jsx
│   ├── globals.css
│   ├── layout.js
│   └── page.js
├── jsconfig.json
├── next.config.mjs
├── package-lock.json
├── package.json
├── postcss.config.mjs
├── README.md
└── tailwind.config.js
```

### 实现子组件
页面主要有 Header、Sidebar、Content、Footer 组成，在 components 文件夹实现一下其功能

#### Header
因为 Next.js 默认开启了 Tailwind，可以通过 class 实现简单的页面样式

```jsx
// app/components/Header.jsx
const Header = () => {
  return (
    <header className="bg-blue-600 text-white p-4">
      <h1 className="text-xl">My Application Header</h1>
    </header>
  );
};

export default Header;
```

#### Sidebar
```jsx
// app/components/Sidebar.jsx
function getSidebar() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(Array.from({ length: 5 }, (_, index) => `item ${index + 1}`));
    }, Math.floor(Math.random() * 5000));
  });
}

const Sidebar = async () => {
  const list = await getSidebar();
  return (
    <aside>
      <h2 className="font-semibold">Sidebar</h2>
      <ul>
        {list.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </aside>
  );
};

export default Sidebar;
```

在 Next.js 中支持[使用 fetch API 获取异步数据](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)，为了简化 demo 在 Sidebar 组件中通过 setTimeout 模拟异步取数，在一个5s 内随机的时间（为了演示和 Content 组件无论什么顺序输出都可以正确处理）返回数据

#### Content
```jsx
// app/components/Content.jsx
function getContent() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('This is the dynamic content data from the server.');
    }, Math.floor(Math.random() * 5000));
  });
}

const Content = async () => {
  const data = await getContent();

  return (
    <main>
      <h2 className="font-semibold">Content Area</h2>
      <p>{data}</p>
    </main>
  );
};

export default Content;
```

Content 和 Sidebar 同样依赖异步数据

#### Footer
```jsx
// app/components/Footer.jsx
const Footer = () => {
  return (
    <footer className="text-center">
      <p>The footer component was loaded asynchronously.</p>
    </footer>
  );
};

export default Footer;

```

#### Loading
```jsx
// app/components/Footer.jsx
const Loading = () => {
  return (
    <div className="flex items-center justify-center h-full">
      <div className="dot w-4 h-4 bg-blue-500 rounded-full animate-bounce mr-2"></div>
      <div className="dot w-4 h-4 bg-blue-500 rounded-full animate-bounce mr-2 delay-200"></div>
      <div className="dot w-4 h-4 bg-blue-500 rounded-full animate-bounce mr-2 delay-400"></div>
    </div>
  );
};

export default Loading;
```

实现一个简单的 Loading 组件，用于 Sidebar 和 Content 的 fallback UI，配合 css 实现简单动效

```css
/* app/components/loading.css */
@keyframes bounce {
  0%,
  20%,
  50%,
  80%,
  100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-15px);
  }
  60% {
    transform: translateY(-7px);
  }
}
.animate-bounce {
  animation: bounce 1s infinite;
}
.delay-200 {
  animation-delay: 0.2s;
}
.delay-400 {
  animation-delay: 0.4s;
}
```

### 使用 Suspense 组装 App
```jsx
// app/page.jsx
import { Suspense } from 'react';
import dynamic from 'next/dynamic';

import Header from './components/Header';
import Sidebar from './components/Sidebar';
import Content from './components/Content';
import Loading from './components/Loading';

// Footer 懒加载，服务端跳过
const Footer = dynamic(() => import('./components/Footer'), {
  ssr: false,
  loading: () => <p>Loading Component...</p>,
});

export default function Home() {
  return (
    <div className="flex flex-col min-h-screen">
      <Header />
      <div className="flex flex-grow">
        <div className="w-1/6 bg-gray-200 p-4">
          <Suspense fallback={<Loading />}>
            <Sidebar />
          </Suspense>
        </div>
        <div className="flex-grow p-4">
          <Suspense fallback={<Loading />}>
            <Content />
          </Suspense>
        </div>
      </div>
      <div className="flex justify-center h-[60px] bg-slate-500 text-white p-4">
        <Footer />
      </div>
    </div>
  );
}
```

因为 Footer 组件在页面底部，外层容器占位避免 CLS，组件本身在客户端异步加载，进一步降低首屏还在需要的静态资源。相对于 [React.lazy ](https://juejin.cn/post/7400566246435897394)使用 Next.js 的`dynamic`（[docs](https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading)）有几个额外的好处

+ 在 Next.js 中使用 React.lazy() 加载的组件，在服务端会直接 SSR，而通过 dynamic() 加载的组件可以通过 ssr option 来控制组件是否在服务端渲染
+ 可以通过 `dynamic` 提供的 `loading` 属性自定义加载或错误状态，无需在外层包裹 Suspense 

### 效果预览
为了更清楚观察异步组件加载，使用 Chrome 对 CPU 4 x slowdown

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723443854184-d3441d41-bd00-46ef-9eb7-25ffa961ddf1.gif)

