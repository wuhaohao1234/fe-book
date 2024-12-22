[Vite 下一代的前端工具链](https://vitejs.cn/vite3-cn/)

Vite 通过在一开始将应用中的模块区分为依赖和源码两类，改进了开发服务器启动时间。

+ 依赖 大多为在开发时不会变动的纯 JavaScript。一些较大的依赖（例如有上百个模块的组件库）处理的代价也很高。依赖也通常会存在多种模块化格式（例如 ESM 或者 CommonJS）
+ 源码 通常包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS 或者 Vue/Svelte 组件），时常会被编辑。同时，并不是所有的源码都需要同时被加载（例如基于路由拆分的代码模块）

## 按需编译
Vite 以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 方式提供源码。这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1720924836433-d3e3ee92-c1d5-416b-9a48-19e5a3cc35ee.png)

## 预构建 
首次启动 `vite` 时会对依赖进行预构建

+ **CommonJS 和 UMD 兼容性**: 开发阶段中，Vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM
+ **性能**：Vite 将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能

## 缓存利用
+ **文件系统缓存**：<font style="color:rgb(33, 53, 71);">Vite 会将预构建的依赖缓存到 </font>`node_modules/.vite`
+ **浏览器缓存**：解析后的依赖请求会以 HTTP 头 `max-age=31536000,immutable` 强缓存，以提高在开发时的页面重载性能

<font style="color:rgb(33, 53, 71);">  
</font>

