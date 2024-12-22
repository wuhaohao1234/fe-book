为了减少静态资源体积提高加载性能，前端代码上线前都要构建压缩

+ 删除空白和注释
+ 缩短变量和函数名
+ 提取常量
+ 使用表达式简化代码结构
+ 删除未使用代码

```javascript
function greetUser(name) {
  if (name) {
    console.log("Hello, " + name + "!");
  } else {
    console.log("Hello, guest!");
  }
}

const userName = "Alice";
greetUser(userName);
```

压缩后变成了

```javascript
function a(n){console.log(n?"Hello, "+n+"!":"Hello, guest!")}const b="Alice";a(b);
```

Source map 是一种压缩后代码与源代码的映射文件，即使生产环境中的代码经过了压缩和混淆，开发者也可以通过 Source map 在浏览器的开发者工具中查看和调试未压缩的源代码

当生产环境中的代码出现错误时，Source map 可以帮助开发者准确地定位错误发生的位置，错误堆栈中的行号和列号会指向原始的源代码，而不是压缩后的代码，这使得错误追踪和修复变得更加容易

## Source map 文件构成
一个典型的 source map 文件是 JSON 格式，包含以下信息：

+ **version**: Source map 的版本号
+ **file**: 生成代码的文件名
+ **sources**: 原始源文件的列表
+ **sourcesContent**: 原始源文件的内容
+ **names**: 变量名和属性名的列表，在生成的代码中被压缩或混淆
+ **mappings**: 编码的映射信息，描述生成代码和原始代码各个部分之间的对应关系

```json
{
  "version": 3,
  "file": "index-BhN5xBAf.js",
  "sources": [
    "../../src/main.js"
  ],
  "sourcesContent": [
    "function greetUser(name) {\n  if (name) {\n    console.log(\"Hello, \" + name + \"!\");\n  } else {\n    console.log(\"Hello, guest!\");\n  }\n}\n\nconst userName = \"Alice\";\ngreetUser(userName);"
  ],
  "names": [
    "greetUser",
    "name",
    "userName"
  ],
  "mappings": "ssBAAA,SAASA,EAAUC,EAAM,CAErB,QAAQ,IAAI,UAAYA,EAAO,GAAG,CAItC,CAEA,MAAMC,EAAW,QACjBF,EAAUE,CAAQ"
}
```

## Souece map 生成
Source map 通常在构建步骤中由构建工具生成，配置一般都非常简单

### webpack
```javascript
module.exports = {
  devtool: 'source-map', // 或 'inline-source-map'
};
```

### vite
```javascript
export default defineConfig({
  build: {
    sourcemap: true // 或 'inline'
  }
});
```

### babel
```json
{
  "presets": ["@babel/preset-env"],
  "sourceMaps": true
}
```

### Source map 加载
Source map 是在浏览器加载并解析 JavaScript 文件时被加载的。当浏览器加载 JavaScript 文件时，如果启用了开发者工具，且该 JavaScript 文件末尾包含了一个指向 source map 文件的注释（或通过 HTTP 头指定），浏览器会尝试加载对应的 source map 文件

```json
//# sourceMappingURL=index-BhN5xBAf.js.map
```

在大多数现代浏览器中，source map 通常只有在开发者工具打开时才会被请求和加载，这是因为 source map 文件中包含了用来调试的额外信息，而这些信息通常只在开发阶段有用。在开发者工具中点击查看具体的源代码、设置断点或追踪异常时，浏览器会参考 source map 将映射信息应用于显示原始代码视图

## 安全使用
在生产环境中使用 source map 虽然能极大地帮助调试和问题诊断，但也可能会带来安全风险，例如源码泄漏和内部实现暴露。为了在生产环境中安全地使用 Source map，可以采取以下安全措施：

+ **控制访问**<font style="color:rgb(51, 51, 51);">：将 source map 文件放置在受控的访问路径中，只允许特定的 IP 地址或用户组访问</font>
+ **<font style="color:rgb(51, 51, 51);">仅在特定环境中发布</font>**<font style="color:rgb(51, 51, 51);">：在明确的调试需求环境中才部署 source map，而非公开在所有生产环境中</font>
+ **<font style="color:rgb(51, 51, 51);">动态配置</font>**<font style="color:rgb(51, 51, 51);">：使用环境变量或配置项来控制是否生成和部署 source map</font>

