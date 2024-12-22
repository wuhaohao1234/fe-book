## 状态管理的动机
React 使用数据驱动 UI 的方式，状态是指组件内部数据的变化，而状态管理也就是维护应用的数据变化，[Redux 文档](https://cn.redux.js.org/understanding/thinking-in-redux/motivation)中对状态管理 library 动机的介绍非常具象：

随着 JavaScript 单页应用开发日趋复杂，编码要管理的 state（状态）比以往任何时候都要多。 这些 state 可能包括服务器响应、缓存数据、本地生成尚未持久化到服务器的数据，也包括 UI 状态，如激活的路由，被选中的标签，是否显示加载动效或者分页器等等。

管理不断变化的 state 非常困难。如果一个 model 的变化会引起另一个 model 变化，那么当 view 变化时，就可能引起对应 model 以及另一个 model 的变化，这个变化反过来又可能引起另一个 view 的变化。当这些连锁反应到一定程度之后，开发者根本搞不清楚到底发生了什么：state 在什么时候、由于什么原因、如何变化已然不受控制。 当系统变得错综复杂的时候，想重现问题或者添加新功能就会变得举步维艰

状态对 React 应用是如此重要，为了确保复杂应用状态的可维护性、可预测性和可扩展性，各路状态管理 library 应运而生

## useContext 之殇
useContext 是 React 内置的状态管理解决方案，在一定程度上解决了组件状态共享的问题，看一个简单的 demo

```jsx
import React, { createContext, useContext, useState } from 'react';

// 创建一个 Context
const DemoContext = createContext();

// 创建一个 Provider，用于提供共享的状态
const DemoProvider = ({ children } => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('Anonymous');

  return (
    <DemoContext.Provider value={{ count, setCount, name, setName }}>
      {children}
    </DemoContext.Provider>
  );
});

const Child1 = () => {
  const { count, setCount, name } = useContext(DemoContext);

  return (
    <div>
      <h2>Child 1</h2>
      <p>Count: {count}</p>
      <p>Name: {name}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
    </div>
  );
}

const Child2 = () => {
  const { name, setName } = useContext(DemoContext);

  return (
    <div>
      <h2>Child 2</h2>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
    </div>
  );
};

const App = () => {
  return (
    <DemoProvider>
      <Child1 />
      <Child2 />
    </DemoProvider>
  );
};
```

在这个示例中创建了一个 DemoContext 用于存储和管理 `count` 和 `name` 两个状态，用于子组件共享并更新

+ DemoProvider 组件则用来提供这些状态，并将其传递给子组件
+ 在 Child1 中展示了如何使用 useContext 获取和修改 `count` 和 `name`
+ 在 Child2 中则展示了如何只修改 `name`

在使用 useContext 处理复杂应用时候会出现的问题 Recoil 做了恰如其分的总结

+ 组件间的状态共享只能通过将 state 提升至它们的公共祖先来实现，但这样做可能导致重新渲染一颗巨大的组件树（管理成本和子组件意外渲染问题）
+ Context 只能存储单一值，无法存储多个各自拥有消费者的值的集合。
+ 以上两种方式都很难将组件树的顶层与子组件进行代码分割

另外 useContext 也没有提供对异步请求的解决方案

## 状态管理 library 要解决的问题
状态管理 library 的目标是提供一种机制来管理和维护 React 应用中的状态，并且使得这些状态能够跨组件共享、状态的变化可以预测。本文主要关注以下几个问题来对比，以求窥得其设计精妙之处

+ 如何做到数据共享，兼顾子组件精准渲染
+ 如何获取和修改状态
+ 如何管理异步工作流

## 几种状态管理的实现思路
正如文章标题中提到的 Redux、Zustand、Mobx、Valtio、Recoil、jotai、XState 状态管理解决方案百花齐放，先看看社区怎么选

[https://npmtrends.com/jotai-vs-mobx-vs-recoil-vs-redux-vs-valtio-vs-xstate-vs-zustand](https://npmtrends.com/jotai-vs-mobx-vs-recoil-vs-redux-vs-valtio-vs-xstate-vs-zustand)

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722683161758-17d2ea17-0dda-4680-94c2-4fe43c4a4de6.png)

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722683223112-8bd81e91-e250-45f8-ba45-67a2f1e47645.png)

