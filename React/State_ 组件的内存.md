## 数据驱动 UI
React 被描述为“数据驱动 UI”，这一理念的核心在于视图（UI）是由数据状态决定的。换句话说，组件的呈现（即其返回的 JSX）是其当前数据状态的直接结果。当数据改变时，React 会根据新的数据自动更新和重新渲染 UI，这使得状态管理和 UI 更新更加可预测和一致

在 React 从 0.14 开始支持函数组件，在 16.8 以前 React 的函数组件又被称为无状态组件，因为它们只能通过 props 接收数据，不维护内部状态或者生命周期方法

```javascript
const Greeting = (props) => {
  return <h1>Hello, {props.name}!</h1>;
};
```

上面是一个纯函数的例子，组件渲染结果只和传入的 props 数据有关，函数没有任何副作用，这时候可以用以下公式描述 React

f(props) = UI

这种模式有些类似于 [EJS](https://ejs.co/)、[handlebars](https://handlebarsjs.com/) 等前端模版引擎，只不过使用了 JSX 语法，提升了 UI 代码的可读性

## 无处不在的变化
> Components often need to change what’s on the screen as a result of an interaction. Typing into the form should update the input field, clicking “next” on an image carousel should change which image is displayed, clicking “buy” should put a product in the shopping cart. Components need to “remember” things: the current input value, the current image, the shopping cart. In React, this kind of component-specific memory is called state.
>

而大部分 Web UI 除了静态展示，还需要响应用户的交互对 UI 做出变化，在表单输入 input 内容应该更新、点击轮播图片的 “下一个” 图标，显示的图片应该切换、点击“下单”商品应该被加入到购物车等。业界有各种流派解决这个问题

+ jQery 年代的绑定事件，手工操作 DOM 结构及数据变化，无状态管理，在复杂场景分散的 DOM 操作和事件绑定代码块使得逻辑难以维护
+ 组件内部维护状态数据，单向驱动 UI 发生变化，UI 的变化只能是状态数据变化引起组件重绘，这也是 React 采用的方式
+ 双向绑定相当于前两个的组合集成在了框架内部，早期的 MVVM 框架 Knockout、backbone 到现在的 Vue、Svelte 支持双向绑定
+ 使用集中式状态管理，数据变化反馈到外层，外层使用新参数调用组件重新渲染，这个基本就是没有性能优化的 Redux

我们来重点看下 React 组件响应用户交互的处理方式——通过用户交互事件改变组件内部状态数据，然后触发组件重新渲染，页面展示新的 UI

```javascript
const Counter = ( props ) => {
  // 使用 props 初始化 state，仅在组件第一次渲染时候调用
  const [count, setCount] = useState(props.initialCount);

  // 在事件处理程序中修改 state，触发组件重新渲染，组件使用新的 state
  const increment = () => {
    setCount(count + 1);
  }

  return (
    <div>
      <p>Current Count: {count}</p>
      // button 点击触发 click 事件
      <button onClick={increment}>Increment</button>
    </div>
  );
}

```

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1721738760314-0420aa95-1514-4535-87ac-a7c377f9ae01.png)

React 有几个设定

+ props 对象作为组件参数，传入之后不允许修改
+ 组件内部维护状态数据，称之为 state，并且暴露 API setState/useState 修改
+ 定时器或者用户事件等触发组件状态数据发生变化
+ 组件状态发生变化后触发组件使用原有的 props 和新的 state 重新渲染，更新 UI

这样就实现了组件响应用户交互，达到动态效果，我们可以改写一下上面的公式

f(props, state) = UI

> <font style="color:rgb(51, 51, 51);">As a component’s memory, state is not like a regular variable that disappears after your function returns. State actually “lives” in React itself—as if on a shelf!—outside of your function. When React calls your component, it gives you a snapshot of the state for that particular render. Your component returns a snapshot of the UI with a fresh set of props and event handlers in its JSX, all calculated using the state values from that render!</font>
>

在组件的生命周期内 props 保持不变，state 并不会像普通变量那样在函数返回后销毁。 state 实际  lives in React，存在于组件之外。组件每次渲染，会返回基于当下渲染状态（state）计算出来的 UI 快照，因此我们常把 state 理解为组件的 memory，只要知道当时的 state，就可以还原对应的 UI

## useState
useState 是 React Hooks 中最基础和最常用的 Hook 之一，它允许在函数组件中添加状态管理。useState Hook 接受一个初始状态值，返回一个数组，该数组包含当前状态值和一个用于更新状态的函数。

