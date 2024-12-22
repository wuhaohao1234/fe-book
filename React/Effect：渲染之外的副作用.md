React 借鉴了许多函数式编程的理念，使得它在构建复杂用户界面时能够保持代码的简洁、可维护和可测试性。函数式编程在很大程度上依赖于纯函数

:::info
纯函数（Pure Function）是指在计算机科学和函数式编程中使用的一类函数。一个函数如果具备以下两个主要特性，我们就可以称之为纯函数：

1. 引用透明性：纯函数在相同的输入下总是产生相同的输出。也就是说，函数的输出仅依赖于其输入参数，不依赖于任何外部状态或变量。
2. 无副作用：纯函数不会改变函数外部的任何状态或变量，即在函数执行过程中，不会产生任何影响外部环境的副作用，如更改全局变量、输出日志、修改输入参数等。

纯函数的这两个特性为代码的测试、调试、并行化和重构提供了很大的方便。由于纯函数的输出完全由输入决定，预测函数的行为变得更加简单。另外，由于没有副作用，纯函数之间不会互相影响，这有助于提高代码的可维护性和可读性。

:::

在理想模式下 React 组件应该是纯函数，有些组件需要与外部系统同步。例如根据 React state 控制非 React 组件、设置服务器连接或在组件出现在屏幕上时发送分析日志，在 React 中被称为副作用，它们是“额外”发生的事情，与渲染过程无关

## 编写 Effect
编写 Effect 需要遵循以下三个步骤：

1. 声明 Effect，默认情况下 Effect 会在每次提交后都会执行
2. 指定 Effect 依赖，大多数 Effect 应该按需执行，而不是在每次渲染后都执行
3. 必要时添加清理函数，组件卸载或者依赖项改变时候，在执行新的 Effect 之前调用

```javascript
import React, { useEffect } from 'react';

const MyComponent = () => {
  useEffect(() => {
    // React 渲染、DOM 更新之后执行此处的代码
    console.log('Component mounted or updated');

    // 清除副作用，组件卸载时候执行
    return () => {
      console.log('Component will unmount or cleanup before next effect');
    };
  }, []); // 仅首次渲染执行
  
  return <div>Hello, World!</div>;
};
```

## 合理的使用依赖
减少或者合理使用 useEffect 的依赖项可以减少组件重新渲染次数，显著优化性能

### 使用 useMemo 或 useCallback
```jsx
import React, { useEffect, useMemo, useState } from 'react';

const MyComponent = ({ items }) => {
  const [filteredItems, setFilteredItems] = useState([]);

  // 假设这是一种昂贵的计算
  const filteredItemsMemo = useMemo(() => {
    return items.filter(item => item.value > 10);
  }, [items]);

  useEffect(() => {
    // 使用缓存的计算结果
    setFilteredItems(filteredItemsMemo);
  }, [filteredItemsMemo]);

  return (
    <div>
      {filteredItems.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};

export default MyComponent;
```

### 使用函数更新状态
```jsx
const [count, setCount] = useState(0);

useEffect(() => {
 // 只依赖某些真正需要的变量
 const id = setInterval(() => {
   setCount(prevCount => prevCount + 1);  // 使用函数式更新避免依赖count
 }, 1000);

 return () => clearInterval(id);
}, []);
```

### 使用稳定的 dispatch 函数
useReducer 创建的 dispatch 函数是一个闭包,它持有对内部状态和 reducer 函数的引用，这些引用在创建 dispatch 时就固定了。因为这些引用不会变，所以 dispatch 函数也不会变

也就是说 dispatch 函数在组件的整个生命周期内是同一个引用，即使组件重新渲染 dispatch 引用也保持不变，利用这一点可以把原本依赖 state 的 useEffect 改成依赖稳定的 dispatch 函数