可以看到最年长的 Redux 仍然遥遥领先，而同流派的 Zustand 已经实现对 Mobx 的反超，在增长趋势惊人，在具体了解几个 library 之前可以按照其实现思路做一个分类

### store 模式
Store 模式通过集中式的存储来管理应用的状态，这种模式的基本思想是将应用的所有状态存储在一个单一的地方，通常称为“store”。组件通过与 store 的交互来读取和更新状态，而不是直接在组件内部管理状态。Store 模式有几个核心概念

+ Store：保存应用所有 state 的对象。它是只读的，唯一改变 state 的方法是触发 actions
+ Actions：描述应用行为的普通对象。它们是 store 进行状态更新的唯一途径，Actions 通常包含一个 type 字段来标识动作的类型，以及其他必要的数据
+ Dispatch：触发 action 的过程。组件通过 dispatch 来发送 actions 给 store，以触发 state 更新
+ Reducers：接受当前的 state 和一个 action，然后返回一个新的 state，Reducers 描述了 action 如何改变 state
+ Selectors：用于从 store 中选择片段数据，使组件获得最少依赖的状态数据

**Redux**、**Zustand** 正是使用了 store 模式设计，Redux 官网的 gif 很好的演示了 redux 中数据流动的方向及 Store、 Dispach、Action、Reducer 的配合过程

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1722672900447-d10ab0f9-e05d-407d-94d9-da791cae9180.gif)

Store 模式中的核心概念也深刻影响了其它状态管理 library，后续出现了相似的术语，代表的含义基本一致

### 响应模式
响应模式是一种通过观察（observe）状态变化并自动更新界面的方法。这种模式的核心思想是，当数据发生变化时，界面能够自动地响应并更新显示，可以使得状态管理更加直观、高效和模块化。响应模式一般有以下特征

+ 统一管理状态：在响应式模式中通常会采用一个统一的存储或者状态容器用于存储整个应用的状态，但这不代表就是 Store 模式
+ 状态观察者和订阅者：状态容器会维护一组观察者监听状态数据的变化，当状态发生改变时所有注册的观察者都会被通知
+ 自动更新：观察者通常是一些自动更新的组件或者函数。当它们收到状态变化的通知时，会自动重新计算和渲染与之关联的界面

响应模式一般有几个核心概念

+ State：所有的数据状态都应该集中在一个地方进行管理，确保数据的一致性和可预测性。一般使用 Proxy 使其 observable
+ Actions：触发状态改变的事件或操作。它们是描述“发生了什么”的简单对象，不包含具体的状态改变逻辑
+ Derivations：派生状态是基于主状态的计算值。这类似于计算属性，当相关的主状态改变时，派生状态会自动更新，经常用于减少重复计算和优化性能

**MobX**、**Valtio** 都使用了响应模式，MobX 官网流程图很好的解释了其 data flow

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722675313682-d09befef-1b2c-4ded-bf0a-4c2fd0597dad.png)

### 原子模式
不同于 Store 模式和响应模式把 State 集中起来管理，状态维护在组件顶部，子组件需要的话通过 selector 按需获取，数据自上向下流动。原子模式提倡把应用的全局状态拆分为多个小的、独立的状态单元——这些状态单元被称为原子（Atom），提供更细粒度的状态管理，以便组件可以更高效地更新和渲染。原子状态模式有几个核心理念

