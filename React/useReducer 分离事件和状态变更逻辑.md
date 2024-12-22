useReducer 是 React 提供的一个 Hook，通常用来管理复杂状态逻辑，它是 Redux 等状态管理库的简化版本，可以在不借助外部库的情况下管理复杂的本地状态。相比于 useState，useReducer 更适合处理那种状态变化遵循特定模式的场景，比如多个子状态之间存在依赖关系或复杂的状态更新逻辑

## 一个简单的 Todo
```jsx
import React, { useState } from 'react';

const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [nextId, setNextId] = useState(1);
  const [newTodo, setNewTodo] = useState('');
  
  const handleAddTodo = () => {
    const newTodoItem = {
      id: nextId,
      text: newTodo,
      completed: false,
    };
    
    setTodos([...todos, newTodoItem]);
    setNextId(nextId + 1);
    setNewTodo(''); // 清空输入框
  };

  const handleToggleTodo = (id) => {
    const updatedTodos = todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    setTodos(updatedTodos);
  };

  const handleRemoveTodo = (id) => {
    const updatedTodos = todos.filter(todo => todo.id !== id);
    setTodos(updatedTodos);
  };

  return (
    <div>
      <input
        type="text"
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Enter a new todo"
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => handleToggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => handleRemoveTodo(todo.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoApp;
```

代码中使用了 handleAddTodo、handleToggleTodo、handleRemoveTodo 三个事件处理程序来修改 state，达到更新 UI 的效果。随着程序复杂度的提升，会发现修改 state 的逻辑分散在应用的各个角落，状态变化原因难以追溯

## useReducer 介绍
useReducer 的设计目标之一确实是将事件处理程序（即触发状态更新的操作）和具体的状态变更逻辑分离开来，也就是让 React 组件数据流稍微变化一下

| ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1721952931010-5e5adadf-1e0a-4b10-b58c-3ab9b1012a4c.png) | ![](https://cdn.nlark.com/yuque/0/2024/png/87727/1721952983223-ed730c4f-6e30-411c-9513-e304ce85cec3.png) |
| --- | --- |


看个简单的示例

```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

// 纯函数，根据当前的状态和接收的事件，返回新的状态
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  // state：当前的状态，dispatch：分发action的函数
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        Increment
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        Decrement
      </button>
    </div>
  );
}

export default Counter;
```

### 基础 API
```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

useReducer 接收三个参数：

1. reducer: 一个用于处理状态更新的函数，接收当前状态和一个动作（action），并返回新的状态
2. initialState: 初始状态，可以是一个值或一个函数，如果是函数则会返回函数执行后的值
3. init: 一个懒初始化函数，如果传递了 init，则 initialState 将作为惰性初始化函数的参数

当初始状态需要通过某个计算得到时，可以使用惰性初始化：

```jsx
function init(initialCount) {
  return { count: initialCount };
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);

  // 剩余部分与之前的示例相同
}
```

### action
action 表示动作对象，使用以下约定俗成的数据结构

```jsx
{
  type: 'ACTION_TYPE',
  payload: /* 数据，用于状态更新 */
}
```

+ type（必需）：一个字符串，描述要执行的操作类型
+ payload（可选）：更新的状态需要的部分数据

### reducer
reducer 名称源自于 JavaScript 数组的 reduce() 方法，传给 reduce() 方法的参数称之为 reducer

```jsx
const sum = numbers.reduce((accumulator, currentValue) => {
    return accumulator + currentValue;
}, 0);  // 初始值为 0
```

根据 accumulator 和 currentValue 返回下一次计算结果，而 React 中的 reducer 是一样的，都接受目前的状态和 action ，然后返回下一个状态

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}
```

## 使用 useReducer 改造 Todo
```jsx
import React, { useReducer } from 'react';

const initialState = {
  todos: [],
  nextId: 1
};

const todoReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, { id: state.nextId, text: action.payload, completed: false }],
        nextId: state.nextId + 1
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        )
      };
    case 'REMOVE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    default:
      return state;
  }
};

const TodoApp = () => {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [newTodo, setNewTodo] = useState('');

  const handleAddTodo = () => {
    dispatch({ type: 'ADD_TODO', payload: newTodo });
    setNewTodo('');
  };

  const handleToggleTodo = (id) => {
    dispatch({ type: 'TOGGLE_TODO', payload: id });
  };

  const handleRemoveTodo = (id) => {
    dispatch({ type: 'REMOVE_TODO', payload: id });
  };

  return (
    <div>
      <input
        type="text"
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Enter a new todo"
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {state.todos.map((todo) => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => handleToggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => handleRemoveTodo(todo.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoApp;
```

这样所有状态处理的逻辑都被维护在了 `todoReducer`中，随着应用复杂度上升，代码仍然可以保持可维护性

## 使用 use-immer 简化 useReducer
因为每次需要返回新的 state，上面的代码中充斥着这样的语句，构造新的 state 对象

```jsx
{
  ...state,
  todos: state.todos.map(todo =>
    todo.id === action.payload 
    ? { ...todo, completed: !todo.completed } : todo
  )
}
```

对于 state 层级深、数据结构复杂，可以利用不可变数据 library Immer 提供的 [use-immer](https://www.npmjs.com/package/use-immer?activeTab=readme) 简化 state 修改逻辑

```jsx
import React, { useState } from 'react';
import { useImmerReducer } from 'use-immer';

const initialState = {
  todos: [],
  nextId: 1
};

const todoReducer = (draft, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      draft.todos.push({ 
        id: draft.nextId, 
        text: action.payload, 
        completed: false 
      });
      draft.nextId += 1;
      break;
    case 'TOGGLE_TODO':
      const todoToToggle = draft.todos.find(todo => todo.id === action.payload);
      if (todoToToggle) {
        todoToToggle.completed = !todoToToggle.completed;
      }
      break;
    case 'REMOVE_TODO':
      draft.todos = draft.todos.filter(todo => todo.id !== action.payload);
      break;
    default:
      break;
  }
};

const TodoApp = () => {
  const [state, dispatch] = useImmerReducer(todoReducer, initialState);
  const [newTodo, setNewTodo] = useState('');

  const handleAddTodo = () => {
    dispatch({ type: 'ADD_TODO', payload: newTodo });
    setNewTodo('');
  };

  const handleToggleTodo = (id) => {
    dispatch({ type: 'TOGGLE_TODO', payload: id });
  };

  const handleRemoveTodo = (id) => {
    dispatch({ type: 'REMOVE_TODO', payload: id });
  };

  return (
    <div>
      <input
        type="text"
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Enter a new todo"
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {state.todos.map((todo) => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => handleToggleTodo(todo.id)}
            >
              {todo.text}
            </span>
            <button onClick={() => handleRemoveTodo(todo.id)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoApp;
```

