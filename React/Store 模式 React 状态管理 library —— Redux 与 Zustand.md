Redux 和 Zustand 都使用 Store 模式，而且两者目前在 npm trends 中也处于领先

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722683161758-17d2ea17-0dda-4680-94c2-4fe43c4a4de6.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)

通过一个 Todo 的 demo 演示下两者在 Store 模式下如何管理状态数据，处理异步任务

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722764154074-ee7a85fe-d043-49c0-b244-c236ccc37e49.png)

### 回顾 Store 模式
Store 模式通过集中式的存储来管理应用的状态，这种模式的基本思想是将应用的所有状态存储在一个单一的地方，通常称为“store”。组件通过与 store 的交互来读取和更新状态，而不是直接在组件内部管理状态。Store 模式有几个核心概念

+ Store：保存应用所有 state 的对象。它是只读的，唯一改变 state 的方法是触发 actions
+ Actions：描述应用行为的普通对象。它们是 store 进行状态更新的唯一途径，Actions 通常包含一个 type 字段来标识动作的类型，以及其他必要的数据
+ Dispatch：触发 action 的过程。组件通过 dispatch 来发送 actions 给 store，以触发 state 更新
+ Reducers：接受当前的 state 和一个 action，然后返回一个新的 state，Reducers 描述了 action 如何改变 state
+ Selectors：用于从 store 中选择片段数据，使组件获得最少依赖的状态数据

## Redux
### <font style="color:rgb(28, 30, 33);">安装依赖</font>
```bash
npm install @reduxjs/toolkit react-redux
```

+ react-redux：将 React 应用与 Redux 状态管理结合起来，Provider 由其提供
+ @reduxjs/toolkit：减少直接使用 redux 需要手写的“样板代码”，提供简化标准的 Redux 任务的API