```jsx
const MyComponent = () => {
  const [state, dispatch] = useReducer(...);

  useEffect(() => {
    const fetchData = async () => {
      dispatch({ type: 'FETCH_INIT' });

      try {
        const response = await fetch('https://api.example.com/data');
        const result = await response.json();
        dispatch({ type: 'FETCH_SUCCESS', payload: result });
      } catch (error) {
        dispatch({ type: 'FETCH_FAILURE', payload: error });
      }
    };

    fetchData();
  }, [dispatch]);

  const { data, loading, error } = state;

  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error.message}</div>;
  }

  return (
    <div>
      {data && data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};
```

## 清理函数的作用
useEffect 的清理函数有两个主要的执行时机

1. 组件卸载时：当组件即将从 DOM 中被移除时，清理函数会被调用。这个过程有助于避免内存泄漏和其他资源泄露。例如如果你在 useEffect 中设置了订阅、计时器或者事件监听器，在组件卸载时需要清理这些资源
2. 依赖项变化时：如果 useEffect 的依赖项数组中有一个或多个依赖项发生变化，旧的副作用会先被清理，然后再运行新的副作用函数

清理函数通过管理资源、避免重复操作、取消异步操作等方式，确保 React 组件在其生命周期内的行为是可控和高效的

### 防止内存泄漏
当组件卸载时，清理函数会释放任何仍在占用的资源或内存，确保不会发生内存泄漏。例如清除定时器、取消订阅网络请求或 WebSocket 连接等

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    console.log('Interval running');
  }, 1000);
  
  return () => {
    clearInterval(interval); // 清理定时器
  };
}, []);
```

### 避免重复操作
在组件重新渲染时，清理函数可以中断或取消之前的副作用，避免重复执行同样的副作用操作

```jsx
useEffect(() => {
  const handleResize = () => {
    console.log('Window resized');
  };

  window.addEventListener('resize', handleResize);
  
  return () => {
    window.removeEventListener('resize', handleResize); // 移除事件监听器
  };
}, []);
```

### 协调异步操作
清理函数可以在组件卸载时取消未完成的异步操作，比如网络请求，以防止在组件不再需要时更新它的状态

```jsx
useEffect(() => {
  const fetchData = async () => {
    const controller = new AbortController();
    const signal = controller.signal;
    
    try {
      const response = await fetch('https://api.example.com/data', { signal });
      const data = await response.json();
      console.log(data);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Fetch error:', error);
      }
    }
    
    return () => {
      // 取消未完成的请求
      controller.abort();
    };
  };
  
  fetchData();
}, []);
```

## useLayoutEffect
React 组件更新到页面有几个过程

1. React 组件渲染阶段，生成新的虚拟 DOM，计算出浏览器需要做的更新
2. React 向浏览器提交 DOM 变更阶段
3. 浏览器 DOM 树构建、更新
4. 样式计算和布局，浏览器绘制页面

useEffect 在浏览器完成渲染之后异步地调用执行，而很多时候需要对浏览器渲染结果立刻做出改变，这样会造成页面两次渲染。`useLayoutEffect` 可以保证在所有浏览器 DOM 变更之后、浏览器重新绘制之前执行（也就是上面流程这种 3 和 4 之间）

这意味着 useLayoutEffect 内的代码会在浏览器刷新屏幕之前执行，可以同步读取布局信息，并可通过 DOM API 修改布局来避免用户看到中间状态的闪烁或不一致

```jsx
import React, { useLayoutEffect, useRef, useState } from 'react';

const Example = () => {
  const [height, setHeight] = useState(0);
  const divRef = useRef(null);

  useLayoutEffect(() => {
    // 浏览器完成DOM更新后立即执行，但在绘制到屏幕前
    if (divRef.current) {
      setHeight(divRef.current.getBoundingClientRect().height);
    }
  });

  return (
    <div>
      <div ref={divRef} style={{ height: '100px' }}>
        This is a div.
      </div>
      <p>Div height: {height}px</p>
    </div>
  );
};

