`[use](https://react.dev/reference/react/use)` 是 React 19 引入的新 hook，官方的介绍是 “use is a React API that lets you read the value of a resource like a Promise or context.”

```javascript
const value = use(resource);
```

## 简化异步操作
当组件 UI 渲染依赖异步数据的时候一般需要 useState 和 useEffect 配合，写的代码可能是这样的

```jsx
import { useState, useEffect } from 'react';

function getUses() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, name: 'Alex' },
        { id: 2, name: 'Berlin' },
        { id: 3, name: 'Crimson' },
        { id: 4, name: 'Dani' },
        { id: 5, name: 'Franklin' },
      ]);
    }, 2000);
  });
}

function App() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    getUses().then(data => {
      setUsers(data);
      setIsLoading(false);
    });
  }, []);

  return (
    <div>
      {isLoading ? (
        <p>加载中...</p>
      ) : (
        <ul>
          {users.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default App;
```

使用 Promise 调用 use API 时，它会与 [Suspense](https://zh-hans.react.dev/reference/react/Suspense) 和 [错误边界](https://zh-hans.react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 集成

+ 当传递给 use 的 Promise 处于 pending 时，调用 use 的组件也会挂起
+ 如果调用 use 的组件被包装在 Suspense 边界内，将显示后备 UI
+ 一旦 Promise 被解决，Suspense 后备方案将被使用 use API 返回的数据替换
+ 如果传递给 use 的 Promise 被拒绝，将显示最近错误边界的后备 UI

```jsx
import { Suspense, use } from 'react';

function getUses() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, name: 'Alex' },
        { id: 2, name: 'Berlin' },
        { id: 3, name: 'Crimson' },
        { id: 4, name: 'Dani' },
        { id: 5, name: 'Franklin' },
      ]);
    }, 2000);
  });
}

function UsersList() {
  const users = use(getUses()); // 使用 use Hook 来处理异步数据

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <div>
      <Suspense fallback={<div>加载中...</div>}>
        <UsersList />
      </Suspense>
    </div>
  );
}

export default App;
```

## 可以在 if 条件中使用
在 use() 之前所有 hook 必须在组件顶级使用，如果在 if 或循环条件中使用可能会导致 state 混乱，而 use() 没有这个限制

```jsx
import { useState, Suspense, use } from "react";

function getUses() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, name: "Alex" },
        { id: 2, name: "Berlin" },
        { id: 3, name: "Crimson" },
        { id: 4, name: "Dani" },
        { id: 5, name: "Franklin" },
      ]);
    }, 2000);
  });
}

function getAdmins() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 101, name: "Byron" },
        { id: 102, name: "VV" }
      ]);
    }, 2000);
  });
}

function UsersList(props) {
  let users;
  if (props.type === "admin") {
    users = use(getAdmins());
  } else {
    users = use(getUses());
  }

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <div>
      <Suspense fallback={<div>加载中...</div>}>
        <UsersList type="admin" />
      </Suspense>
    </div>
  );
}

export default App;
```

## 不能在 try...catch 中使用
不能在 try-catch 块中调用 use，最简单方法是使用 Promise.catch 方法做好异常兜底，提供替代值给 use

```jsx
import { Suspense, use } from "react";

function getUses() {
  return new Promise((resolve, reject) => {
    reject(0)
  });
}

function UsersList() {
  const userPromise = getUses().catch(() => {
    return [];
  });

  const users = use(userPromise); // 使用 use Hook 来处理异步数据

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <div>
        <Suspense fallback={<div>加载中...</div>}>
          <UsersList />
        </Suspense>
    </div>
  );
}

export default App;
```

因为 use API 也和 ErrorBoundary 做了集成，因此也可以在组件外层包裹 ErrorBoundary 做错误处理

```jsx
import { Suspense, use } from "react";
import { ErrorBoundary } from "react-error-boundary";

function getUses() {
  return new Promise((resolve, reject) => {
    reject(0)
  });
}

function UsersList() {
  const users = use(getUses()); // 使用 use Hook 来处理异步数据

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <div>
      <ErrorBoundary fallback={<div>出错了</div>}>
        <Suspense fallback={<div>加载中...</div>}>
          <UsersList />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}

export default App;
```

:::info
关于 Suspense 和 use 的实现原理可以参考 [Suspense 支持异步请求](https://www.yuque.com/sunluyong/fe-interview/ginwio7mk3wcryda)

:::