```jsx
import React, { useState } from 'react';

function Counter() {
  // 声明一个新的状态变量，命名为 "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### 更新对象和数组
数组和对象的更新需要使用展开运算符

```jsx
const [user, setUser] = useState({ name: 'John', age: 30 });

const updateUser = () => {
  setUser(prevUser => ({ ...prevUser, age: prevUser.age + 1 }));
};

const [items, setItems] = useState([1, 2, 3]);

const addItem = () => {
  setItems(prevItems => [...prevItems, prevItems.length + 1]);
};
```

或者使用不可变对象库，比如 Immer

```jsx
import produce from "immer";
import React, { useState } from 'react';

function App() {
  const [user, setUser] = useState({
    name: 'Alice',
    address: {
      city: 'New York',
      zip: '10001'
    }
  });

  const updateCity = () => {
    setUser(produce(user, draft => {
      draft.address.city = 'Los Angeles';
    }));
  };

  return (
    <div>
      <p>Name: {user.name}</p>
      <p>City: {user.address.city}</p>
      <button onClick={updateCity}>Move to LA</button>
    </div>
  );
}

export default App;
```

### 根据上一次的状态更新
当新的状态依赖于前一个状态时，使用函数式更新可以避免潜在的并发问题

```jsx
import React, { useState } from 'react';

