`Object.defineProperty` 和 `Proxy` 是 JavaScript 中用于拦截和处理对象属性读写的两种强大工具。Vue.js 的响应式设计正是依赖了这两个特性

## <font style="color:rgb(51, 51, 51);">Object.defineProperty</font>
Object.defineProperty 方法直接在对象上定义一个新属性，或者修改对象的现有属性，并返回这个对象，基础语法

```javascript
Object.defineProperty(obj, prop, descriptor)
```

descriptor 对象可以包含以下键值：

1. 数据描述符:
    - value: 该属性的值，默认为 undefined
    - writable: 属性的值是否可以被修改，默认为 false
    - enumerable: 属性是否可以通过 for...in 循环或 Object.keys 枚举，默认为 false
    - configurable: 属性描述符是否可以被修改，以及属性是否可以被删除，默认为 false
2. 存取描述符:
    - get: 属性的 getter 函数，如果没有 getter，则为 undefined
    - set: 属性的 setter 函数，如果没有 setter，则为 undefined

```javascript
let obj = {};
Object.defineProperty(obj, 'name', {
  value: 'John Doe',
  writable: false,      // 不可修改
  enumerable: true,     // 可枚举
  configurable: false   // 不可配置
});

console.log(obj.name); // John Doe

obj.name = 'Byron Sun';
console.log(obj.name); // John Doe, 因为 writable: false
```



Vue 2 中，Object.defineProperty 被用来实现响应式数据绑定，通过定义 getter 和 setter，可以在数据变化时自动触发相应的更新逻辑

```javascript
function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    get() {
      console.log(`Get ${key}: ${val}`);
      return val;
    },
    set(newVal) {
      console.log(`Set ${key}: ${newVal}`);
      if (val !== newVal) {
        val = newVal;
        // 这里可以添加额外通知逻辑，例如视图更新
      }
    }
  });
}

let data = { message: 'Hello, world!' };
defineReactive(data, 'message', data.message);

data.message = 'Hello, Vue!'; // Set message: Hello, Vue!
console.log(data.message);    // Get message: Hello, Vue!

```

## <font style="color:rgb(51, 51, 51);">Proxy</font>
Proxy 是 ES6 新增功能，用于在目标对象与外界访问之间插入一层“代理”，以对外界的访问行为进行拦截、过滤或修改

Proxy 对象将行为自定义委托给所谓的 "handler" 对象，这个 handler 定义了目标对象的一些特定操作行为。下面是 Proxy 的基本构造函数：

```javascript
let proxy = new Proxy(target, handler);
```

+ target：被代理的目标对象，可以是任何对象（包括函数）
+ handler：用于定义拦截行为的对象，其属性函数用于定义对目标对象的操作方式

### 常用 handler
1. `get(target, property, receiver)` 获取属性值时触发
2. `set(target, property, value, receiver)` 设置属性值时触发
3. `has(target, property)`in 操作符被调用时触发
4. `deleteProperty(target, property)` delete 操作符被调用时触发
5. `apply(target, thisArg, argumentsList)` 代理函数调用时触发
6. `construct(target, argumentsList, newTarget)` 代理构造函数调用时触发

### 基本属性拦截
```javascript
let person = {
  name: "John Doe",
  age: 30
};

let handler = {
  get: (target, property) => {
    if (property === 'age' && target[property] < 18) {
      return 'Minor';
    }
    return target[property];
  },
  set: (target, property, value) => {
    if (property === 'age' && value < 0) {
      console.log("Age cannot be negative");
      return false;
    }
    target[property] = value;
    return true;
  }
};

let proxy = new Proxy(person, handler);

console.log(proxy.name); // Output: John Doe
console.log(proxy.age);  // Output: 30
proxy.age = -5;           // Output: Age cannot be negative
proxy.age = 17;
console.log(proxy.age);  // Output: Minor
```

