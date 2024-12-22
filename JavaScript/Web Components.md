Web Components 是浏览器原生支持的 Web 标准，允许开发者创建可重用、封装的自定义 HTML 元素。这些元素具有跨框架兼容性，可以无缝集成到原生 HTML、React、Vue 和 Angular 等各种 Web 开发环境中

## 三个关键技术
### Custom Elements
<font style="color:rgb(51, 51, 51);">Custom Elements 允许开发者定义自定义的 HTML 标签，并为其附加特定的行为。这些自定义标签可以像原生 HTML 元素一样使用，并接受属性和事件</font>

+ **<font style="color:rgb(51, 51, 51);">定义元素</font>**<font style="color:rgb(51, 51, 51);">：通过 JavaScript 的类继承 </font>`<font style="color:rgb(51, 51, 51);">HTMLElement</font>`<font style="color:rgb(51, 51, 51);">，并在其中定义元素的特定行为</font>
+ **<font style="color:rgb(51, 51, 51);">注册元素</font>**<font style="color:rgb(51, 51, 51);">：使用 </font>`<font style="color:rgb(51, 51, 51);">customElements.define()</font>`<font style="color:rgb(51, 51, 51);"> 方法注册自定义标签，并绑定相应的类</font>

```javascript
class MyElement extends HTMLElement {
  constructor() {
    super();
    // 元素初始化相关代码
  }

  connectedCallback() {
    // 当元素被添加到文档流中时的行为
    this.innerHTML = `<p>Hello, Web Components!</p>`;
  }
}

customElements.define('my-element', MyElement);
```

### <font style="color:rgb(51, 51, 51);">Shadow DOM</font>
<font style="color:rgb(51, 51, 51);">Shadow DOM 提供了一种将组件的样式和结构进行封装和隔离的方法。通过使用影子 DOM，组件可以避免样式污染或被外部影响</font>

+ **<font style="color:rgb(51, 51, 51);">封装</font>**<font style="color:rgb(51, 51, 51);">：Shadow DOM 为组件创建了一个私有的子 DOM 树，外部页面的 CSS 不会影响到其样式</font>
+ **<font style="color:rgb(51, 51, 51);">隔离</font>**<font style="color:rgb(51, 51, 51);">：Shadow DOM 中的样式和模板可以被完全封装，直至需要公开的属性</font>

```javascript
class MyElement extends HTMLElement {
  constructor() {
    super();
    // 创建影子 DOM 根节点
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.innerHTML = `<style>p { color: red; }</style><p>Hello, Shadow DOM!</p>`;
  }
}

customElements.define('my-element', MyElement);
```

当 mode 被设置为`open`时，创建的 Shadow DOM 可以通过 JavaScript 直接访问和操作

```javascript
// 创建一个影子 DOM
const shadow = this.attachShadow({ mode: 'open' });

console.log(this.shadowRoot); // 访问影子 DOM
```

在某些场景中，开发者可能需要更高程度的封装和安全性，不希望外部代码访问和操作影子 DOM。在这种情况下可以选择关闭模式

```javascript
const shadow = this.attachShadow({ mode: 'closed' });
console.log(this.shadowRoot); // 这将输出 null
```

### HTML Templates
<font style="color:rgb(51, 51, 51);">HTML Templates 不会在页面加载时直接显示，在需要时通过 JavaScript 克隆和使用这些模板内容</font>

+ **<font style="color:rgb(51, 51, 51);">模板定义</font>**<font style="color:rgb(51, 51, 51);">：在 </font>`<font style="color:rgb(51, 51, 51);"><template></font>`<font style="color:rgb(51, 51, 51);"> 元素中定义模板，不显示在文档但可以进行克隆</font>
+ **<font style="color:rgb(51, 51, 51);">内容克隆</font>**<font style="color:rgb(51, 51, 51);">：通过 </font>`<font style="color:rgb(51, 51, 51);">template.content.cloneNode(true)</font>`<font style="color:rgb(51, 51, 51);"> 方法克隆模板内容并插入到 DOM 中</font>

