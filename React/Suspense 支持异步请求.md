<Suspense> 允许在子组件完成加载前展示备选方案（fallback），子组件有两种准备状态

+ **组件代码没有 ready：**[Suspense 支持组件懒加载](https://juejin.cn/post/7400566246435897394)中演示了 Suspense 和 React.lazy 配合支持组件按需加载
+ **组件数据没有 ready：**如果子组件中依赖了异步数据，Suspense 可以在异步数据 ready 前展示 fallback，数据 ready 后重新渲染子组件

这次通过一个子组件需要异步获取数据的 demo，学习一下 Suspense 是怎么支持子组件数据 ready 前后的渲染切换的

## 一个简单的 demo
不使用 Suspense 我们一般自己设置 loading 状态，让组件在等待数据期间展示 fallback UI

```tsx
function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('Data fetched!'); // 模拟网络请求
    }, 2000);
  });
}

// 组件
function MyComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchData()
      .then(response => {
        setData(response);
        setLoading(false);
      })
  }, []);

  if (loading) {
    return <div>Loading...</div>; // 加载中状态
  }
  
  return <div>{data}</div>;
}

function App() {
  return (
    <div>
      <h1>Data Fetching Example</h1>
      <MyComponent />
    </div>
  );
}
```

## 使用 Suspense 在等待数据期间展示 fallback UI
Suspense 在等待子组件数据加载期间展示 fallback UI 的原理和动态导入组件的原理类似

1. 在第一次执行时候子组件主动 throw 加载数据的 Promise 对象
2. Suspense catch 该 Promise 对象，如果状态是 pending 则暂停子组件渲染，并渲染 fallback UI
3. 一旦 Promise 对象状态变为 fulfilled，React 重新渲染子组件

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723279710235-4df5cb39-6391-4150-bc35-316227ed4bd0.png)

```tsx
let data; // 使用全局变量

function fetchData() {
  if (data) return data;
  const promise = new Promise(resolve => {
    setTimeout(() => {
      data = 'data fetched!'
      resolve()
    }, 2000)
  })
  throw promise;
}

function Content() {
  fetchData();
  // 使用全局定义的 data
  return <p>{data}</p>
}

function App() {
  return (
    <>
      <h1>Suspense async data</h1>
      <Suspense fallback={'loading data...'}>
        <Content />
      </Suspense>
    </>
  )
}
```

1. `fetchData`在首次调用时候主动 throw 了 Promise 对象
2. 该 Promise 对象被 Suspense catch，对该 Promise 对象追加 `.then`等待其 resolve
3. React 暂停子组件渲染，显示 fallback UI，等待 Promise 对象 resolve
4. 当数据获取到后更新全局 data 变量，然后 resolve Promise 对象
5. Promise 对象追加的 `.then`回调执行，销毁 fallback UI，触发子组件重新渲染

代码使用了全局的 data 对象，非常的不优雅，可以简单调整一下代码

```tsx
function createResouce(promise) {
  let status = 'pending';
  let result;

  promise
    .then((res) => {
      result = res;
      status = 'fulfilled';
    })
    .catch((err) => {
      result = err;
      status = 'rejected';
    });

  return () => {
    if (status === 'pending') {
      throw promise;
    } else if (status === 'fulfilled') {
      return result;
    } else if (status === 'rejected') {
      throw result;
    }
  };
}

function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('data fetched!');
    }, 1000);
  });
}

// 需要在 Content 组件外部定义
const getData = createResouce(fetchData());

function Content() {
  const data = getData();
  return <div>{data}</div>;
}

function App() {
  return (
    <>
      <h1>Suspense async data</h1>
      <Suspense fallback={'loading data...'}>
        <Content />
      </Suspense>
    </>
  );
}
```

这样可以解决全局变量的问题，为了防止在重新渲染过程中再次创建 Promise 对象，导致程序死循环，需要在组件外部定义数据获取函数

