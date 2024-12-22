## 什么是 INP 指标
**INP（Interaction to Next Paint）** 指标用于衡量用户在与页面交互（如点击、键入、拖拽等）后，系统响应并在界面上进行下一次绘制所需的时间。具体来说 INP 衡量的是所有用户交互的响应时间，捕捉到的是用户感觉到的延迟。根据谷歌的标准 INP 的评分划分

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1733565389032-90f653ce-c3ef-4d29-90df-551ea13095b1.png)

### INP 的测量方式
INP 通过观察用户访问页面的整个生命周期中发生的所有单击、敲击和键盘交互的延迟来评估页面对用户交互的整体响应能力。最终的 INP 值是观察到的最长相互作用，忽略异常值

1. **捕捉交互事件**：INP 监测页面上的所有交互事件，如点击、触摸、键盘输入等
2. **记录时间**：记录用户触发交互事件到浏览器完成下一次绘制之间的时间间隔
3. **聚合结果**：INP 取所有交互的时间分布，关注于最长的响应时间

## INP 和 FID 区别
INP 考虑所有页面交互，而首次输入延迟（FID）仅考虑第一次交互。它还仅测量第一次交互的输入延迟，而不是运行事件处理程序所需的时间或呈现下一帧的延迟

鉴于 FID 也是一种负载响应指标，其背后的基本原理是，如果在加载阶段与页面进行的第一次交互几乎没有或没有可察觉的输入延迟，则该页面给人留下了良好的第一印象

INP 不仅仅是第一印象，通过对所有交互进行采样，可以全面评估响应能力，使 INP 成为比 FID 更可靠的总体响应指标

## 优化 INP 指标的方法
### 1. 减少主线程阻塞
+ 将耗时的 JavaScript 任务拆分为更小的任务，避免单个任务占用过长时间
+ 使用 `setTimeout`、`requestIdleCallback` 延迟处理部分非 UI 渲染任务，比如数据统计
+ 将计算密集型任务移至 Web Workers，避免阻塞主线程

```javascript
// 主线程
const worker = new Worker('worker.js');
worker.postMessage(data);
worker.onmessage = (e) => {
  console.log('Result from worker:', e.data);
};

// worker.js
self.onmessage = (e) => {
  const result = computeHeavyTask(e.data);
  self.postMessage(result);
};
```

### 2. 优化事件处理
对高频触发的事件（如 scroll、resize、mousemove）使用去抖或节流机制，减少事件处理的频率

```javascript
// 节流函数
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// 使用节流
window.addEventListener('scroll', throttle(handleScroll, 200));
```



使用事件委托，将多个子元素的事件监听器合并到父元素上，减少 DOM 节点的事件监听器数量

```javascript
// 使用事件委托
document.querySelector('#parent').addEventListener('click', (e) => {
  if (e.target.matches('.child')) {
    // 处理子元素的点击事件
  }
});
```

### 3. 延迟非关键 JavaScript 加载
使用 `defer` 和 `async`延迟加载非关键的 JavaScript 脚本，让浏览器在解析 HTML 后再执行 JavaScript，确保脚本不会阻塞页面渲染

```html
<script src="script.js" defer></script>
<script src="script.js" async></script>

```

使用代码拆分和懒加载技术，根据用户的操作按需加载 JavaScript 代码，减少初始加载时间

```javascript
import React, { Suspense } from 'react';

const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>

  );
}
```

### 4. 优化动画和过渡
尽量使用 CSS 动画和过渡，利用 GPU 加速，减少主线程负担，尽量避免在动画过程中进行布局计算（如读取布局属性），减少重排和重绘

```css
.animated {
  transform: translateZ(0); /* 启用硬件加速 */
  transition: transform 0.3s ease;
}
```

### 8. 提高渲染性能
对于长列表或大量 DOM 节点，使用虚拟化技术（如 React Window、React Virtualized）仅渲染可视区域的元素，减少 DOM 复杂度

```javascript
import { FixedSizeList as List } from 'react-window';

const Row = ({ index, style }) => (
  <div style={style}>Row {index}</div>

);

function App() {
  return (
    <List
      height={500}
      itemCount={1000}
      itemSize={35}
      width={300}
    >
      {Row}
    </List>

  );
}
```

使用 `React.memo`、`useMemo` 和 `useCallback` 等优化组件和函数，避免因父组件更新导致子组件不必要的重渲染

```javascript
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  // 渲染逻辑
});

function Parent({ data }) {
  const memoizedData = useMemo(() => computeData(data), [data]);
  return <ExpensiveComponent data={memoizedData} />;
}
```

## 小试牛刀
```javascript
// 原始代码
function handleClick() {
  // 密集计算
  const result = computeHeavyTask();
  updateUI(result);
}
```

示例中点击按钮后，INP 指标较高，用户体验不佳，主要原因是

+ 点击事件触发大量计算或 DOM 操作，导致主线程阻塞
+ 没有使用异步处理，渲染任务长时间占用主线程

使用以下几个步骤进行优化

1. **异步处理计算**：

```javascript
// 优化后使用 Web Worker
function handleClick() {
  const worker = new Worker('worker.js');
  worker.postMessage('start');
  worker.onmessage = (e) => {
    updateUI(e.data);
  };
}
```

2. **拆分任务**：

```javascript
function handleClick() {
  setTimeout(() => {
    const result = computeHeavyTask();
    updateUI(result);
  }, 0);
}
```

3. **优化 DOM 操作**：

```javascript
function updateUI(result) {
  // 批量更新 DOM，减少重排和重绘
  requestAnimationFrame(() => {
    // 更新操作
  });
}
```

通过这些优化措施，点击按钮后的处理过程更加高效，减少了主线程阻塞时间，INP 指标得到了提升

