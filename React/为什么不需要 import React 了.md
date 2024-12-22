在 React 17 之前，每个使用 JSX 的文件都需要显式导入 React。这是因为 JSX 代码在编译时会被转换成 `React.createElement` 调用，因此需要 React 在作用域中

```javascript
import React from 'react';

function MyComponent() {
  return <div>Hello, world!</div>;
}
```

在编译后，JSX 会被转换成：

```jsx
import React from 'react';

function MyComponent() {
  return React.createElement('div', null, 'Hello, world!');
}
```

## React 17 后无需引用 React
React 17 引入了一种新的 JSX 转换机制，可以避免开发者在每个使用 JSX 的文件中显式导入 React，新的转换机制会自动为 JSX 代码插入必要的导入

```jsx
// 不需要导入 React 库
function MyComponent() {
  return <div>Hello, world!</div>;
}
```

使用新转换方式的现代 JavaScript 工具链会自动进行如下处理：

```jsx
import { jsx as _jsx } from 'react/jsx-runtime';

function MyComponent() {
  return _jsx('div', { children: 'Hello, world!' });
}
```

## 为什么做这个改动
传统上 JSX 和 React 是紧密绑定的，使用 JSX 意味着必须引入 React。新的 JSX 转换机制通过独立的运行时（如 `react/jsx-runtime`），使 JSX 的使用不再依赖整个 React 库，这为其他库和框架采用 JSX 提供了更多可能性。

Preact 可以非常方便地适配新的 JSX 转换机制，这样开发者可以更轻松地将现有的 React 代码移植到 Preact 中

```jsx
/** @jsx h */
import { h, render } from 'preact';

const App = () => <div>Hello, World!</div>;

render(<App />, document.body);
```

类似的，其他虚拟 DOM 库（如 Inferno）也可以利用新的 JSX 转换运行时来支持 JSX，使得开发者可以在这些库中使用熟悉的 JSX 语法：

```jsx
/** @jsx createElement */
import { createElement, render } from 'inferno';

const App = () => <div>Hello, World!</div>;

render(<App />, document.getElementById('root'));
```

## 构建配置
无论是哪个库，如果希望使用新的 JSX 转换机制，需要在编译器（如 Babel、TypeScript）中进行适当的配置

### Babel 配置
```json
{
  "presets": [
    ["@babel/preset-react", {
      "runtime": "automatic"
    }]
  ]
}
```

### TypeScript 配置
```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

  
  


