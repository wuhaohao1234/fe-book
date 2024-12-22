BEM（Block, Element, Modifier）是一种 CSS 类名命名约定方法，确保每个类名的职责清晰，避免全局样式冲突

## 基础概念
在 BEM 中有三个基础概念

1. Block：页面或应用程序的独立实体，可以是按钮、菜单、表单等。块是最大的模块化单位
2. Element：块的一部分，没有独立的意义，依赖于块，比如按钮中的图标，表单中的输入字段
3. Modifier：修饰块或元素的外观、状态或行为，是块或元素的变体，例如不同的按钮大小、不同的状态（激活、禁用等）

```html
<!-- block -->
<div class="button">
  <!-- element -->
  <span class="button__text">Click me</span>
</div>
```

## 命名规则
BEM 的命名规则如下

+ 块：.block
+ 元素：.block__element
+ 修饰符：.block--modifier 或 .block__element--modifier

```html
<div class="button button--large button--primary">
  <span class="button__text">Click me</span>
</div>

<div class="button button--small button--secondary">
  <span class="button__text">Click me</span>
</div>
```

## demo
```html
<div class="login-form">
  <form class="login-form__form">
    <div class="login-form__group">
      <label class="login-form__label" for="username">Username</label>
      <input class="login-form__input" type="text" id="username">
    </div>
    <div class="login-form__group">
      <label class="login-form__label" for="password">Password</label>
      <input class="login-form__input" type="password" id="password">
    </div>
    <button class="button button--primary login-form__button">
      Login
    </button>
  </form>
</div>
```

