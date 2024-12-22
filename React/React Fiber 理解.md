React Fiber 是 React 16 中引入的一种全新的协调（Reconciliation）引擎，用于优化和改进 React 的渲染过程，使其能够更高效地处理复杂的用户界面的渲染和响应用户交互

## 为什么需要 Fiber
在 Fiber 之前 React 组件的渲染过程是同步和不可中断的，这意味着一旦开始渲染组件树，就会处理完所有任务直到完成，这就引发了一些用户体验上的问题

+ **阻塞主线程**：<font style="color:rgb(51, 51, 51);">大型组件树或复杂的用户界面更新会占用大量时间，导致浏览器无法及时响应用户输入，造成界面卡顿</font>
+ **缺乏任务优先级**：<font style="color:rgb(51, 51, 51);">所有更新被视为同等重要，高优先级的用户交互和低优先级的后台数据更新无法区别对待</font>

```jsx
class App extends React.Component {
  constructor() {
    super();
    this.state = {
      items: []
    };
  }

  // 模拟一个非常大的数据集
  componentDidMount() {
    const items = [];
    for (let i = 0; i < 10000; i++) {
      items.push(`Item ${i}`);
    }
    this.setState({ items });
  }

  render() {
    return (
      <div>
        <h1>Large List</h1>
        <ul>
          {this.state.items.map((item, index) => (
            <li key={index}>{item}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```

在无 Fiber 的旧版本 React 中，这种渲染大列表的操作将完全占用主线程直到渲染完成，这可能导致浏览器在渲染过程中不响应用户的输入（无法点击按钮、滚动等）

## <font style="color:rgb(51, 51, 51);">React Fiber 解题思路</font>
React Fiber 通过以下几个核心思路，解决了渲染卡顿的问题：

1. **任务切片：**将渲染任务分解为更小的单元（Fiber），在多个时间片中依次执行。这种方式避免了一次性执行大量渲染任务，减少了对主线程的长时间占用
2. **优先级调度：**允许为不同类型的更新赋予不同的优先级。例如，用户输入事件会得到更高的优先级，以确保用户界面可以快速响应，而不重要的后台任务可以被调度到稍后执行
3. **渲染可中断与恢复：**通过将渲染工作拆分成更小的单元任务，使得 React 可以在必要时暂停工作。这使得浏览器在长时间任务执行期间可以响应用户的交互，从而提高了用户体验

## React Fiber 实现
### Fiber 节点
Fiber 引入了一种新的数据结构——Fiber 节点，每个 Fiber 节点代表组件树中的一个组件，同时 Fiber 节点之间采用链表结构，便于高效地遍历和调度任务

Fiber 节点包含组件的类型、props、状态等信息

```jsx
const fiberNode = {
  tag,           // 节点类型，如 FunctionComponent、HostComponent
  key,           // 用于识别节点的唯一标识
  stateNode,     // 与 Fiber 对应的宿主实例，如 DOM 元素
  child,         // 子 Fiber 节点
  sibling,       // 兄弟 Fiber 节点
  return,        // 父 Fiber 节点
  pendingProps,  // 下一次渲染的 props
  memoizedProps, // 当前渲染的 props
  lanes,         // 更新优先级
  alternate,     // 指向旧版本的 Fiber 节点
  // 其它属性...
};
```

每个 Fiber 节点代表一个单元，所有 Fiber 节点共同组成一个 Fiber 树，使用`child`、`sibling` 和 `return` 字段构成了 Fiber 节点之间的链接关系，使 React 能够遍历组件树并知道从哪里开始、继续或停止工作

```jsx
<A>
  <B1>
    <C1/>
    <C2/>
  </B1>
  <B2>
    <C3/>
    <C4/>
  </B2>
</A>
```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1733562505237-954a5b93-1528-4966-ad31-90020eb8c67f.png)

### Fiber 工作流
React Fiber 的工作流程分为两个主要阶段

1. **渲染阶段（Reconciliation Phase）：**在渲染阶段 React Fiber 通过遍历组件树，生成新的 Fiber 节点，并进行比较和标记需要更新的部分，这一阶段可以被中断、暂停和恢复，实现异步渲染
    1. 从根节点开始，逐一访问每个组件，遍历组件树
    2. 为每个组件创建对应的 Fiber 节点，记录其配置信息
    3. 将新旧 Fiber 树进行比较，标记需要进行的更新操作（如插入、删除、修改）
    4. 根据任务的优先级，将渲染任务分配到不同的时间片中执行