```html
<template id="my-template">
  <style>p { color: blue; }</style>
  <p>This is a template paragraph!</p>
</template>

<script>
  const template = document.getElementById('my-template');
  const content = template.content.cloneNode(true);
  document.body.appendChild(content);
</script>
```

## 简单计数器组件示例
```javascript
class SimpleCounter extends HTMLElement {
  constructor() {
    super();
    this.count = 0;
    
    // 创建一个影子 DOM 根节点
    const shadow = this.attachShadow({ mode: 'open' });

    // 定义一个模板
    const template = document.createElement('template');
    template.innerHTML = `
      <style>
        div {
          font-size: 24px;
          display: flex;
          align-items: center;
        }
        button {
          margin: 0 5px;
          padding: 5px 10px;
          font-size: 18px;
        }
      </style>
      <div>
        <button id="decrement">-</button>
        <span id="count">0</span>
        <button id="increment">+</button>
      </div>
    `;
    
    // 克隆模板内容
    const templateContent = template.content.cloneNode(true);
    
    // 将克隆的内容添加到影子 DOM
    shadow.appendChild(templateContent);

    // 绑定按钮点击事件
    shadow.getElementById('increment').addEventListener('click', () => {
      this.count++;
      this.update();
    });

    shadow.getElementById('decrement').addEventListener('click', () => {
      if (this.count > 0) this.count--;
      this.update();
    });
  }

  // 更新显示的计数值
  update() {
    this.shadowRoot.getElementById('count').textContent = this.count;
  }
}

// 注册自定义元素
customElements.define('simple-counter', SimpleCounter);
```

> 为了封装良好，template 定义在了组件内部，其实这时候直接用 innerHTML 就可以了，上面的写法纯粹是为了展示 HTML Template 用法
>

这样在 HTML 中引入 js 文件后就可以直接使用注册的元素了

```html
<simple-counter></simple-counter>
```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732713821994-91e9dc10-ea4f-407c-9a8e-159ceb5cbd77.png)

## 使用 slot 支持外部内容插入
通过 Slot 可以让 Web Components 组件接受的外部内容的占位符，将外部内容插入到你定义的组件的特定位置，而不必改变组件的内部结构

稍微修改上面 demo 的 template

```javascript
const template = document.createElement('template');
template.innerHTML = `
  <style>
    div {
      font-size: 24px;
      display: flex;
      align-items: center;
    }
    button {
      margin: 0 5px;
      padding: 5px 10px;
      font-size: 18px;
    }
    .container {
      border: 1px solid #ccc;
      padding: 10px;
    }
  </style>
  <div class="container">
    <slot name="title"><!-- 默认内容，可以为空 --></slot>
    <div>
      <button id="decrement">-</button>
      <span id="count">0</span>
      <button id="increment">+</button>
    </div>
  </div>
`;
```

```html
<simple-counter>
  <span slot="title">My Custom Counter</span>
</simple-counter>
```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732714200474-a1acf376-c4be-4833-bc76-9ae99d0bd03f.png)

## 属性支持
Web Components 支持在自定义元素上定义和使用属性，以便通过 HTML 属性或 JavaScript 动态设置和获取组件的状态

+ 如果需要监听属性的变化，可以通过定义 `static get observedAttributes()` 方法来声明哪些属性是需要被观察的
+ 当被观察的属性发生变化时，`attributeChangedCallback` 方法被调用，这时可以执行相应逻辑

```javascript
class CustomElement extends HTMLElement {
  static get observedAttributes() {
    return ['label'];
  }

  constructor() {
    super();
    this.attachShadow({ mode: 'open' });

    // 初始结构
    this.shadowRoot.innerHTML = `
      <style>
        div {
          font-size: 18px;
          color: #333;
        }
      </style>
      <div id="labelContainer"></div>
    `;
  }

  // 当属性变化时被调用
  attributeChangedCallback(name, oldValue, newValue) {
    if (name === 'label') {
      this.updateLabel(newValue);
    }
  }

  // 更新标签内容
  updateLabel(text) {
    this.shadowRoot.getElementById('labelContainer').textContent = text;
  }

  // 可以提供 getter 和 setter 让属性的操作更方便
  get label() {
    return this.getAttribute('label');
  }

  set label(value) {
    this.setAttribute('label', value);
  }
}

// 注册自定义元素
customElements.define('custom-element', CustomElement);
```



