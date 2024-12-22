在 Webpack 生态系统中，有多个常见的 CSS 相关加载器（loaders），用于处理 CSS 文件，使其可以在项目中顺利引用和使用

## less-loader
less-loader 的作用是将 less 语法转化为 css 语法

## sass-loader
sass-loader 用于处理 scss、saqss 语法为 css 语法，值得注意的是过往 sass-loader 需要预装 node-sass，`node-sass` 是 LibSass 的 Node.js 绑定，通过调用 C++ 编写的 LibSass 库，将 Sass/SCSS 文件编译为 CSS，这同事带来 node-sass 对 Node.js 版本依赖问题，官方推荐使用 dart-sass 替换

```bash
npm install --save-dev sass sass-loader css-loader style-loader

```



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
        test: /\.s[ac]ss$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'sass-loader',
            options: {
              implementation: require('sass'), // 使用 Dart Sass
            },
          },
        ],
      },
    ],
  },
};
```

## css-loader
`css-loader` 的主要作用是解析和处理 CSS 文件，使其可以作为 JavaScript 模块被导入。它主要包括以下功能：

1. 解析 CSS：将 CSS 文件中的内容解析为一个 JavaScript 对象，包含所有的类规则和样式信息。
2. 处理 `@import` 和 `url()`：在 CSS 中处理 `@import` 和 `url()` 等资源引用，并将这些依赖项作为模块导入。
3. 支持 CSS Modules：可以将 CSS 类名作用域化，避免全局命名冲突。

## style-loader
`style-loader` 的主要作用是将解析后的 CSS 样式插入到 DOM 中的 `<style>` 标签中，从而在网页中生效。它通常与 `css-loader` 连用。

1. 注入样式：将解析后的 CSS 样式作为 `<style>` 标签插入到网页的 `<head>` 中。
2. 支持热替换：在开发环境中支持 CSS 的热替换（HMR），可以即刻看到样式变更。