2. **提交阶段（Commit Phase）：**提交阶段是一个同步、不可中断的过程，负责将渲染阶段生成的变化应用到实际的 DOM 上，这个阶段确保了视图的一致性和稳定性
    1. 调用生命周期方法：在提交前后调用组件的生命周期方法
    2. DOM 操作：根据标记的更新操作执行实际的 DOM 更新，如新增节点、删除节点、更新属性等
    3. 布局与效果处理：执行浏览器的布局和绘制操作，以及其它需要同步完成的效果

## 双缓冲技术
React 在更新视图时候会维护两个树

1. **<font style="color:rgb(51, 51, 51);">Current Tree</font>**<font style="color:rgb(51, 51, 51);">：代表已提交到 DOM 上的组件树，所有节点都是已经完成渲染的组件</font>
2. **<font style="color:rgb(51, 51, 51);">Work-In-Progress Tree</font>**<font style="color:rgb(51, 51, 51);">：是当前 Fiber 树的一个副本，代表正在进行协调（Reconciliation）和渲染的组件树</font>

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1733563266573-22182166-59f1-4842-9459-121fcfd265e1.png)

当需要更新组件时，React 会创建一个 WIP Tree 的副本，以承载新的更新

1. 渲染阶段：React 在 WIP Tree 上执行协调和渲染过程，逐步构建新的组件树，由于 WIP Tree 是独立的，可以随时中断和恢复渲染过程，处理高优先级任务
2. 提交阶段：一旦协调和渲染完成 React 将 WIP Tree 的更改应用到真实 DOM 上，WIP Tree 成为新的当前 Fiber 树，准备接收下一次更新

这种流程确保了渲染过程的高效性和稳定性，同时允许 React Fiber 实现任务中断与恢复、优先级调度等高级功能

## 调度器 (Scheduler)
调度器是 React Fiber 架构中负责管理渲染任务的执行顺序和优先级的模块，它的主要目标是确保高优先级的任务能够及时响应用户操作，而低优先级的任务则在系统空闲时执行，从而提升整体应用的性能和用户体验。调度器有几个核心功能

+ **任务优先级管理**：<font style="color:rgb(51, 51, 51);">根据任务的重要性和紧急程度，为任务分配不同的优先级，确保关键任务优先执行</font>
+ **任务队列与排序**：<font style="color:rgb(51, 51, 51);">维护多个优先级的任务队列，并按照优先级对任务进行排序，优化执行顺序</font>
+ **时间分片**：<font style="color:rgb(51, 51, 51);">将渲染任务分解为更小的单元，在浏览器空闲时间逐步执行</font>
+ **任务中断与恢复**：<font style="color:rgb(51, 51, 51);">允许在渲染过程中中断当前任务，处理高优先级任务后再继续执行被中断的任务</font>

## 为什么 Vue 不需要 Fiber
<font style="color:rgb(51, 51, 51);">Vue 通过以下机制实现高效的响应式更新：</font>

1. **<font style="color:rgb(51, 51, 51);">依赖收集</font>**<font style="color:rgb(51, 51, 51);">：在数据属性被访问时，Vue 记录下哪些组件依赖于这些数据</font>
2. **<font style="color:rgb(51, 51, 51);">数据劫持</font>**<font style="color:rgb(51, 51, 51);">：使用 Object.defineProperty或 Proxy拦截数据属性的读取和修改操作</font>
3. **<font style="color:rgb(51, 51, 51);">响应式更新</font>**<font style="color:rgb(51, 51, 51);">：当数据发生变化时，Vue 会通知所有依赖于该数据的组件，触发重新渲染</font>

<font style="color:rgb(51, 51, 51);">这种精细的依赖追踪和更新机制，能够精准地识别哪些数据变化影响到哪些组件，确保仅对必要的部分进行重新渲染，因此在处理复杂的应用状态和高频率的更新时，不需要 Fiber 依然能够保持高性能</font>

<font style="color:rgb(51, 51, 51);background-color:rgb(249, 250, 255);">  
</font>

