传统的前端开发将样式定义在独立的 CSS 文件中，而 CSS-in-JS 是一种使用 JavaScript 来编写 CSS 的一种模式

## 标签模版字符串
在介绍业界流行解决方案之前需要先了解 CSS in JS 常用的`标签模板字符串`语法

```javascript
tag`string text ${expression} string text`;
```

+ **tag**: 一个函数，用于处理模板字符串
+ **string text**: 模板字符串中的静态部分
+ **expression**: 模板字符串中的插值表达式，它们的值会被传递给标签函数

当使用标签模板字符串时，JavaScript 会自动解析模板字符串，并将静态字符串部分和插值表达式分别传递给 tag 函数

```javascript
function tag(strings, ...values) {
  console.log(strings); // ["Hello, ", "! You have ", " new messages."]
  console.log(values);  // [name, count]
}

const name = "Alice";
const count = 10;

tag`Hello, ${name}! You have ${count} new messages.`;
```

看一个实用的例子

```javascript
function escapeHTML(strings, ...values) {
  return strings.reduce((result, str, i) => {
    // reduce 回调没有设置初始值，strings[0] 做为初始值，i 从 1 开始计数
    let value = values[i - 1];
    if (value) {
      value = String(value)
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;');
    }
    return result + value + str;
  });
}

const userInput = '<script>alert("XSS")</script>';
const message = escapeHTML`<p>User says: ${userInput}</p>`;

console.log(message); // <p>User says: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</p>
```

## styled-components
[styled-components](https://styled-components.com/) 是一个流行的 CSS-in-JS 库，专门用于 React 应用程序，它通过使用标签模板字面量来定义样式

```javascript
import React from 'react';
import styled from 'styled-components';

const Button = styled.button`
  background: ${props => props.primary ? 'palevioletred' : 'white'};
  color: ${props => props.primary ? 'white' : 'palevioletred'};
  padding: 1em;
  border: none;
  border-radius: 3px;
`;

const App = () => (
  <div>
    <Button primary>Primary Button</Button>
    <Button>Default Button</Button>
  </div>
);

export default App;
```

## emotion
[Emotion](https://emotion.sh/docs/introduction) 是一个高性能的 CSS-in-JS 库，提供了灵活和强大的工具，用于在 React 等现代前端框架中编写样式。它支持 styled components 和 css prop 两种语法

### styled components
和 styled-components 的写法几乎可以无痕切换

```javascript
import React from 'react';
import styled from '@emotion/styled';

// 定义静态样式的按钮
const Button = styled.button`
  background: palevioletred;
  color: white;
  padding: 1em;
  border: none;
  border-radius: 3px;
  cursor: pointer;
`;

// 定义动态样式的按钮
const DynamicButton = styled.button`
  background: ${props => props.primary ? 'palevioletred' : 'white'};
  color: ${props => props.primary ? 'white' : 'palevioletred'};
  padding: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
  cursor: pointer;
  margin: 0.5em;

  &:hover {
    background: ${props => props.primary ? '#d66c83' : '#f7e0e4'};
  }
`;

// 定义带有嵌套样式的容器
const Container = styled.div`
  text-align: center;
  margin: 2em;
`;

const ButtonComponent = () => (
  <Container>
    <Button>Default Button</Button>
    <DynamicButton primary>Primary Button</DynamicButton>
    <DynamicButton>Secondary Button</DynamicButton>
  </Container>
);

export default ButtonComponent;
```

### css prop
> <font style="color:rgb(33, 37, 41);">Any component or element that accepts a </font>`<font style="color:rgb(33, 37, 41);">className</font>`<font style="color:rgb(33, 37, 41);"> prop can also use the </font>`<font style="color:rgb(33, 37, 41);">css</font>`<font style="color:rgb(33, 37, 41);"> prop. The styles supplied to the </font>`<font style="color:rgb(33, 37, 41);">css</font>`<font style="color:rgb(33, 37, 41);"> prop are evaluated and the computed class name is applied to the </font>`<font style="color:rgb(33, 37, 41);">className</font>`<font style="color:rgb(33, 37, 41);"> prop.</font>
>

<font style="color:rgb(33, 37, 41);">css prop 需要使用 babel 开启</font>

```javascript
{
  "presets": [
    [
      "@babel/preset-react",
      { "runtime": "automatic", "importSource": "@emotion/react" }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}
```

css prop 有 object styles 和 string styles 两种写法

```javascript
import { css } from '@emotion/react';
import React from 'react';

// 使用 string styles
const staticButtonStyles = css`
  background: palevioletred;
  color: white;
  padding: 1em;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  margin: 0.5em;
`;

//  使用 object styles
const dynamicButtonStyles = (primary) => css({
  background: primary ? 'palevioletred' : 'white',
  color: primary ? 'white' : 'palevioletred',
  padding: '1em',
  border: `2px solid palevioletred`,
  borderRadius: '3px',
  cursor: 'pointer',
  margin: '0.5em',
  '&:hover': {
    background: primary ? '#d66c83' : '#f7e0e4',
  },
});

const containerStyles = css({
  textAlign: 'center',
  margin: '2em',
});

const ButtonComponent = () => (
  <div css={containerStyles}>
  <button css={staticButtonStyles}>Static Button</button>
  <button css={dynamicButtonStyles(true)}>Primary Button</button>
  <button css={dynamicButtonStyles(false)}>Secondary Button</button>
  </div>
);

export default ButtonComponent;
```

### 零运行时模式
对生产环境优化性能，Emotion 提供了零运行时模式（Zero-runtime CSS-in-JS），可以使用 `@emotion/babel-plugin` babel 插件来进行零运行时模式配置：

```javascript
module.exports = {
  presets: ['@babel/preset-react'],
  plugins: ['@emotion']
};
```

  
然后使用 `css` 函数来定义样式：

```javascript
import { css } from '@emotion/react';

const buttonStyle = css`
  background: palevioletred;
  border-radius: 3px;
  border: none;
  color: white;
  padding: 1em;
`;

const Button = () => (
  <button css={buttonStyle}>
    Click Me
  </button>
);

export default App;
```

零运行时模式本质上和 CSS Modules 方案是一致的，Emotion 做的比较好的是允许静态样式和动态样式混合编写，在构建时候只会构建静态的样式，动态样式依旧在运行时依赖 JS 执行

## 优势与不足
CSS in JS 有几个核心优势

+ 生成的样式作用域仅限于对应的组件，防止了全局样式污染和样式冲突问题
+ 可以根据组件的状态或属性动态生成样式，增加了样式的灵活性和可控性
+ 将样式与组件逻辑放在同一个文件中，提升了组件的封装性和复用性
+ 样式按需安装，理论上可以减少冗余 CSS 代码
+ 提供了服务端渲染支持，在服务器端预渲染 CSS，提高首屏渲染性能

同时也有几个不足

+ 在运行时动态生成样式导致额外的性能开销，特别是在大型应用中或者需要频繁更新样式的场景
+ 样式是动态生成调试样式相对困难，在浏览器开发者工具中不易追踪



虽然 CSS in JS 会带来一定的性能开销，但可以作用域隔离和使用动态属性等特性使其特别适合需要切换主题的组件库和中后台场景，Antd 最新版本就采用对 emotion 二次封装的[ antd-style](https://ant-design.github.io/antd-style/zh-CN/guide) 来为组件提供丰富的样式自定义 & 换肤能力

