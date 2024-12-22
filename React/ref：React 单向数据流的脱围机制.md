当希望组件“记住”某些信息，但又不想让这些信息触发新的渲染时，可以使用 ref

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('你点击了 ' + ref.current + ' 次！');
  }

  return (
    <button onClick={handleClick}>
      点击我！
    </button>
  );
}
```

`ref.current`<font style="color:rgb(35, 39, 47);"> 属性可以访问该 ref 的当前值，当 ref.current 被修改时不会触发组件的重新渲染，但 React 组件重新渲染时候回保留当前的 ref，其内部正是基于 useState 实现</font>

```jsx
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

第一次渲染期间，useRef 返回 `{ current: initialValue }`。 该对象由 React 存储，因此在下一次渲染期间 ref 对象可以被保留。而该 state 设置函数是用不到的，因为 ref 可以直接操作其 current 属性

## 使用 ref 操作 DOM
ref 可以指向任何值，但 ref 最常见的用法是访问 DOM 元素，由于 React 会根据渲染结果自动处理更新 DOM ，因此在组件中通常不需要操作 DOM。但有些操作必须 React 无法代劳，需要开发者直接操作 DOM，比如让一个节点获得焦点、滚动到它或测量它的尺寸和位置等，所以你需要一个指向 DOM 节点的 ref 来实现 React 和外部系统或浏览器交互，因此 ref 也被称为 React 单向数据流的脱围机制



当把 ref 放在像 <input /> 这样输出浏览器元素的内置组件上时，React 会将该 ref 的 current 属性设置为相应的 DOM 节点

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      {/* 通过 ref 属性设置 */}
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

## ref 回调函数
一个组件内可以包含多个 ref，在组件顶层预定义即可，但当为列表中的每一项都绑定 ref 时候，无法预知需要多少个 ref

```jsx
<ul>
  {items.map((item) => {
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

这样的调用方式是不行的，因为 React Hook 只能在组件顶层调用，不能在条件语句、循环语句、map() 函数中调用，ref 也可以使用函数的方式初始化

```jsx
<div ref={(node) => {
  // node 表示当前的 DOM 元素
  console.log(node);

  return () => {
    // 清理函数，和 useEffect 返回值作用类似
    console.log('Clean up', node)
  }
}}>
```



看一个简单的例子

```jsx
import React, { useRef, useEffect } from 'react';

const ListWithRefs = ({ items }) => {
  // 使用 useRef 创建一个对象来存储每个列表项的 DOM 引用
  const listItemRefs = useRef({});

  // 使用 useEffect 在组件渲染后访问每个 DOM 节点
  useEffect(() => {
    Object.values(listItemRefs.current).forEach((item) => {
      if (item) {
        console.log(`DOM node for item ${item.textContent}`);
      }
    });
  }, [items]);

  // ref 回调函数
  const setListItemRef = (index) => (node) => {
    if (node) {
      listItemRefs.current[index] = node;
    }
  };

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index} ref={setListItemRef(index)}>
          {item}
        </li>
      ))}
    </ul>
  );
};
```

## 访问 React 组件的 DOM 节点
ref 放在原生 html 标签上时，React 会将 ref.current 属性设置为相应的 DOM 节点，但如果把 ref 放在 React 组件上，默认会返回 null。React 不允许组件访问其它组件的 DOM 节点，甚至自己的子组件也不行，手动操作另一个组件的 DOM 节点组件强依赖于其它组件的内部实现，在开发环境 React 还会给出错误提示

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1722070311787-b60c3b4f-5729-4b0e-b085-6687d3b0436b.png)

如果一个组件希望自己内部的 DOM 被其它组件获取，可以选择使用 forwardRef 来主动暴露自己 DOM 的 ref

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```



看一个简单的示例

```jsx
import { forwardRef, useRef } from 'react';

// 创建一个简单的输入组件，将 ref 转发给 input 元素
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    // 通过 ref 直接访问子组件的 input 元素
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

## 防止过度暴露
在上面的例子中 MyInput 暴露了原始的 DOM 元素 input，这让父组件可以对其调用 focus()。然而这也让父组件能够做其他事情 —— 比如改变其 CSS 样式等。在一些情况下如果需要限制暴露的功能，可以在 forwardRef 中用 useImperativeHandle

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 该对象作为 ref.current 的值
    setFocus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.setFocus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

这样 ref.current 只能拿到 useImperativeHandle 中返回的对象，获取不到真实的 DOM 节点，组件可以更加稳定

## React 什么时候设置的 refs
React 组件更新分为两个阶段

+ 渲染阶段（reconciliation）：这里的渲染不是指浏览器的渲染，而是 React 执行开发者写的组件渲染代码，计算出最新的组件树，并生成一系列变更（effect list），但并不实际应用这些变更
+ 提交阶段（commit）：这个阶段是 React 将需要做的变更通过 DOM API 提交给浏览器的阶段，对真实的 DOM 进行变更，并调用生命周期方法，处理由新的状态或属性触发的副作用（如 useEffect 的回调）

通常不需要在渲染阶段访问 refs，在第一次渲染期间，DOM 节点尚未创建，因此 ref.current 将为 null。在组件更新的过程中，需要更新的 DOM 节点还没有更新，读取不到新的内容

React 在提交阶段设置 ref.current，在更新 DOM 之前 React 将受影响的 ref.current 值设置为 null，**更新 DOM 后 React 立即将 ref 设置到相应的 DOM 节点**，因此一般在事件处理程序或者 useEffect 中使用 ref

## 使用 refs 操作 DOM 最佳实践
Refs 作为一种脱围机制，通常是为了处理那些不能通过状态和属性解决的情况，例如例如与第三方库集成、管理焦点、动态测量 DOM 元素的尺寸和位置等

应该避免对 React 通过 props 和 state 可以控制、管理的 DOM 节点进行破坏性操作，例如修改、添加子元素、删除子元素等，滥用 refs 可能会导致代码难以维护和 debug

  


  