+ 细粒度的状态管理：将应用的状态拆分为多个独立的 Atom，每个 Atom 管理一块独立的、可重用的状态
+ 共享状态与组合：其它组件可以订阅这些 Atom，当 Atom 的状态改变时，订阅这些 Atom 的组件会自动重新渲染
+ 无缝更新：因为每个 Atom 独立管理其状态，更新一个 Atom 对其它 Atom 以及订阅它的组件不会有直接的影响。这减少了不必要的重渲染和性能开销
+ 不可变性和状态演化：像 React 一样，原子状态模式通常也强调状态的不可变性，这意味着每次状态更新都会生成一个新的状态，而不是直接修改原来的状态

在原子模式中有两个核心概念

+ Atom：用于定义状态的最小单位，类似于独立的状态片段，当 atom 被更新，每个被订阅的组件都将使用新值进行重渲染
+ Selectors：用于取派生状态，基于一个或多个 atom 计算出新的状态，从组件的角度来看，selector 和 atom 具有相同的功能，因此可以交替使用

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722680395449-5410b576-1901-4d0e-963b-b7a861f4b7f0.png)

可以说原子模式是对 useState + useContext 的升级，原子模式的代表作是 **Recoil** 和 **Jotai**

### 状态机模式
既然 React 组件的 UI 由状态驱动，而数学领域又有专门处理状态的状态机模型，不如做个结合，XState 应运而生。目前使用状态机模式的 library 只有 XState，首先了解一下其核心概念

#### 状态机模型
状态机是一种数学模型，它描述了在任何给定时间只能处于一种状态的系统，系统在不同的状态之间可以通过确定的事件进行转化（是不是和 React 理念特别契合），由以下几个部分组成：

+ 状态（States）：系统可能的不同状态
+ 事件（Events）：引起状态变化的事件
+ 转换（Transitions）：由特定事件引起的状态改变
+ 初始状态（Initial State）：系统开始时的状态

为了更好的理解状态机和状态图，可能需要读一下[状态机和状态图简介 | XState 文档](https://lecepin.github.io/xstate-docs-cn/zh/guides/introduction-to-state-machines-and-statecharts/#%E7%8A%B6%E6%80%81-states)

#### Actor model
XState 支持 Actor model，允许运行多个独立的状态机（每个称为 actors），这些 actor 可以独立处理事件，并且可以相互通信从而实现复杂的并发操作。可以看出来 XState 同样没有使用统一管理的 Store

#### 核心组件
XState 种有几个核心组件

1. createMachine：定义状态机和状态图的函数
2. interpret：创建状态机解释器，它是状态机的运行实例
3. useMachine：React Hook，用于在 React 组件中使用状态机

```jsx
import { createMachine } from 'xstate';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: {
      on: { TOGGLE: 'active' }
    },
    active: {
      on: { TOGGLE: 'inactive' }
    }
  }
});

// 注册一个状态转换监听器，每当状态转换时，这个监听器会被调用
const lightService = interpret(lightMachine)
  .onTransition((state) => console.log(state.value));

export default toggleMachine;
```

这就是一个开关的状态机，在组件中使用非常简单

```jsx
import React from 'react';
import { useMachine } from '@xstate/react';
import toggleMachine from './toggleMachine';

const ToggleButton = () => {
  const [state, send] = useMachine(toggleMachine);

  return (
    <button onClick={() => send('TOGGLE')}>
      {state.matches('inactive') ? 'Off' : 'On'}
    </button>
  );
};

export default ToggleButton;
```

  
了解了这些基础概念之后，可以通过大家很熟悉的 Todo 来更好的理解各个状态管理 library 来如何管理状态，为了展示其管理异步任务能力，在本地通过 Promise 模拟网络请求

:::info
+ [Store 模式 React 状态管理 library —— Redux 与 Zustand](https://juejin.cn/post/7399276698619052041)
+ 响应模式 React 状态管理 library —— MobX 与 Valtio
+ 原子模式 React 状态管理 library —— Recoil 与 Jotai
+ 状态机模式 React 状态管理 library —— XState

:::