## Jotai 实现
[Jotai](https://jotai.org/) 是一个支持 Suspense 特性的小巧的状态管理 library，使用 Jotai 管理异步状态特别方便

```tsx
import { Suspense } from 'react';
import { atom, useAtom } from 'jotai';

function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('data fetched!');
    }, 1000);
  });
}

const dataAtom = atom(async () => {
  const data = await fetchData();
  return data;
});

function Content() {
  const data = useAtom(dataAtom);
  return <div>{data}</div>;
}

function App() {
  return (
    <>
      <h1>Jotai with Suspense</h1>
      <Suspense fallback={'loading data...'}>
        <Content />
      </Suspense>
    </>
  );
}
```

有了上面的知识会不会已经猜出来 Jotai 怎么实现的了

```tsx
const use =
  ReactExports.use ||
  (<T>(
    promise: PromiseLike<T> & {
      status?: 'pending' | 'fulfilled' | 'rejected'
      value?: T
      reason?: unknown
    },
  ): T => {
    if (promise.status === 'pending') {
      throw promise
    } else if (promise.status === 'fulfilled') {
      return promise.value as T
    } else if (promise.status === 'rejected') {
      throw promise.reason
    } else {
      promise.status = 'pending'
      promise.then(
        (v) => {
          promise.status = 'fulfilled'
          promise.value = v
        },
        (e) => {
          promise.status = 'rejected'
          promise.reason = e
        },
      )
      throw promise
    }
  })
```

## use
:::success
React 19 已经正式支持 [use hook](https://react.dev/reference/react/use)

:::

Jotai 的实现最开始有一句判断

```tsx
const use = ReactExports.use || ...
```

`ReactExports.use`是在面向未来兼容 [React canary](https://zh-hans.react.dev/community/versioning-policy#canary-channel) 版本（预发布版本）中原生提供的 [use](https://react.dev/reference/react/use) 功能

> `use` is a React API that lets you read the value of a resource like a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) or [context](https://react.dev/learn/passing-data-deeply-with-context).
>

来看一下未来的使用方式，首先需要安装 React canary 版本

```bash
npm i react@canary react-dom@canary
```

这时候可以在组件内部创建 promise 了，代码逻辑更加内聚

```tsx
import { Suspense, use } from 'react';

function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('data fetched!');
    }, 1000);
  });
}
function Content() {
  const data = use(fetchData());
  return <div>{data}</div>;
}

function App() {
  return (
    <>
      <h1>Suspense async data</h1>
      <Suspense fallback={'loading data...'}>
        <Content />
      </Suspense>
    </>
  );
}
```

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1723280582384-4516e827-e53e-4ac2-8de4-404c8a990c40.gif)

使用之后可以看到 Console 中会有警告，所以现在还不能在线上环境使用，未来可期

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723277677142-34ed2078-7586-43fa-9256-fea93f5f6b8f.png)

## ErrorBoundary 支持
除了 Suspense 外 React 中 ErroBoundary 也是使用同样的原理实现的，只不过 catch 的不是 Promise 而是 Error

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1723280807306-b82e4df7-3e86-41a5-80dc-c3bf21f8c052.png)

创建一个 ErrorBoundary 组件

```tsx
import React, { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新状态，以便下一个渲染可以展示降级UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // 可以将错误报告到错误报告服务
    console.error("Error captured by ErrorBoundary: ", error, info);
  }

  render() {
    if (this.state.hasError) {
      // 可以渲染任何自定义的降级UI
      return <h2>Something went wrong.</h2>;
    }
    return this.props.children; 
  }
}
```

组件获取数据失败，触发 ErrorBoundary 渲染 fallback UI

```tsx
// ...
function App() {
  return (
    <>
      <h1>Suspense async data</h1>
      <ErrorBoundary>
        <Suspense fallback={'loading data...'}>
          <Content />
        </Suspense>
      </ErrorBoundary>
    </>
  );
}
```

