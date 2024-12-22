模块按需加载又称为懒加载，是在在需要时才加载模块，而不是在应用程序的初始加载时加载所有模块，通过按需加载，可以减少初始加载时需要下载的 JavaScript 文件的大小，从而加快页面加载速度

然而 webpack 带来的静态资源 bundle 理念将所有模块捆绑进一个文件中（除非拆分多个 entry），让按需加载的代码拆分有些困难。随着 ES6 的引入动态导入（Dynamic Import）机制，开发者可以不再依赖静态 import 语法，使用动态导入配合 webpack + React lazy、Suspense 特性，带来了模块按需加载的丝滑开发体验

## 基础用法
ES6 的动态导入（Dynamic Import）使得在代码运行时可以按需加载模块。传统的模块导入是静态的，所有依赖项在编译时就确定，而动态导入则在运行时决定加载哪些模块

动态导入使用 `import()` 函数，可以在任意地方使用，该函数返回一个 Promise，当模块成功加载时，Promise 解析为该模块的导出内容

```tsx
import('./MyComponent.js').then(module => {
  const MyComponent = module.default;
  // 使用组件
});
```

**React.lazy：**允许定义懒加载的组件。当组件被渲染时，React 会在后台加载其对应的 JavaScript 文件

```tsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

lazy 的组件需要配合 Suspense 使用，**Suspense：**提供了一个“fallback”界面，在懒加载的组件还未完全加载完成时显示，当组件完成加载后重新渲染组件内容

```tsx
<Suspense fallback={<div>Loading...</div>}>
  <LazyComponent />
</Suspense>
```

## 实例 demo
创建目录结构

```plain
src
├── components
│   ├── AsyncComponent.tsx
│   └── SyncComponent.tsx
└── App.tsx
```



```tsx
// components/AsyncComponent.tsx
export default function AsyncComponent() {
  return <div>This is a lazily loaded component</div>
}

// components/SyncComponent.tsx
export default function SyncComponent() {
  return <div>This is normal component</div>
}

// App.tsx
import { lazy, Suspense } from 'react';
import SyncComponent from './components/SyncComponent';

const AsyncComponent = lazy(() =>
  new Promise(resolve => {
    // 放大异步加载过程，方便观测效果
    setTimeout(() => {
       resolve(import('./components/AsyncComponent'));
    }, 2000);
  })
);

const App = () => {
  return (
    <div className="content">
      <Suspense fallback={<div>loading...</div>}>
        <AsyncComponent />
      </Suspense>
      <SyncComponent />
    </div>
  );
}
```

[此处为语雀卡片，点击链接查看](https://www.yuque.com/sunluyong/fe-interview/nqtgkulhp3rfvbdc#fR1up)

## dynamic import 文件名
在 `React.lazy` 动态导入的组件，在 webpack 等打包时候会自动创建一个和原文件名很相关的新的文件名称

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723096784643-170422f8-261f-4f50-a889-021b74425db0.png)

可以在 `import()` 函数的路径中加入`webpackChunkName`注释，这样 webpack 等构建工具会根据注释生成指定的文件名，这种做法可以帮助开发者在调试时更好地理解代码

```tsx
// 使用注释命名懒加载的组件
const LazyComponent = lazy(() =>
  import(/* webpackChunkName: "LazyComponent" */ './components/AsyncComponent')
);
```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723096687687-d8817bb2-0384-4560-bdc6-dfc41777fb70.png)

除了 webpackChunkName 注释，还有几个有用的

+ `webpackPrefetch: true`父组件加载完成后开始加载
+ `webpackPreload: true`和父组件并行加载

## dynamic import 静态资源地址
除了根据 webpackChunkName 生成文件名，在使用动态导入（dynamic import）时，组件的静态资源地址是由构建工具（如 Webpack、Rollup、Vite 等）决定的，以 webpack 为例

如果 `publicPath` 被设置，webpack 会使用 `publicPath` 来生成请求 base 地址

```tsx
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/assets/', // 所有文件的公共路径
  },
  mode: 'development',
};
```



如果需要使用特定地址 `__webpack_public_path__` 被动态更改，那么在应用运行时动态导入请求将自动使用该路径

```tsx
__webpack_public_path__ = 'https://cdn.example.com/assets/';

document.getElementById('loadButton').addEventListener('click', () => {
  import('./lazyComponent.js')
    .then(module => {
      const LazyComponent = module.default;
      LazyComponent(); // 调用懒加载组件的功能
    })
    .catch(err => {
      console.error('Error loading the component:', err);
    });
});
```

当点击按钮后，浏览器将请求的资源路径会是 `https://cdn.example.com/assets/lazyComponent.js`

## 如何实现的
+ 当 webpack 解析到 `import()`时会自动进行代码分割
+ `import()`会返回一个 Promise，当模块成功加载时，Promise 解析为该模块的导出内容
+ `React.lazy()`返回 LazyComponent 类型的组件，throw import() 返回的 Promise
+ `Suspense`  catch error 的类型如果是 Promise，对该 Promise 注册 .then 和 .catch，当 Promise 状态发生变化时触发组件 re-render
    - 如果状态是 pending，使用 fallback 渲染占位组件
    - 如果状态是 fulfilled，使用 resolve 的组件正常渲染
    - 如果状态是 rejected，继续传递 error 给 ErrorBoundary 处理

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723114145644-056aa93b-9900-4e34-bc81-eca82d817a0a.png)