```html
<custom-element id="myElement" label="Initial Label"></custom-element>

<button id="changeLabelButton">Change Label</button>

<script>
  // 获取按钮和自定义元素的引用
  const button = document.getElementById('changeLabelButton');
  const customElem = document.getElementById('myElement');

  // 为按钮添加点击事件监听器
  button.addEventListener('click', () => {
    // 通过设置属性更新组件内容
    customElem.setAttribute('label', 'Updated Label');

    // 或者使用 setter 方法更新
    // customElem.label = 'Updated Label';
  });
</script>
```

## 自定义事件
在 Web Components 中，自定义事件允许组件与外部世界进行交互，通过使用 JavaScript 的 `CustomEvent` 接口，可以在自定义元素中创建和派发事件，让其它部分的代码可以监听这些事件并做出响应

```javascript
class SimpleCounter extends HTMLElement {
  constructor() {
    super();
    this.count = 0;

    this.attachShadow({ mode: 'open' });

    this.shadowRoot.innerHTML = `
      <style>
        div {
          font-size: 24px;
          display: flex;
          align-items: center;
        }
        button {
          margin: 0 5px;
          padding: 5px 10px;
          font-size: 18px;
        }
      </style>
      <div>
        <button id="decrement">-</button>
        <span id="count">0</span>
        <button id="increment">+</button>
      </div>
    `;

    this.shadowRoot.getElementById('increment').addEventListener('click', () => {
      this.count++;
      this.update();
      this.dispatchCountChangedEvent();
    });

    this.shadowRoot.getElementById('decrement').addEventListener('click', () => {
      if (this.count > 0) {
        this.count--;
        this.update();
        this.dispatchCountChangedEvent();
      }
    });
  }

  update() {
    this.shadowRoot.getElementById('count').textContent = this.count;
  }

  dispatchCountChangedEvent() {
    const event = new CustomEvent('countChanged', {
      detail: { count: this.count },  // 传递当前计数值
      bubbles: true,                  // 允许事件冒泡
      composed: true                  // 允许事件通过影子 DOM 树边界传播
    });
    this.dispatchEvent(event);
  }
}

customElements.define('simple-counter', SimpleCounter);
```

```html
<simple-counter id="myCounter"></simple-counter>

<script>
  // 获取自定义元素引用
  const counter = document.getElementById('myCounter');

  // 监听自定义事件
  counter.addEventListener('countChanged', (event) => {
    console.log('Count changed to:', event.detail.count);
  });
</script>
```

## 生命周期
<font style="color:rgb(51, 51, 51);">Web Components 提供了一套生命周期回调方法，让开发者能够在组件的不同生命周期阶段执行特定的代码</font>

1. **connectedCallback**：<font style="color:rgb(51, 51, 51);">当元素被插入到 DOM 中时调用，适合执行设置默认的属性、启动数据获取、设置事件监听器等操作</font>
2. **disconnectedCallback**：<font style="color:rgb(51, 51, 51);">当元素从 DOM 中移除时调用，适合执行清理工作，例如移除事件监听器、停止定时器等</font>
3. **attributeChangedCallback**：<font style="color:rgb(51, 51, 51);">当元素的属性增加、被移除或更改时调用，要使用此回调必须定义 static get observedAttributes() 方法</font>
4. **adoptedCallback**：<font style="color:rgb(51, 51, 51);">当元素从一个文档被移动到另一个文档时调用，这个情况在一般的 Web 应用中较少发生，常见于与多个文档交互的复杂应用如 Shadow DOM 的迁移</font>