[为什么 Redux Toolkit 是如今使用 Redux 的方式](https://cn.redux.js.org/introduction/why-rtk-is-redux-today)

### 创建文件结构
```plain
src/
├── features/
│   └── todos/
│       ├── filterSlice.ts
│       ├── TodoApp.tsx
│       ├── todoFilter.ts
│       ├── TodoList.tsx
│       └── TodoSlice.tsx
└── store/
    └── store.ts
└── index.tsx
```

### 定义 state 和 reducers
Redux 在诞生之初就因为样板代码过多而饱受诟病，Redux Toolkit（RTK）诞生的一个重要目的就是简化 Redux 应用的开发过程，减少样板代码、改进开发体验

在 Redux Tookit 中把特定功能的 initalState、reducers 和 action creators 集成在了一个文件内，这个文件一般根据其功能被称为 **xxxSlice**，整棵状态树（store）由多个小片段（state slices）拼成，最终可以通过 `configureStore` 整合成一个集中的 Store

#### createSlice 方法的参数
+ **name**: string: 定义这个 slice 的名字，它将作为生成的 action types 的前缀
+ **initialState**: T: 定义该 slice 的初始状态，类型 `T` 表示初始状态的类型
+ **reducers**: { [key: string]: (state: T, action: PayloadAction<any>) => void }:
    - 定义这个 slice 处理的 reducers，这些 reducers 是一组处理特定 slice 状态的纯函数
    - 每个 reducer 函数接收两个参数：当前的状态 `state` 和 `action`，返回新的 state

createSlice 为 Redux 应用提供了良好的类型支持，对在写代码过程中问题发现特别有帮助

#### createSlice 返回的对象
+ **reducer**: 把所有的 reducer 组合在一起，供 store 使用
+ **actions**: 自动生成的 action creators，每个 action creator 的名字与对应的 reducer 的 key 一致
+ **name**: slice 的名称，用于作用域 action types

#### filterSlice
首先定义一个 Todo 状态筛选的 slice

```tsx
// features/todos/filterSlice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

export enum IFilter {
  ALL = 'ALL',
  COMPLETED = 'COMPLETED',
  ACTIVE = 'ACTIVE',
}

const initialState = {
  filter: IFilter.ALL,
}

const filterSlice = createSlice({
   // 定义 slice 的名字，该 slice 下的 action type 前缀默认设置为 filter，避免命名冲突
  name: 'filter',
  initialState,
  reducers: {
    setFilter: (state, action: PayloadAction<IFilter>) => {
      state.filter = action.payload;
    },
  },
});

export const { setFilter } = filterSlice.actions;
export default filterSlice.reducer;
```

createSlice 极大的简化了 Action types 定义、Action Creators 定义、reducer 中的 switch(actions type) 样板代码。如果不使用 Redux Toolkit，上面代码需要这么写

```tsx
// src/store/actionTypes.ts
export const SET_FILTER = 'SET_FILTER'; // action 类型定义
export enum IFilter {
  ALL = 'ALL',
  COMPLETED = 'COMPLETED',
  ACTIVE = 'ACTIVE',
}

// src/store/actions.ts
import { SET_FILTER, IFilter } from './actionTypes';

// action creator
export const setFilter = (filter: IFilter) => ({
  type: SET_FILTER,
  payload: filter,
});

// src/store/reducer.ts
import { SET_FILTER, IFilter } from './actionTypes';
interface FilterState {
  filter: IFilter;
}

const initialState: FilterState = {
  filter: IFilter.ALL,
};

const filterReducer = (
  state = initialState, 
  action: { type: string; payload: IFilter }): FilterState => {
  switch (action.type) {
    case SET_FILTER:
      return {
        ...state,
        filter: action.payload,
      };
    default:
      return state;
  }
};

export default filterReducer;
```

虽然使用了 Redux Toolkit 后 Action 看起来还是很繁琐，但 Redux 很坚持 Action 的使用

> Actions 是用来描述在 app 中发生了什么的普通对象，并且是描述突变数据意图的唯一途径。很重要的一点是**不得不 dispatch 的 action 对象并非是一个样板代码，而是 Redux 的一个 基本设计原则。**
>
> 不少框架声称自己和 Flux 很像，只不过缺少了 action 对象的概念，在可预测性方面这是从 Flux 或 Redux 的倒退。如果没有可序列化的普通对象 action，便无法记录或重演用户会话，也无法实现 带有时间旅行的热重载。如果你更喜欢直接修改数据，那你并不需要使用 Redux 。
>

#### todoSlice
todoSlice 中需要实现 todo 的添加、修改删除功能，而这些 action 都是异步的，就 Redux 本身而言，Redux store 对异步逻辑一无所知，它只知道如何同步 dispatch action，通过调用 reducer 函数更新状态，并通知 UI 某些事情发生了变化，因此任何异步都必须发生在 store 之外，这就是 Redux middleware 的作用

Thunk middleware 拦截所有通过 `dispatch` 发出的 actions，并检查 action 是否是一个函数

+ 如果 action 是一个对象（普通的 action），Thunks 中间件会直接将其传递给下一个 middleware 或者 reducer
+ 如果 action 是一个函数（Thunk），中间件将执行这个函数并传递 `dispatch` 和 `getState` 函数作为参数给它。函数的逻辑可以同步或异步地进行一系列动作，包括额外的 dispatch 操作或状态读取

当所有中间件处理完成后，最终的 actions 会传递给 Redux store 的 reducer，从而更新状态，流程如下所示

![](https://cdn.nlark.com/yuque/0/2024/gif/87727/1722765019109-5c9b3b88-abab-4e21-84b4-bde51f1d8fca.gif)

Redux Toolkit 的默认设置了支持异步的 thunk middleware，通过`createAsyncThunk`方法可以定义和处理异步 actions，其接受一个字符串类型的 action type 和一个返回 promise 的回调函数。这个回调函数是异步操作的核心（例如，API 请求）。`createAsyncThunk` 会生成一个异步操作，并自动生成对应的 action types 和 action creators

```tsx
// features/todos/todoSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from "@reduxjs/toolkit";

// 类型统一使用 IXxxZzz 的格式,方便识别
export type ITodo = {
  id: number;
  title: string;
  completed: boolean;
}

export enum IStatus {
  IDLE = 'idle',
  LOADING = 'loading',
  SUCCEEDED = 'succeeded',
  FAILED = 'failed',
}

type ITodoState = {
  todos: ITodo[];
  status: IStatus;
}

const initialState: ITodoState = {
  todos: [],
  status: IStatus.IDLE,
};

// 模拟异步请求
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

// 使用 createAsyncThunk 创建异步 Action
export const addTodoAsync = createAsyncThunk<ITodo, string>(
  'todos/addTodoAsync',
  async (title: string) => {
    await delay(1000);
    return { id: Date.now(), title, completed: false };
  }
);

export const removeTodoAsync = createAsyncThunk(
  'todos/removeTodoAsync',
  async (id: number) => {
    await delay(1000);
    return id;
  }
);

export const toggleTodoAsync = createAsyncThunk(
  'todos/toggleTodoAsync',
  async (id: number) => {
    await delay(1000);
    return id;
  }
);

export const todoSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: {},
  // 异步 action 的处理在 extraReducers 中
  extraReducers: (builder) => {
    // builder.addCase 可以理解为 switch...case 的简写
    builder
      .addCase(addTodoAsync.pending, (state) => {
        state.status = IStatus.LOADING;
      })
      .addCase(addTodoAsync.fulfilled, (state, action: PayloadAction<ITodo>) => {
        state.status = IStatus.SUCCEEDED;
        // 直接操作当前 state 对象，而不是返回新的 state 对象
        state.todos.push(action.payload);
      })
      .addCase(removeTodoAsync.pending, (state) => {
        state.status = IStatus.LOADING;
      })
      .addCase(removeTodoAsync.fulfilled, (state, action: PayloadAction<number>) => {
        state.status = IStatus.SUCCEEDED;
        state.todos = state.todos.filter((todo) => todo.id !== action.payload);
      })
      .addCase(toggleTodoAsync.pending, (state) => {
        state.status = IStatus.LOADING;
      })
      .addCase(toggleTodoAsync.fulfilled, (state, action: PayloadAction<number>) => {
        state.status = IStatus.SUCCEEDED;
        const todo = state.todos.find((todo) => todo.id === action.payload);
        if (todo) {
          todo.completed = !todo.completed;
        }
      })
  },
});

export default todoSlice.reducer;
```

Redux 强调在 reducer 中不允许改变当前的 State，需要生成新的 State，这也是 Redux 用户最常犯的一个错误，而 Redux Toolkit 中集成 [Immer](https://immerjs.github.io/immer/)，这样就可以“直接”操作 State，Immer 完成将代码转换为安全不可变更新的工作

### 配置 store
Redux Tookit 提供了 configureStore 完成 reducer 的合并，并且启用了 thunk 等常用中间件

```tsx
// store/store.ts
import { configureStore } from '@reduxjs/toolkit';
import todoReducer from '../features/todos/todoSlice';
import filterReducer from '../features/todos/filterSlice';

// 合并多个 reducer，state 的初始值也随着 reducer 被转入
const store = configureStore({
  reducer: {
    todos: todoReducer,
    filter: filterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export default store;
```

### 创建组件
#### TodoFilter
```tsx
// features/todos/TodoFilter.tsx
import { IFilter } from './filterSlice';

type IProps = {
  setFilter: (filter: IFilter) => void;
};

export default function TodoFilter({ setFilter }: IProps) {
  return (
    <div>
      <label>
        <input type="radio" defaultChecked name="filter" onChange={() => setFilter(IFilter.ALL)}/> All 
      </label>
      <label>
        <input type="radio" name="filter" onChange={() => setFilter(IFilter.ACTIVE)} /> Active 
        </label>
      <label>
        <input type="radio" name="filter" onChange={() => setFilter(IFilter.COMPLETED)} /> Completed 
      </label>
    </div>
  );
};
```

#### TodoList
在组件中调用 `useDispatch` 会返回 Redux store 的 `dispatch` 函数，通过 `dispatch` 函数，可以发送（dispatch）actions 来触发 state 的更新

```tsx
// features/todos/TodoList.tsx
import { useDispatch } from 'react-redux';
import { toggleTodoAsync, removeTodoAsync, ITodo } from './todoSlice';
import { AppDispatch } from '../../store/store';

export default function TodoList({todos}: {todos: ITodo[]}){
  const dispatch = useDispatch<AppDispatch>();
  return (
    <ul>
      {
        todos.map(todo => (
          <li key={todo.id} style={{textDecoration: todo.completed ? 'line-through' : 'none'}}>
            <input 
              type="checkbox" 
              checked={todo.completed} 
              onChange={() => dispatch(toggleTodoAsync(todo.id))} />
            <span>{todo.title}</span>
            <button 
              onClick={() => dispatch(removeTodoAsync(todo.id))}>
              Remove
            </button>
          </li>
        ))
      }
    </ul>
  );
}
```

#### TodoApp
通过 `useSelector` 可以从 Redux store 中访问组件所需的部分 state，`useSelector` 会自动订阅 Redux store 的更新，当所选择的 state 更新时，组件会重新渲染，这样通过选择性地从 Redux store 获取所需的 state，避免了不必要的 re-renders，提升了组件的性能，[通过 useSelector 来根据当前的状态派生出组件需要用的状态](https://cn.redux.js.org/usage/deriving-data-selectors)，可以显著提升代码可维护性，在后续其它状态管理 library 后会反复出现

```tsx
// features/todos/TodoApp.tsx
import { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { RootState, AppDispatch } from '../../store/store';
import { addTodoAsync, IStatus } from './todoSlice';
import { setFilter, IFilter } from './filterSlice';
import TodoList from './TodoList';
import TodoFilter from './TodoFilter';

export default function TodoApp() {
  const dispatch = useDispatch<AppDispatch>();
  const todos = useSelector((state: RootState) => state.todos.todos);
  const filter = useSelector((state: RootState) => state.filter.filter);
  const status = useSelector((state: RootState) => state.todos.status);
  const [title, setTitle] = useState('');

  const handleAddTodo = () => {
    if (title.trim() !== '') {
      dispatch(addTodoAsync(title));
      setTitle('');
    }
  };

  const handleFilterChange = (filter: IFilter) => {
    dispatch(setFilter(filter));
  };

  const filterdTodos = todos.filter((todo) => {
    switch (filter) {
      case IFilter.ALL:
        return true;
      case IFilter.ACTIVE:
        return !todo.completed;
      case IFilter.COMPLETED:
        return todo.completed;
      default:
        return true;
    }
  });

  return (
    <div className="todos-app">
      <h1>Redux Todo</h1>
      <input
        value={title}
        placeholder="Add a new todo..."
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            handleAddTodo();
          }
        }}
        onChange={(e) => setTitle(e.target.value)} />
      <button onClick={handleAddTodo}>Add Todo</button>
      {status === IStatus.LOADING && <p>Loading...</p>}
      <TodoFilter setFilter={handleFilterChange} />
      <TodoList todos={filterdTodos} />
    </div>
  );
};
```

### 将一切连接到应用程序
`Provider` 是 React-Redux 库提供的一个高阶组件，用于将 Redux store 和 React 应用连接起来。它的主要作用是将 Redux store 提供给整个组件树中的所有组件，使得组件可以通过 React-Redux 提供的 hooks（`useSelector` 和 `useDispatch` 可以访问 store 的数据、派发 action 正是因为 Provider）来访问 Redux store 和触发 actions。组件数据的共享就是这样实现的

```tsx
// ./index.tsx
import { Provider } from'react-redux';
import store from './store/store';
import TodoApp from './features/todos/TodoApp';

export default function() {
  return (
    <Provider store={store}>
      <TodoApp />
    </Provider>
  );
}
```

## Zustand
理解了 Redux 的各种概念和数据流程之后，再看其他状态管理 library 会简单很多，尤其是同样使用 Store 模式的 Zustand

### 安装依赖
```bash
npm install zustand
```

### 创建目录结构
```plain
src/
├── components/
│   ├── TodoApp.tsx
│   ├── TodoList.tsx
│   └── TodoFilter.tsx
└── store/
    └── todoStore.ts
└── index.tsx
```

### 定义 store
Zustand 中 store 的定义和 Redux 的 slice 非常相似，虽然支持多 Store 模式不强制单一数据源，但 Zustand 鼓励使用 Single Store 模式，对于复杂的业务逻辑，和 Redux 一样支持 Slice Pattern

> Your applications global state should be located in a single Zustand store.
>
> If you have a large application, Zustand supports [splitting the store into slices](https://docs.pmnd.rs/zustand/guides/slices-pattern).
>

Zustand 代码简洁之处还来自于同步/异步 Action 书写方式保持了一致性

```tsx
// store/todoStore.ts
import { create } from 'zustand';

export type ITodo = {
  id: number;
  title: string;
  completed: boolean;
}

export enum IFilter {
  ALL = 'ALL',
  ACTIVE = 'ACTIVE',
  COMPLETED = 'COMPLETED',
}

export enum IStatus {
  IDLE = 'idle',
  LOADING = 'loading',
  SUCCEEDED = 'succeeded',
  FAILED = 'failed',
}

type ITodoStore = {
  todos: ITodo[];
  status: IStatus;
  filter: IFilter;
  addTodo: (title: string) => void;
  toggleTodo: (id: number) => void;
  removeTodo: (id: number) => void;
  setFilter: (filter: IFilter) => void;
}

// 模拟异步
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

export const useTodoStore = create<ITodoStore>(set => ({
  todos: [],
  status: IStatus.IDLE,
  filter: IFilter.ALL,
  addTodo: async (title: string) => {
    set({ status: IStatus.LOADING });
    await delay(1000);
    set(state => ({
      todos: [...state.todos, { id: Date.now(), title, completed: false }],
      status: IStatus.SUCCEEDED,
    }));
  },
  toggleTodo: async (id: number) => {
    set({ status: IStatus.LOADING });
    await delay(1000);
    set(state => ({
      todos: state.todos.map(todo => todo.id === id ? { ...todo, completed: !todo.completed } : todo),
      status: IStatus.SUCCEEDED,
    }));
  },
  removeTodo: async (id: number) => {
    set({ status: IStatus.LOADING });
    await delay(1000);
    set(state => ({
      todos: state.todos.filter(todo => todo.id !== id),
      status: IStatus.SUCCEEDED,
    }));
  },
  setFilter: (filter: IFilter) => set({ filter }),
}));
```

### 创建组件
#### TodoFilter
```tsx
// components/TodoFilter.tsx
import { IFilter } from '../store/todoStore';

type IProps = {
  setFilter: (filter: IFilter) => void;
}

export default function TodoFilter({ setFilter }: IProps) {
  return (
    <div>
      <label>
        <input type="radio" defaultChecked name="filter" onChange={() => setFilter(IFilter.ALL)} /> All
      </label>
      <label>
        <input type="radio" name="filter" onChange={() => setFilter(IFilter.ACTIVE)} /> Active
      </label>
      <label>
        <input type="radio" name="filter" onChange={() => setFilter(IFilter.COMPLETED)} /> Completed
      </label>
    </div>
  );
}
```

#### TodoList
```tsx
import { useTodoStore, ITodo } from "../store/todoStore";

export default function TodoList({todos}: {todos: ITodo[]}) {
  // 可以理解为一个 selector，获取组件需要的状态数据与方法
  const { toggleTodo, removeTodo } = useTodoStore(state => ({
    toggleTodo: state.toggleTodo,
    removeTodo: state.removeTodo
  }));

  return (
    <ul>
      {
        todos.map(todo => (
          <li key={todo.id} style={{textDecoration: todo.completed ? 'line-through' : 'none'}}>
            <input 
              type="checkbox" 
              checked={todo.completed} 
              onChange={() => toggleTodo(todo.id)} />
            <span>{todo.title}</span>
            <button onClick={() => removeTodo(todo.id)}>Remove</button>
          </li>
        ))
      }
    </ul>
  );
}
```

#### TodoApp
```tsx
import { useState } from 'react';
import { useTodoStore, IFilter, IStatus } from '../store/todoStore';
import TodoList from './TodoList';
import TodoFilter from './TodoFilter';

export default function TodoApp() {
  const [title, setTitle] = useState('');
  // 再一次使用 selector 获取需要的状态和方法
  const { todos, filter, status, addTodo, setFilter } = useTodoStore(state => ({
    todos: state.todos,
    filter: state.filter,
    status: state.status,
    addTodo: state.addTodo,
    setFilter: state.setFilter,
  }));

  const handleAddTodo = async () => {
    if (!title.trim()) return;
    await addTodo(title);
    setTitle('');
  }

  const filteredTodos = todos.filter(todo => {
    switch (filter) {
      case IFilter.ALL:
        return true;
      case IFilter.ACTIVE:
        return!todo.completed;
      case IFilter.COMPLETED:
        return todo.completed;
      default:
        return true;
    }
  });

  return (
    <div>
      <h1>Zustand Todo</h1>
      <input
        value={title}
        placeholder="Add a new todo..."
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            handleAddTodo();
          }
        }}
        onChange={(e) => setTitle(e.target.value)} />
      <button onClick={handleAddTodo}>Add Todo</button>
      {status === IStatus.LOADING && <p>Loading...</p>}
      <TodoFilter setFilter={setFilter} />
      <TodoList todos={filteredTodos} />
    </div>
  );
}
```

Zustand 没有和 Redux 一样使用 Context API，因此不需要 Provider 包裹，TodoApp 可以直接使用

### Redux VS Zustand
虽然同样使用 Store 模式和不可变数据模式，但可以看出来即使使用了可以减少样板代码的 Redux Toolkit，Zustand 的代码量仍然远远少于使用了 Redux。Zustand 没有使用 Context API 不用多一层包裹 HOC，而且除了 Store 几乎没有其它概念，因此代码也不需要理解就能直接使用

Redux 提供了一个明确的架构和严格的约定来管理应用状态，这对大型应用尤为重要。它规定了状态管理中的各个部分应该如何组织和交互，包括 actions、reducers 和 store。通过这些约定大大提高了代码的可读性和一致性。只要了解了 Redux 的基础概念（成本确实有点高）任何开发人员都可以快速理解和上手一个新的 Redux 项目，因为它们知道在哪里查找状态逻辑和如何扩展功能

同样的 UI 功能 Zustand 可以用更少的代码实现也并不是因为 library 本身做了很多，毕竟 Zustand gzip 后只有 1.2k。Zustand 的简单来自于对样板代码的缩减，在数据流程中移除了 Action、和 dispatch，修改为了函数传递和调用，但没有样板代码的代价就是样板代码解决的问题，可以被开发者写到任意地方，个人对 Redux 的看法也几经波折

+ 在 Redux 出世不久时惊为天人，从其它状态管理 library 的相似概念中也能看到 redux 的影响力，而且切实解决了复杂单页应用的状态管理问题
+ 过了两三年进入疲劳期，没完没了的样板代码
+ 最近维护的代码越来越多，使用了不同的解决方案，每天不停的寻找状态为什么变了，当前的状态究竟是什么，越是想念 Redux 的好，因此才萌生了编写本文的念头

相信 Zustand 的作者对这个问题也应该有同感，在 Zustand 中可以通过中间件支持 Redux 的时间旅行、使用 Slice Pattern & 合并 Store，甚至有一个 [Redux-like patterns](https://docs.pmnd.rs/zustand/guides/flux-inspired-practice#redux-like-patterns) 的支持。流行的状态管理 library 之间没有高低，但开发人员的水平有，如果认同 [Redux 三大原则](https://cn.redux.js.org/understanding/thinking-in-redux/three-principles)

+ 单一数据源：整个应用的 全局 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一 store 中
+ 不可变数据：State 是只读的 唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象
+ 使用纯函数来执行修改：为了描述 action 如何改变 state tree，需要编写纯函数的 reducers

在职业生涯初期应该尽可能多使用 Redux，这样可以保证应用可读性的下限，当掌握了其数据流转过程时候再和团队做好约定，切换到 Zustand 编码提速



:::info
+ [Redux、Zustand、Mobx、Valtio、Recoil、jotai、XState 状态管理怎么选 —— 基础概念](https://juejin.cn/post/7399276698619035657)
+ [Store 模式 React 状态管理 library —— Redux 与 Zustand](https://juejin.cn/post/7399276698619052041)
+ 响应模式 React 状态管理 library —— MobX 与 Valtio
+ 原子模式 React 状态管理 library —— Recoil 与 Jotai
+ 状态机模式 React 状态管理 library —— XState

:::