export default Example;
```

##  useEvent
有这样的一个 case

```jsx
function Chat() {
  const [text, setText] = useState('');

  const clickHandler = () => {
    sendMessage(text);
  };

  return <SendButton onClick={clickHandler} />;
}
```

一个很简单的事件处理程序，点击 button 展示一下当前的 state，但 clickHandler 有一个问题只是透传 state，每次组件渲染都需要创新创建，这不合理，如果使用 useCallback 优化发现其实是个死循环

```jsx
function Chat() {
  const [text, setText] = useState('');

  const clickHandler = useCallback(() => {
    sendMessage(text);
  }, [text]);

  return <SendButton onClick={clickHandler} />;
}
```

这时候其实可以使用 useRef 和 useLayoutEffect 解决这个问题

```jsx
function useEvent(handler) {
  const handlerRef = useRef(null);

  useLayoutEffect(() => {
    handlerRef.current = handler;
  });

  return useCallback((...args) => {
    const fn = handlerRef.current;
    return fn(...args);
  }, []); // 依赖列表为空 []，返回的回调函数在组件的整个生命周期内保持不变
}
```

无论 state 是否发生变化，返回值总是稳定的，但每次组件重新渲染，会调用最新的 handlerRef.current，而 handlerRef.current 已经在每次渲染时更新为最新的 handler，handler 已经从闭包中获取到了最新的 state



```jsx
function Chat() {
  const [text, setText] = useState('');

  const clickHandler = useEvent(() => {
    sendMessage(text);
  });

  return <SendButton onClick={clickHandler} />;
}
```

React 官方已经对 useEvent 有了 RFC [https://github.com/reactjs/rfcs/pull/220](https://github.com/reactjs/rfcs/pull/220)

## 你也许不需要 Effect
Effect 通常用于暂时跳出 React 代码并与外部系统进行同步，通过 useEffect 可以将包含副作用的逻辑（如数据获取、订阅等）从纯渲染逻辑中分离出来，使组件更加简洁和易维护，但如果组件中没有副作用，大可不必使用 Effect，React 官网教程也总结了[无需使用 React 的几种 case](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)，相信读了之后可以更好的使用 useEffect，列举几个印象深刻的

### 不要把可计算出来的逻辑写到 useEffect
当渲染使用的数据可以通过 props 和 state 计算得出时候，这个计算逻辑可以写在组件外层，不需要通过 useEffect 修改数据实现，典型例子就是 List 筛选

List 组件接收一个 items 列表作为 prop，然后用 state 变量 selection 来保持已选中的项。当 items 接收到一个不同的数组时，需要把 selection 重置为 null

```jsx
function List({ items }) {
  const [selection, setSelection] = useState(null);

  // 当 items 变化时，在 Effect 中调整 state
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

每当 items 变化时，List 及其子组件会先使用旧的 selection 值渲染，然后 React 会更新 DOM 并执行 Effect，最后调用 setSelection(null) 将导致 List 及其子组件重新渲染，重新启动整个流程。这样的写法本质上是把可计算的结果冗余的设置了 state，其实 selection item 完全可以直接依赖计算结果

```jsx
function List({ items }) {
  
  const [selectedId, setSelectedId] = useState(null);
  // 在渲染期间计算所需内容
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

### 区分事件与 Effect
两者在很多时候会互相配合的对 state 进行修改，但其触发逻辑是不同的

+ 事件只在处理用户交互操作带来的变化时候运行
+ 每当依赖数据发生变化，useEffect 就会运行

当不确定某些代码是应该在 Effect 中还是在事件处理程序中时，可以反问自己为什么这段代码需要运行，Effect 只用来执行显示给用户时组件有必要执行的代码



比如有个 Form 组件需要发送两个请求：

1. 在页面加载之际会发送一个分析请求
2. 当用户填写表格并点击提交按钮时，它会向 /api/register 接口发送一个请求

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  // 点击按钮的事件处理程序
  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

第一个 useEffect 是合理的，因为表单显示给用户的时候就需要发型分析请求，但第二个明显不合理，这个请求不是渲染表单引起的，而是用户交互操作引起的，应该使用事件处理程序

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