```javascript
class LifeCycleElement extends HTMLElement {
  static get observedAttributes() {
    return ['data-value'];
  }

  constructor() {
    super();
    console.log('Constructor: 元素实例化');
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <div>Initial Value</div>
    `;
  }

  connectedCallback() {
    console.log('connectedCallback: 元素插入到 DOM 中');
    this.shadowRoot.querySelector('div').textContent = 'Connected to the DOM';
    this.start();
  }

  disconnectedCallback() {
    console.log('disconnectedCallback: 元素从 DOM 中被移除');
    this.stop();
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log(`attributeChangedCallback: 属性 ${name} 发生变化，从 ${oldValue} 变为 ${newValue}`);
    if (name === 'data-value') {
      this.shadowRoot.querySelector('div').textContent = `Attribute data-value: ${newValue}`;
    }
  }

  adoptedCallback() {
    console.log('adoptedCallback: 元素被移动到新的文档');
  }

  start() {
    // 例如：启动一个定时器
    this.interval = setInterval(() => console.log('Active...'), 1000);
  }

  stop() {
    // 清理工作：停止定时器
    clearInterval(this.interval);
  }
}

// 注册自定义元素
customElements.define('lifecycle-element', LifeCycleElement);
```

## Web Components 组件库
[Lit](https://lit.dev/docs/) 是一个用于构建快速、轻量级 Web Components 的 JavaScript 库，它由 Google 的开发团队创建，旨在简化和加速开发符合 Web Components 标准的组件。Lit 本身并不是一个完整的框架，而是一个工具集，帮助开发者轻松构建、自定义和组合 Web Components

> Lit is a simple library for building fast, lightweight web components.
>
> At Lit's core is a boilerplate-killing component base class that provides reactive state, scoped styles, and a declarative template system that's tiny, fast and expressive.
>

```typescript
import { LitElement, html, css } from 'lit';
import { customElement, property } from 'lit/decorators.js';

// 定义和注册一个新的自定义元素
@customElement('my-element')
export class MyElement extends LitElement {
  // 定义 CSS 样式
  static styles = css`
    p {
      color: blue;
    }
  `;

  // 使用装饰器定义属性，并指定类型
  @property({ type: String })
  message: string = 'Hello, World!';

  // 渲染模板
  render() {
    return html`
      <p>${this.message}</p>
    `;
  }
}
```

## 为什么 Web Components 没有流行
![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732718690366-cd560b64-2968-46d5-ae79-4935c2ee1025.png)

可以看出 Web Components 的[浏览器兼容性](https://caniuse.com/?search=web%20components)其实还算可以，在研发框架方面，目前基本上所有主流框架 [Vue](https://vuejs.org/guide/extras/web-components.html)、React、Angular、[Svelte](https://svelte.dev/docs/svelte/custom-elements)、Solid 等均都支持 Web Components，[Web Component DevTools](https://matsuuu.github.io/web-component-devtools/) 也可以很好的对 Web Components 组件进行调试

Web Components 是标准 Web 规范浏览器原生支持，在性能上也有一定的优势，还支持原生的样式隔离，在一些领域有明显优势

1. 跨框架组件库：创建可在多种框架中使用的通用UI组件
2. 微前端架构：构建可独立部署、技术栈无关的应用模块
3. 嵌入式组件：开发可嵌入第三方网站的小部件或组件
4. 企业级设计系统：构建统一的、可在不同项目间共享的组件库
5. 长期维护的项目：利用标准化技术降低对特定框架的依赖

看起来好处很多，但遗憾的是在业界 Vue、React 等组件方案是主流，Web Components 方案并没有大行其道，主要有几个原因

+ 功能局限：Web Components 的 API 比起框架需要的工具链和理念相对简单，但缺乏内置的状态管理、路由等高级特性，在大型应用中处理状态和复杂逻辑时会变得更繁琐。React 和 Vue 提供了强大的状态管理和生态工具，使得复杂应用的开发变得更为简单高效
+ 生态系统较小：Web Components 标准的发展和完善相对缓慢，相比 Vue 和 React，缺乏丰富的工具、库和社区支持。而 React 和 Vue 背后有庞大的社区和丰富的第三方库，提供了大量的即插即用解决方案，而 Web Components 则较少受到这种级别的社区支持

  


<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);">  
</font>

  




