在 Node.js 中，package.json 文件中的 main、module 和 exports 字段用于定义模块的入口点、导出方式和导出内容，它们在包的导入和导出中起着不同的作用。  


1. [main](https://docs.npmjs.com/cli/v10/configuring-npm/package-json/#main)：
    - 作用：main 字段定义了传统 CommonJS 规范下模块的入口点，即在使用该包的时候默认导入的内容。
    - 例如："main": "index.js" 表示在导入该包时，默认会导入 index.js 文件。
2. module：
    - 作用：module 字段定义了 ES Module 规范下的现代模块入口点，它允许在模块系统支持 ES Module 的环境中，以模块化的方式进行导入。
    - 例如："module": "src/index.mjs" 表示在支持 ES Module 的环境下，会导入 src/index.mjs。
3. exports：exports 字段定义了包的导出方式及导出内容，可以用于自定义包的导出行为，例如指定不同的导出入口点或默认导出不同的内容。优先级高于 main 和 module

```json
{
  "exports": {
    ".": "./index.js",
    "import": "./index.mjs"
  }
}
```

    - `".": "./index.js"` 表示在使用 CommonJS 规范的环境下，会导出 index.js 文件
    - `"import": "./index.mjs"` 表示在支持 ES Module 规范的环境下，会导出 index.mjs 文件

<font style="color:rgb(51, 51, 51);background-color:rgb(247, 249, 253);"></font>

exports 还可以定义子模块

```json
  "exports": {
    ".": {
      "react-server": "./react-dom.react-server.js",
      "default": "./index.js"
    },
    "./client": {
      "react-server": "./client.react-server.js",
      "default": "./client.js"
    },
    "./server": {
      "react-server": "./server.react-server.js",
      "workerd": "./server.edge.js",
      "bun": "./server.bun.js",
      "deno": "./server.browser.js",
      "worker": "./server.browser.js",
      "node": "./server.node.js",
      "edge-light": "./server.edge.js",
      "browser": "./server.browser.js",
      "default": "./server.node.js"
    },
    ...
  }
```

      

```javascript
import ReactDOMServer from 'react-dom/server';
```

