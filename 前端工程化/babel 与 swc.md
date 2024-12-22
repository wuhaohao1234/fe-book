Babel 和 SWC 是两个广泛使用的 JavaScript/TypeScript 编译器，用于将现代 JavaScript 代码转换为兼容性更高的代码，以支持旧版本的浏览器或运行环境

## babel
[https://babeljs.io/docs/](https://babeljs.io/docs/)

### tooling packages
+ @babel/core：Babel 的核心库，包含了 Babel 的主要编译功能。负责解析、转换和生成代码，本身不包含任何插件或预设，而是通过加载用户指定的插件和预设来实现代码转换
+ @babel/runtime：提供一系列辅助函数和运行时工具，避免编译过程中对辅助代码的冗余引入，从而减少重复代码，并提供对一些高级 JavaScript 特性（如 async/await，生成器等）的运行时支持
+ @babel/parser：JavaScript 解析器，用于将 JavaScript 源代码转换为抽象语法树（AST）
+ @babel/travers：用于遍历和修改抽象语法树的工具，允许开发者对 AST 进行各类操作，如代码分析、代码转换和优化等
+ @babel/types：提供一系列用于创建和操作 AST 节点的函数，极大地简化了编写代码转换和代码分析插件的复杂度
+ @babel/generator：将抽象语法树 (AST, Abstract Syntax Tree) 转换回代码的工具
+ @babel/code-frame：在编译错误或警告信息中高亮显示代码片段，并指明具体的错误位置
+ @babel/template：生成预定义的抽象语法树模板，简化编写和生成代码片段的过程

### integration packages
+ @babel/cli：在命令行中使用 Babel 来编译 JavaScript 代码
+ @babel/polyfill：引入需要的 JavaScript Polyfills 和 runtime 支持（7.4.0  不再使用，开始使用 core-js、regenerator-runtime）
+ @babel/plugin-transform-runtime：通过引用`@babel/runtime`提供构建辅助代码，缩减包体积，优化一些高级语法特性的实现方式
+ @babel/register：在 Node.js 环境中动态编译和运行 JavaScript 文件，拦截 `require` 调用，实时编译加载的 JavaScript 文件。这样文件在被加载时会自动进行 Babel 编译
+ @babel/standalon：支持在浏览器中直接编译和执行现代 JavaScript 代码，而不需要通过 Node.js 或构建工具进行预编译

### presets
+ @babel/preset-env：将最新的 JavaScript 语法构建为浏览器可兼容的 JavaScript
+ @babel/preset-react：将 React JSX 语法构建为 JavaScript 语法
+ @babel/preset-typescript：将 TypeScript 语法构建为 JavaScript 语法

## SWC:
[https://swc.rs/](https://swc.rs/)

SWC (Speedy Web Compiler) 使用 Rust 编写，旨在提供超快的 JavaScript 和 TypeScript 编译速度，同时保持较低的内存占用。SWC 可用于替代 Babel 和 Terser 等工具，提升编译和构建任务的效率。虽然也支持 bundler，但目前还不太完善，目前没有替代 webpack 的趋势，更多是集成在 webpack 中使用

+ @swc/cli： SWC 提供的命令行工具，用于快速方便地在命令行中执行 SWC 编译任务
+ @swc/core：SWC 的核心库，用于将 SWC 的编译和转换功能集成到 JavaScript 项目中
+ @swc/wasm：SWC 提供的一个 WebAssembly 模块，用于在 Web 环境和 Node.js 环境中执行 SWC 的编译和转译功能
+ @swc/jest：用于在 Jest 测试框架中集成 SWC 编译器的适配器，利用 SWC 的高性能编译功能来加速 Jest 测试
+ swc-loader：用于在 Webpack 中集成 SWC 编译器的加载器 ，利用 SWC 的高性能编译功能来加速 Webpack 的构建过程

```typescript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.ts',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js']
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'swc-loader',
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './dist/index.html',
      inject: 'body'
    })
  ],
  devtool: 'source-map',
  devServer: {
    contentBase: './dist',
    hot: true
  }
};
```



虽然 `swc-loader` 会使用默认的配置，但可以通过创建 `.swcrc` 文件进行自定义配置。

```typescript
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true,
      "dynamicImport": true
    },
    "target": "es2015",
    "loose": false,
    "minify": {
      "compress": true,
      "mangle": true
    },
    "transform": {
      "react": {
        "refresh": true,
        "runtime": "automatic"
      }
    }
  },
  "module": {
    "type": "commonjs",
    "strict": true,
    "strictMode": true
  },
  "sourceMaps": true
}
```

