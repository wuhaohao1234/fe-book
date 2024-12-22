`queueMicrotask` 是一个现代 JavaScript API，用于在当前（或者接下来的）任务完成后但在浏览器重新渲染之前安排一个微任务，这使得开发者可以在事件循环的微任务队列中安排一个任务，并确保它在事件循环的当前周期内尽快执行

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732542918001-306804df-334b-4a33-a23d-b50d6931473c.png)

## 使用 queueMicrotask
queueMicrotask 的用法相当简单，它接受一个回调函数，将该函数添加到事件循环的微任务队列中

```javascript
queueMicrotask(() => {
  console.log('microtask 1');
});

queueMicrotask(() => {
  console.log('microtask 2');
});

console.log('This is a synchronous log');
```

输出

```plain
This is a synchronous log
microtask 1
microtask 2
```

即使该微任务是在同步代码之后安排的，它也会在同步代码执行完成后立即执行——在任何异步的宏任务之前执行

## 为什么使用 queueMicrotask？
+ **即时性**: 微任务由于其短暂生命周期和紧接在当前任务后执行的特性，使得开发者可以在接下来的逻辑中立即对数据进行处理或状态进行更新
+ **高效性**: 与 `setTimeout(..., 0)` 这样的宏任务相比，微任务更高效，因为它们不会触发额外的事件循环周期
+ **控制异步逻辑**: 在处理复杂异步逻辑时，`queueMicrotask` 提供了一种细粒度的方式来进一步拆分和管理任务执行顺序。例如当需要在调整 DOM 后确保在布局完成前处理某些状态（如更新），微任务是一个很好的选择

## Tips
小心通过 queueMicrotask 安排过多的微任务，尤其是在循环或递归中，因为它们会阻塞后续的宏任务，可能导致整个应用逻辑上的卡顿或无限执行（有点像 Node.js 的 `nextTick`）

  


