> UnoCSS 作者的[重新构想原子化 CSS](https://antfu.me/posts/reimagine-atomic-css-zh)，在[这里](https://unocss.dev/play)体验 UnoCSS，对类名不熟悉可以在[文档](https://unocss.dev/interactive/)中查询
>

## 原子化
TailwindCSS、WindiCSS 是原子化 CSS 框架，UnoCSS 是生成这类原子化 CSS 框架的工具。因此 UnoCSS 生成的每个类名对应一个单一的、简单的规则，<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);">如 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);">text-red-500</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);"> 可能只负责设置文本颜色</font>

## 按需生成
不同于 TailwindCSS 预定义 Utilities，UnoCSS 用了生成式的思路，解析开发者的 HTML、JSX 文档，识别到 CSS classname 后，根据规则生成对应的  CSS 属性

```javascript
import { defineConfig } from 'unocss'

export default defineConfig({
  rules: [
    ['m-1', { margin: '1px' }],
  ],
})
```

这样的静态规则可以生成 `.m-1 { margin: 1px;}`

```javascript
import { defineConfig } from 'unocss'

export default defineConfig({
  rules: [
    [/^m-([\.\d]+)$/, ([_, num]) => ({ margin: `${num}px` })],
  ],
})
```

这样 `m-任意数字`都可以按照规则来生成样式了

```jsx
<div class="m-1">Hello</div>
  <div class="m-7.5">World</div>
```

生成样式

```css
.m-1 { margin: 1px; }
.m-7.5 { margin: 7.5px; }
```



UnoCSS 也提供了 presets 规则集 [https://unocss.dev/guide/#presets](https://unocss.dev/guide/#presets)

```css
import { Preset } from 'unocss'

export const myPreset: Preset = {
  name: 'my-preset',
  rules: [
    [/^m-([.\d]+)$/, ([_, num]) => ({ margin: `${num}px` })],
    [/^p-([.\d]+)$/, ([_, num]) => ({ padding: `${num}px` })],
  ],
  variants: [/* ... */],
  shortcuts: [/* ... */],
  // ...
}

```



:::info
当然 TailwindCSS 的 JIT 编译模式通过类似 CSS Tree Shaking 的方式也可以做到 CSS 文件最终只包含用到的样式

:::

## 即时
使用 UnoCSS ，开发过程中添加或修改 HTML/CSS 在保存文件时，UnoCSS 会即时重新计算和生成必要的 CSS，这样可以立即反映在浏览器中

## 核心特性
1. 高度可定制性：UnoCSS 提供了灵活的配置选项，允许自定义各种规则、主题、以及全新的工具类生成逻辑。开发者可以自己定制甚至创建新规则插件，极大地扩展工具的能力
2. 零运行时开销：因为 CSS 是在构建时生成的，UnoCSS 不会在客户端执行复杂的运行时逻辑，从而不增加运行时的负担
3. 框架无关性：UnoCSS 可以集成到各种构建工具和前端框架中，比如 Vite、Webpack、Nuxt 等，适合于多种技术栈开发的需求
4. 插件与扩展：通过插件机制扩展 UnoCSS 的功能，可以轻松地添加自定义规则和功能扩展
5. 高性能：UnoCSS 的设计注重编译速度和生成后的 CSS 体积，适合于需要高性能和小型构建输出的项目。