function Counter() {
  // 初始化 count 状态为 0
  const [count, setCount] = useState(0);

  // 使用函数式更新来增加 count
  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };

  // 使用函数式更新来减少 count
  const decrement = () => {
    setCount(prevCount => prevCount - 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}

export default Counter;
```

### 延迟初始化
当初始状态的计算开销较大时，直接传递初始值会导致每次组件渲染时都重复执行这个昂贵的计算。通过传递一个返回初始值的函数，React 只会在首次渲染时调用这个函数计算初始状态，达到按需初始化的效果，从而提高性能

```jsx
import React, { useState } from 'react';

// 模拟从 localStorage 读取相关数据的函数
const getInitialValueFromLocalStorage = () => {
  console.log('Reading initial state from localStorage...');
  const savedValue = localStorage.getItem('counter');
  return savedValue !== null ? Number(savedValue) : 0;
};

function App() {
  // 使用延迟初始化，确保只有在首次渲染时才执行
  const [counter, setCounter] = useState(() => getInitialValueFromLocalStorage());

  const increment = () => {
    setCounter(prevCounter => {
      const newCounter = prevCounter + 1;
      localStorage.setItem('counter', newCounter);
      return newCounter;
    });
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default App;
```

## 深究 state
state 有 几个特征

+ Setting state only changes it for the next render.
+ A state variable’s value never changes within a render.
+ React keeps the state values “fixed” within one render’s event handlers.
+ state is fully private to the component declaring it.

下面使用示例来理解一下

### 初始化只被调用一次
我们知道当 state 改变时候会让组件重新执行，但useState 初始化状态代码，无论是通过直接传递初始值还是通过延迟初始化函数计算初始值，初始状态的计算都只在组件首次渲染时执行一次。在组件的后续重新渲染过程中，React 不会再次调用传递给 useState 的初始值或初始化函数，而是基于之前的状态值进行更新，这是因为 React 内部已经保存了状态值

```jsx
import React, { useState } from 'react';

// 模拟一个复杂的计算函数
const computeInitialValue = () => {
  // 无论点击多少次，只会打印一次
  console.log('Computing initial value...');
  return 0;
};

function App() {
  // 使用延迟初始化
  const [counter, setCounter] = useState(() => computeInitialValue());

  const increment = () => {
    setCounter(counter + 1);
  };

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default App;
```

### state 修改是异步、批量的
在同一次用户交互中可以多次调用 state 修改函数，触发组件重新渲染更新 UI，React 并不会立即更新状态并重新渲染组件。而是把这些状态更新操作推入一个队列，并且在当前事件处理程序（或生命周期方法、自定义 Hook 等）执行完之后批量处理这些状态更新操作，这样可以提升组件渲染性能并确保状态更新的效率和一致性

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // 此时的 count 仍然是初始值 0，但每次点击只渲染一次，界面 count +3 
    console.log(count); 
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```

在React 18之前，内置批处理只能在事件处理器中使用，在 React 18 即使是使用 promise、setTimeout 等仍然是批量处理的，这也就意味着** state 在一个渲染周期内值是不会变化的**

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // 输出当前渲染周期的 count 值
    console.log('Initial count:', count);

    setTimeout(() => {
      setCount(count + 1);
      // 这里的 count 仍然是旧的值
      console.log('Inside first setTimeout, count:', count);
    }, 0);

    setTimeout(() => {
      setCount(count + 1);
      // 这里的 count 仍然是旧的值
      console.log('Inside second setTimeout, count:', count);
    }, 0);

    setTimeout(() => {
      setCount(count + 1);
      // 这里的 count 仍然是旧的值
      console.log('Inside third setTimeout, count:', count); 
    }, 0);

    // 作用域内的最终 count 值仍然不变
    console.log('End of handleClick, count:', count);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```

点击按钮时，控制台将会依次打印如下内容：

```plain
Initial count: 0
End of handleClick, count: 0
Inside first setTimeout, count: 0
Inside second setTimeout, count: 0
Inside third setTimeout, count: 0
```

即使是组件已经完成异步更新，但 setTimeout 打印出来的值仍然是在当次渲染过程时的 count，因为在其创建 timer 时候它们的词法环境已经“捕获”了 handleClick 函数执行时的 count 值

### 传入函数获取最新 state
虽然 state 在一个渲染周期内的值不会发生变化，但在需要基于当前状态值进行更新操作时，可以使用函数式更新模式。setState 接受一个函数时，它会确保这个函数在内部处理状态更新逻辑时获得的是最新的状态值

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // 点击一次界面还是只渲染一次，结果 +3
    // 但在每个 setCount 内部可以获取最新 state
    setCount(prevCount => {
      console.log(prevCount); // 第一次: 0
      return prevCount + 1;
    });
    setCount(prevCount => {
      console.log(prevCount); // 第二次: 1
      return prevCount + 1;
    });
    setCount(prevCount => {
      console.log(prevCount); // 第三次: 2
      return prevCount + 1;
    });
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```



这样的话岂不是和上面说的 state 在一个渲染周期内不会发生变化矛盾了？其实并不矛盾，在 setState 内部虽然可以获取到最新的 state，但只能 作用于下次 UI 更新。也就是开始提到的 `Setting state only changes it for the next render.`

本次 UI 渲染使用的是上次 setState 的批量处理结果，无论何种方式更新，UI 表现都是一致的。看个两种方式混合写的例子

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log('Initial count:', count);  // 上次触发时候的值

    setCount(count + 1);
    console.log('After first setCount:', count);  // 仍然是初始值

    setCount(prevCount => {
      console.log('Inside functional update, prevCount:', prevCount);
      return prevCount + 1;
    });

    setCount(prevCount => {
      console.log('Inside functional update, prevCount:', prevCount);
      return prevCount + 1;
    });

    console.log('After all setCount:', count);  // 仍然是上次触发时候的值
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

export default Counter;
```

当点击按钮时，handleClick 内的 console.log 会输出：

```plain
Initial count: 0
After first setCount: 0
Inside functional update, prevCount: 1
Inside functional update, prevCount: 2
After all setCount: 0
```

界面上展示的 Count 值将更新为 `3`

### 同一个 state 组件在不同实例中是隔离的
在 React 中每个组件实例都有它自己独立的状态。当你在多个地方使用同一个组件时，每一个组件实例的状态都是独立且互相隔离的。这一特性可以确保组件在不同的使用场景下不互相干扰，保证了组件的可复用性和独立性。

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  );
}

function App() {
  return (
    <div>
      <h1>Counters</h1>
      <Counter />
      <Counter />
      <Counter />
    </div>
  );
}

export default App;
```

当在界面上点击不同的 Increment 按钮时，会看到每个 Counter 组件实例的计数器独立工作。每个 Counter 组件实例都有自己的状态，当你点击一个 Counter 的按钮时，只会更新该实例的状态，不会影响其他实例。

  
React 在内部使用两个重要数据结构来管理状态和副作用：

1. Fiber 树：React 通过 Fiber 数据结构管理组件的调和（reconciliation）和渲染。每个 Fiber 节点对应一个组件实例，并且持有组件实例的状态和副作用信息。
2. Hooks 数据：对于函数组件，React 会为每个组件实例维护一组 Hook 数据（状态、作用、上下文等），确保每个实例的状态和副作用处理是独立的。

  