### 函数调用拦截
```javascript
let targetFunction = function(msg) {
  console.log(msg);
};

let handler = {
  apply: (target, thisArg, argumentsList) => {
    if (argumentsList.length === 0) {
      console.log("No arguments provided");
    } else {
      return target.apply(thisArg, argumentsList);
    }
  }
};

let proxyFunction = new Proxy(targetFunction, handler);

proxyFunction("Hello, World!");     // Output: Hello, World!
proxyFunction();                    // Output: No arguments provided
```

### 构造函数拦截
```javascript
let targetConstructor = function(name) {
  this.name = name;
};

let handler = {
  construct: (target, args) => {
    console.log(`Creating a new instance with arguments: ${args}`);
    return new target(...args);
  }
};

let proxyConstructor = new Proxy(targetConstructor, handler);

let instance = new proxyConstructor("John Doe"); // Output: Creating a new instance with arguments: John Doe
console.log(instance.name);                       // Output: John Doe
```

### Reflect
Reflect 对象在 ES6 中被引入，提供了一组与 JavaScript 对象操作相关的静态方法。主要目的是将一些原本分散在 Object 上的内建方法集成到一个对象中，并且规范化这些操作，使其行为更加一致、可预测

在 Proxy 的 handler 中可以使用 Reflect 执行原本会发生的默认操作，这对于在代理对象时需要附带某些行为（如日志记录、数据验证等）但保持实际操作不变的情况非常有用

```javascript
let target = {name: "John"};
let handler = {
  get: (target, property, receiver) => {
    console.log(`Getting ${property}`);
    return Reflect.get(target, property, receiver);
  },
  set: (target, property, value, receiver) => {
    console.log(`Setting ${property} to ${value}`);
    return Reflect.set(target, property, value, receiver);
  }
};

let proxy = new Proxy(target, handler);

console.log(proxy.name); // 输出: Getting name，输出: John
proxy.name = "Jane";     // 输出: Setting name to Jane
console.log(proxy.name); // 输出: Getting name，输出: Jane
```

## Vue2 到 Vue3 响应式的改动
使用 Object.defineProperty 实现响应式有一定的局限性：

+ 需要对每个属性进行定义，不能直接监听整个对象
+ 无法动态监视属性的添加和删除
+ 对数组操作的补丁通过方法重写完成，使用起来有学习成本

因此 Vue2 官方文档提到涉及到对象和数组的改变时候的特殊处理方式

### 对象处理
Vue2 无法检测 property 的添加或移除。由于 Vue2 会在初始化实例时对 property 执行 getter/setter 转化，所以 property 必须在 `data` 对象上存在才能让 Vue 将它转换为响应式的

```javascript
var vm = new Vue({
  data:{
    a:1
  }
})

// `vm.a` 是响应式的

vm.b = 2
// `vm.b` 是非响应式的
```

对于已经创建的实例，Vue2 不允许动态添加根级别的响应式 property。但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式 property。

```javascript
Vue.set(vm.someObject, 'b', 2)

```

### 数组处理
Vue2 不能检测以下数组的变动：

1. 当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`
2. 当你修改数组的长度时，例如：`vm.items.length = newLength`

```javascript
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

以下代码可以实现和 `vm.items[indexOfItem] = newValue` 相同的效果，同时也将在响应式系统内触发状态更新：

```javascript
Vue.set(vm.items, indexOfItem, newValue)
vm.items.splice(indexOfItem, 1, newValue)
vm.$set(vm.items, indexOfItem, newValue)
vm.items.splice(newLength)
```

### Vue3 改进
因为 Proxy 对目标对象上的每一个操作都注入了一层拦截机制，Vue 3 使用 Proxy 和 Reflect 实现响应式系统，解决了 Vue 2 的一些局限性，并且支持更复杂和动态的对象操作，类似这样的伪代码

```javascript
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      // 检查当前是否有正在运行的副作用
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      // 查找到该属性的所有订阅副作用
      trigger(target, key)
    }
  })
}
```

+ 能够直接监听整个对象，包括属性的添加和删除
+ 数组索引和长度变化可以得到更好的支持
+ 更加灵活和强大的拦截系统，支持更多类型的操作

  
[Vue 官方文档 - Vue 中的响应式是如何工作的](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#how-reactivity-works-in-vue)

