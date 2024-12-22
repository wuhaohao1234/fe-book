CSS Modules 是一种用于 CSS 作用域隔离和模块化的方法，通过自动生成全局唯一的类名，解决 CSS全局命名空间污染问题，使样式在组件之间不会发生冲突。

CSS Modules 既不是一种规范也不是一个 library，而是一种通过构建工具实现将 CSS 文件中的类名和标识符特殊处理的方法

## 和 BEM 区别
CSS Modules 和 BEM 都在试图解决 class 命名冲突问题，两者在实现上有几个区别

+ BEM 命名依赖开发者手工保障，无需构建工具支持；CSS Modules 开发时候无需保障，依赖构建器将 class name 唯一化处理
+  BEM 的 class 可以在 HTML、JSX 中直接使用；CSS Modules 的 class 需要使用 css-loader 处理后的 JavaScript 对象的属性获取

## 工作原理
1. 编写 CSS 文件：开发者编写常规的 CSS 文件（一般使用 xxx.module.css 命名），class name 只需考虑当前模块，不必担心全局冲突
2. 使用构建工具（Webpack 等）处理：在构建过程中，Webpack 的 `css-loader` 解析 CSS 文件，通过配置 `modules` 选项使其生效
3. 生成唯一类名：构建工具会为每个类名生成唯一的哈希值，并替换原来的类名
4. 导入和使用 CSS：在 JavaScript 模块中导入处理后的 CSS 文件，并使用返回的映射对象来应用样式

## demo
### 创建 css 文件
命名使用 xxx.module.css 的约定，这种命名方式有助于明确哪些 CSS 文件应该被处理为模块化的 CSS，从而提升代码的可维护性和可读性

```css
.container {
  padding: 20px;
  background-color: lightblue;
}

.button {
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  background-color: blue;
  color: white;
}

.button_primary {
  background-color: green;
}

.button_secondary {
  background-color: grey;
}
```

### 组件中使用
```jsx
import React from 'react';

// 使用 JavaScript 对象获取构建处理后的 class name
import styles from './MyComponent.module.css';

const MyComponent = () => (
  <div className={styles.container}>
    <button className={`${styles.button} ${styles.button_primary}`}>
      Primary Button
    </button>
    <button className={`${styles.button} ${styles.button_secondary}`}>
      Secondary Button
    </button>
  </div>
);

export default MyComponent;
```

### webpack 配置
```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      {
        test: /\.module\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true, // css-loader 配置
              importLoaders: 1,
              localIdentName: '[name]__[local]___[hash:base64:5]'
            }
          }
        ]
      }
    ],
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 3000,
  }
};
```

## css-loader 作用
`css-loader` 的主要作用是解析和处理 CSS 文件，使其可以作为 JavaScript 模块被导入。它主要包括以下功能：

1. 解析 CSS：将 CSS 文件中的内容解析为一个 JavaScript 对象，包含所有的类规则和样式信息。
2. 处理 `@import` 和 `url()`：在 CSS 中处理 `@import` 和 `url()` 等资源引用，并将这些依赖项作为模块导入。
3. 支持 CSS Modules：可以将 CSS 类名作用域化，避免全局命名冲突。

其实 Webpack 最核心的作用是将各种资源（如 JavaScript、CSS、图片等）视为模块，并通过依赖图将这些模块打包成一个或多个包含所需资源的文件（bundles），以便在浏览器中加载和运行。而 css-loader 就是把 css 文件转为模块的工具

  


