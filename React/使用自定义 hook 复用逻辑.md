在没有 Hooks 之前复用逻辑需要借助于高阶组件（HOC）或者渲染属性（Render Props） 的模式来实现，同时函数组件没有状态管理的能力，导致只能用类组件，这样的模式会引起大量的嵌套，使代码难以维护。通过使用 Hooks 开发者可以将逻辑分离到独立的单元中，这样既提高了代码的可读性，也方便测试和复用。除了 React 官方提供的 useState、useEffect 等内置 hook，React 允许开发者自定义 Hook

## 自定义 hook 约定
+ 函数使用 use 开头，然后紧跟一个大写字母
+ 在 hook 内部可以使用其它 hook，但必须在函数顶层使用

每当组件重新渲染，自定义 Hook 中的代码就会重新运行，因为自定义 hook 中使用使用到的 React 原生 hook 也必须定义在顶层，这就意味着在整个组件中所有 hook（包括自定义 Hook 内部的 Hook）仍然会按照调用顺序执行

## useData(url)
我们可以自定义一个发请求的 hook，内部保证组件多次重复渲染不会导致请求结果设置混乱

```jsx
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

使用这样的自定义 hook 可以让 UI 渲染逻辑清晰很多

```jsx
function App() {
  const url = 'https://jsonplaceholder.typicode.com/posts';
  const data = useData(url);

  return (
    <div className="App">
      <h1>帖子列表</h1>
      {/* 数据为null时显示加载中 */}
      {data === null ? (
        <p>加载中...</p>
      ) : (
        <ul>
          {/* 遍历数据数组并展示每个帖子 */}
          {data.map((post) => (
            <li key={post.id}>
              <h2>{post.title}</h2>
              <p>{post.body}</p>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

组件不用自己去管理请求过程的细节，只需要根据是否有数据渲染不同的 UI 样式即可

## usePrevious
在有些场景我们需要知道上一次的 state 值，可以自定义一个 usePrevious

```jsx
function usePrevious(value) {
  const ref = useRef(value);
  useEffect(() => {
    // 组件完成渲染后更新为当前的值
    ref.current = value;
  }, [value]);

  // 直接返回 ref 中保存的前一个状态值
  return ref.current;
}
```

## useCountDown
实现一个简单的 count down hook，可以自定义时间间隔以及每次 tick 的回调函数

```jsx
import { useEffect, useState } from 'react';

function useCountdown(initial, interval, onTick) {
  const [count, setCount] = useState(initial);

  useEffect(() => {
    // 重置计数器
    setCount(initial);
  }, [initial]);

  useEffect(() => {
    if (count <= 0) return;

    const id = setInterval(() => {
      setCount((prevCount) => {
        const newCount = prevCount - 1;
        onTick(newCount);
        return newCount;
      });
    }, interval);

    return () => clearInterval(id);
  }, [interval, onTick]);

  return count;
}

export default useCountdown;
```

这样使用起来就非常简单了

```jsx
function Demo() {
  const count = useCountdown(100, 500, (num) => {
    console.log('Tick:', num);
  });

  return (
    <div>
      <p>倒计时: {count}</p>
    </div>
  );
}
```

