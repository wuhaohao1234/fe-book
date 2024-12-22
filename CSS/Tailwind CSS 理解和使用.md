[Tailwind CSS](https://tailwindcss.com/) 是一个高效、实用工具优先(Utility-First) 的 CSS 框架，通过预定义的类名快速构建响应式和现代化的用户界面，从而简化了样式的编写过程，提升了开发效率和灵活性。

## 简单 demo
```css
@tailwind base; /* 引入基础样式，包括 CSS 重置和基础样式 */
@tailwind components; /* 引入组件样式，包含预定义的可复用组件 */
@tailwind utilities; /* 引入实用工具类，用于控制布局、排版、颜色等各种样式 */
```



```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tailwind CSS Demo</title>
    <link href="./styles.css" rel="stylesheet">
  </head>
  <body class="bg-gray-100 p-6">
    <div class="max-w-md mx-auto bg-white rounded-xl shadow-md overflow-hidden md:max-w-2xl">
      <div class="md:flex">
        <div class="md:flex-shrink-0">
          <img class="h-48 w-full object-cover md:h-full md:w-48" src="https://placekitten.com/300/200" alt="A cute kitten">
        </div>
        <div class="p-8">
          <div class="uppercase tracking-wide text-sm text-indigo-500 font-semibold">Tailwind CSS Demo</div>
          <a href="#" class="block mt-1 text-lg leading-tight font-medium text-black hover:underline">Simple Demo Example</a>
          <p class="mt-2 text-gray-500">This is a basic example of a card component using Tailwind CSS. It demonstrates how to use various utility classes provided by Tailwind.</p>
        </div>
      </div>
    </div>
  </body>
</html>
```

## class 分类
Tailwind CSS 类名可以分为多个类别，分别用于处理不同的样式需求。按照功能和用途，可以将 Tailwind CSS 类名分为以下几个主要类别: 

+ **布局类(Layout) **: 用于管理页面布局和元素的展示。container、box-border、box-content、block、inline-block、flex、grid、hidden 等。
+ **Flexbox 和 Grid 类(Flexbox & Grid) **: 用于实现响应式的 Flexbox 和 Grid 布局
    - Flex 类: flex-row、flex-col、flex-wrap、items-center、justify-between 等。
    - Grid 类: grid-cols-1、grid-rows-2、gap-4、col-span-2 等。
+ **间距类(Spacing) **: 用于管理内边距(padding) 和外边距(margin) 。p-4、px-2、py-1、m-4、mx-auto、my-8 等。
+ **尺寸类(Sizing) **: 用于设置元素的宽度、高度、最小和最大尺寸。w-1/2、w-full、min-w-0、h-8、h-screen、max-h-full 等。
+ **排版类(Typography)** : 用于管理文本样式，包括字体、颜色和对齐等。text-sm、text-lg、font-bold、text-center、leading-loose、tracking-wide 等。
+ **背景类(Backgrounds) **: 用于设置背景颜色、图片和其他背景属性。bg-blue-500、bg-gradient-to-r、bg-cover、bg-fixed 等。
+ **边框类(Borders) **: 用于管理边框的颜色、宽度和样式。border、border-2、border-red-500、rounded-lg、rounded-full 等。
+ **效果类(Effects)** : 用于添加阴影、透明度和其他视觉效果。shadow-md、opacity-75、blur-sm、filter 等。
+ **过渡类(Transitions & Animation) **: 用于设置过渡和动画效果。transition、duration-500、ease-in-out、animate-spin 等。
+ **变换类(Transforms) **: 用于旋转、缩放、平移和倾斜元素。rotate-45、scale-110、translate-x-4、skew-y-6 等。
+ **交互类(Interactivity) **: 用于控制元素的行为，例如悬停、焦点和指针事件。hover:bg-blue-700、focus:outline-none、pointer-events-none 等。

其总量达上千个，在日常使用需要反复查文档，正所谓工欲善其事，必先利其器，几个 VS Code 插件会让开发者使用 Tailwind CSS 更丝滑

+ [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss): 官方插件，提供了类名补全、类名验证、基础文档等功能
+ [Tailwind Docs](https://marketplace.visualstudio.com/items?itemName=austenc.tailwind-docs): 可以直接在 VS Code 中打开 Tailwind CSS 的官方文档，而不需要切换到浏览器
+ [Tailwind Shades](https://marketplace.visualstudio.com/items?itemName=bourhaouta.tailwindshades): 创建符合 Tailwind CSS 的颜色阴影，并将其直接添加到 tailwind.config.js 中
+ [Tailwind Snippets](https://marketplace.visualstudio.com/items?itemName=Zarifprogrammer.tailwind-snippets): 快速插入常用的 Tailwind CSS 类名和代码结构
+ [Tailwind Fold](https://marketplace.visualstudio.com/items?itemName=stivo.tailwind-fold): 折叠冗长的 Tailwind CSS 类名，使代码更加简洁和可读

## JIT 编译模式
Just-in-Time (JIT) 编译模式是 Tailwind CSS 从版本 2.1 开始引入的重要功能，它通过按需生成 CSS 来提升开发体验和优化性能。JIT 模式在开发过程中即时生成所需的 CSS 类名，而不是预先生成大而全的 CSS 文件，显著减少了输出的 CSS 文件大小，并提供更快的编译速度，其工作原理如下

+ JIT 模式会监视项目中的模板文件，当文件保存时它会即时解析和分析这些文件，找出所有使用的 Tailwind CSS 类名
+ 一旦 JIT 解析到某个类名，它会立即生成对应的 CSS 规则，这意味着在项目运行时，只会生成实际使用到的 CSS 类，而不会生成完整的预定义类名集合
+ JIT 模式可以解析和生成动态类名，这对于使用条件渲染或动态生成类名的应用程序非常有用
+ JIT 编译是渐进式的，即当在代码中添加或更改某个类名时，JIT 会立即更新并编译该类名对应的 CSS 规则，而无需重新生成整个 CSS 文件，这种即时响应极大地提升了开发效率

```javascript
module.exports = {
  mode: 'jit', // 启用 JIT 模式
  purge: {
    content: ['./src/**/*.{js,jsx,ts,tsx}', './public/index.html'],
    options: {
      safelist: [ // 即使在代码中未被直接使用也要保留的类名
        'bg-blue-500',
        'bg-red-500',
        'text-blue-500',
        'text-red-500',
        'sm:text-green-500',
      ],
    },
  },
};
```

## 自定义类
即使使用 JIT 模式在 List 列表类 UI 渲染时候，Tailwind 仍然会因为 List Item 被多次重复导致大量 class name 重复，增加 DOM 体积

```html
<ul>
  <li class="p-2 m-2 bg-gray-100 rounded-lg shadow">Item 1</li>
  <li class="p-2 m-2 bg-gray-100 rounded-lg shadow">Item 2</li>
  <li class="p-2 m-2 bg-gray-100 rounded-lg shadow">Item 3</li>
</ul>
```

可以将这些样式提取到一个自定义类中

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

.list-item {
  @apply p-2 m-2 bg-gray-100 rounded-lg shadow;
}
```

然后在 HTML 或 JSX 中使用这个自定义类

```html
<ul>
  <li class="list-item">Item 1</li>
  <li class="list-item">Item 2</li>
  <li class="list-item">Item 3</li>
</ul>
```



当然使用组合类名也可以达到类似效果

```jsx
function App() {
  const items = ["Item 1", "Item 2", "Item 3"];
  const baseClass = "p-2 m-2 bg-gray-100 rounded-lg shadow";

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index} className={baseClass}>{item}</li>
      ))}
    </ul>
  );
}
```

## 适合场景
使用 Tailwind CSS 会带来几个显著的变化

+ HTML 文件中已经包含了样式
+ 使用原子化 class，不用再思考类名
+ 通过 JIT 可以达到类似 ES Module Tree Shaking 的效果，减少 CSS 体积

使用 Tailwind 之后是更好的实践的关注点分离还是相反？在 React 作者早期解释 JSX 是更好的关注点分离实践时候提到过一些观点

> <font style="color:rgb(0, 0, 0);">React 认为渲染逻辑本质上与其他 UI 逻辑内在耦合，比如在 UI 中需要绑定处理事件、在某些时刻状态发生变化时需要通知到 UI，以及需要在 UI 中展示准备好的数据。</font>
>
> <font style="color:rgb(0, 0, 0);">React 并没有采用将</font>_<font style="color:rgb(0, 0, 0);">标记(HTML) 与逻辑(JavaScript) 分离到不同文件</font>_<font style="color:rgb(0, 0, 0);">这种人为的分离方式，而是通过将二者共同存放在称之为“组件”的松散耦合单元之中，来实现</font>[<font style="color:rgb(0, 0, 0);">关注点分离</font>](https://en.wikipedia.org/wiki/Separation_of_concerns)<font style="color:rgb(0, 0, 0);">。</font>
>

也就是说 React 作者认为在 Web Page 开发中关注点应该是独立逻辑的组件，而不是技术文件类型(HTML、CSS、JavaScript) ，按照这个理论使用 Tailwind CSS 无疑可以使用组件逻辑更加内聚，每个组件封装了与自身相关的所有内容，包括样式、行为和显示逻辑。组件之间减少了依赖，这样可以轻松替换或修改单个组件而不影响其他部分



无业务语义的 CSS 究竟是更好维护还是更难维护？在不同的场景显然有不同的答案，如果接手十年老代码，个人更喜欢 Tailwind 风格的原子化类，这样可以放心修改，不用担心关联你影响。个人理解 Tailwind 在几个场景中特别适合

+ 有明确设计规范约束的组件库，不允许开发自由发挥样式修改，使用预定义的类可以更好的保障设计规范的落地
+ 有定制主题诉求的场景，因为开发者直接使用原子化的 class， 因此可以通过配置文件一次性修改所有颜色、字体、间距等
+ 快速构建页面原型和设计，因为可以直接在 HTML 中使用类名来定义样式，而不需要在单独的 CSS 文件中编写大量样式规则，也不用考虑样式复用等，迭代效率会明显提升



