参考：[https://wangdoc.com/typescript](https://wangdoc.com/typescript/)

## 对象
属性名前面加上`readonly`关键字，表示这个属性是只读属性，只能在初始化阶段赋值，以后不能修改

```typescript
interface MyInterface {
  readonly prop: number;
}
```



如果希望属性值是只读的，除了声明时加上 readonly 关键字，还有一种方法，就是在赋值时，在对象后面加上只读断言 as const

```typescript
const myUser = {
  name: "Sabrina",
} as const;

myUser.name = "Cynthia"; // 报错
```



对象后面加了只读断言as const，就变成只读对象了，不能修改属性了

```typescript
const myUser = {
  name: "Sabrina",
} as const;

myUser.name = "Cynthia"; // 报错
```



解构赋值的类型写法，跟为对象声明类型是一样的

```typescript
const {id, name, price}:{
  id: string;
  name: string;
  price: number
} = product;

let { x: foo, y: bar }
  : { x: string; y: number } = obj;

```



只要对象 B 满足 对象 A 的结构特征，TypeScript 就认为对象 B 兼容对象 A 的类型，这称为“结构类型”原则（structural typing）。根据“结构类型”原则，TypeScript 检查某个值是否符合指定类型时，并不是检查这个值的类型名（即“名义类型”），而是检查这个值的结构是否符合要求（即“结构类型”）。

如果类型 B 可以赋值给类型 A，TypeScript 就认为 B 是 A 的子类型（subtyping），A 是 B 的父类型。子类型满足父类型的所有结构特征，同时还具有自己的特征。凡是可以使用父类型的地方，都可以使用子类型，即子类型兼容父类型。

```typescript
type A = {
  x: number;
};

type B = {
  x: number;
  y: number;
};
```



空对象是 TypeScript 的一种特殊值，也是一种特殊类型。

```typescript
const obj = {};
obj.prop = 123; // 报错
```

上面示例中，变量 obj 的值是一个空对象，然后对 obj.prop 赋值就会报错。

原因是这时 TypeScript 会推断变量 obj 的类型为空对象，实际执行的是下面的代码。

```typescript
const obj:{} = {};
```

空对象没有自定义属性，所以对自定义属性赋值就会报错。空对象只能使用继承的属性，即继承自原型对象Object.prototype的属性。如果想强制使用没有任何属性的对象，可以采用下面的写法

```typescript
interface WithoutProperties {
  [key: string]: never;
}

// 报错
const a:WithoutProperties = { prop: 1 };
```

## 接口
interface 可以表示对象的各种语法，它的成员有5种形式。

+ 对象属性
+ 对象的属性索引
+ 对象方法
+ 函数
+ 构造函数

interface 可以继承 type 命令定义的对象类型，interface 还可以继承 class，即继承该类的所有成员



多个同名接口会合并成一个接口，这样的设计主要是为了兼容 JavaScript 的行为。JavaScript 开发者常常对全局对象或者外部库，添加自己的属性和方法。那么，只要使用 interface 给出这些自定义属性和方法的类型，就能自动跟原始的 interface 合并，使得扩展外部类型非常方便。

```typescript
interface Box {
  height: number;
  width: number;
}

interface Box {
  length: number;
}
```

  
interface 与 type 的区别有下面几点：

+ type能够表示非对象类型，而interface只能表示对象类型（包括数组、函数等）
+ interface可以继承其他类型，type不支持继承，使用 & 运算符
+ 同名 interface 会自动合并，同名 type 则会报错
+ interface 不能包含属性映射（mapping），type 可以
+ this 关键字只能用于 interface
+ type 可以扩展原始数据类型，interface 不行

```typescript
// 正确
type MyStr = string & {
  type: 'new'
};

// 报错
interface MyStr extends string {
  type: 'new'
}
```

+ interface 无法表达某些复杂类型（比如交叉类型和联合类型），但是 type 可以

```typescript
type A = { /* ... */ };
type B = { /* ... */ };

type AorB = A | B;
type AorBwithName = AorB & {
  name: string
};
```

## 类
interface 接口或 type 别名，可以用对象的形式，为 class 指定一组检查条件。然后，类使用 implements 关键字，表示当前类满足这些外部类型条件的限制。

```typescript
interface Country {
  name:string;
  capital:string;
}
// 或者
type Country = {
  name:string;
  capital:string;
}

class MyCountry implements Country {
  name = '';
  capital = '';
}
```



TypeScript 不允许两个同名的类，但是如果一个类和一个接口同名，那么接口会被合并进类

```typescript
class A {
  x:number = 1;
}

interface A {
  y:number;
}

let a = new A();
a.y = 10;

a.x // 1
a.y // 10
```



TypeScript 的类本身就是一种类型，但是它代表该类的实例类型，而不是 class 的自身类型，要获得一个类的自身类型，一个简便的方法就是使用 typeof 运算符

```typescript
function createPoint(
  PointClass:typeof Point,
  x:number,
  y:number
):Point {
  return new PointClass(x, y);
}

```

JavaScript 语言中，类只是构造函数的一种语法糖，本质上是构造函数的另一种写法。所以，类的自身类型可以写成构造函数的形式。

```typescript
function createPoint(
  PointClass: new (x:number, y:number) => Point,
  x: number,
  y: number
):Point {
  return new PointClass(x, y);
}

// 构造函数也可以写成对象的形式
function createPoint(
  PointClass: {
    new (x:number, y:number): Point
  },
  x: number,
  y: number
):Point {
  return new PointClass(x, y);
}
```



对于引用实例对象的变量来说，既可以声明类型为 Class，也可以声明类型为 Interface，因为两者都代表实例对象的类型

```typescript
interface MotorVehicle {
}

class Car implements MotorVehicle {
}

// 写法一
const c1:Car = new Car();
// 写法二
const c2:MotorVehicle = new Car();

```

## 泛型
函数泛型声明

```typescript
function fn<T>(arg: T):T {
  
}

let fn1:<T>(arg:T) => T = fn;
let fn2:{ <T>(arg:T): T } = fn;

```



type 命令定义的类型别名，也可以使用泛型。

```typescript
type Nullable<T> = T | undefined | null;
```

  
下面是定义树形结构的例子,一旦类型参数有默认值，就表示它是可选参数。如果有多个类型参数，可选参数必须在必选参数之后

```typescript
type Tree<T = number> = {
  value: T;
  left: Tree<T> | null;
  right: Tree<T> | null;
};
```



很多泛型类型参数并不是无限制的，对于传入的类型存在约束条件。TypeScript 允许在类型参数上面写明约束条件，如果不满足条件，编译时就会报错

```typescript
function compare<T extends { length: number }>(a: T, b:T): T {
  if (a.length >= b.length) {
    return a;
  }
  return b;
}
```

## Enum
Enum 结构比较适合的场景是，成员的值不重要，名字更重要，从而增加代码的可读性和可维护性。

Enum 结构的特别之处在于，它既是一种类型，也是一个值。绝大多数 TypeScript 语法都是类型语法，编译后会全部去除，但是 Enum 结构是一个值，编译后会变成 JavaScript 对象，留在代码中

```typescript
// 编译前
enum Color {
  Red,     // 0
  Green,   // 1
  Blue     // 2
}

// 编译后
let Color = {
  Red: 0,
  Green: 1,
  Blue: 2
};
```



Enum 成员值都是只读的，不能重新赋值。为了让这一点更醒目，通常会在 enum 关键字前面加上 const 修饰，表示这是常量，不能再次赋值。很大程度上，Enum 结构可以被对象的 as cons t断言替代

```typescript
const enum Foo {
  A,
  B,
  C,
}

const Bar = {
  A: 0,
  B: 1,
  C: 2,
} as const;

if (x === Foo.A) {}
// 等同于
if (x === Bar.A) {}
```



Enum 加上 const 还有一个好处，就是编译为 JavaScript 代码后，代码中 Enum 成员会被替换成对应的值，这样能提高性能表现

```typescript
const enum Color {
  Red,
  Green,
  Blue
}

const x = Color.Red;
const y = Color.Green;
const z = Color.Blue;

// 编译后
const x = 0 /* Color.Red */;
const y = 1 /* Color.Green */;
const z = 2 /* Color.Blue */;
```

  
多个同名的 Enum 结构会自动合并，结构合并时，只允许其中一个的首成员省略初始值，否则报错

```typescript
enum Foo {
  A,
}

enum Foo {
  B = 1,
}

enum Foo {
  C = 2,
}

// 等同于
enum Foo {
  A,
  B = 1，
  C = 2
}
```

  
keyof 运算符可以取出 Enum 结构的所有成员名，作为联合类型返回

```typescript
enum MyEnum {
  A = 'yes',
  B = 'no'
}

// 'A'|'B'
type Foo = keyof typeof MyEnum;
```

如果要返回 Enum 所有的成员值，可以使用 in 运算符

```typescript
enum MyEnum {
  A = 'a',
  B = 'b'
}

// { a: any, b: any }
type Foo = { [key in MyEnum]: any };

```

## 断言
类型断言并不是真的改变一个值的类型，而是提示编译器，应该如何处理这个值

```typescript
// 语法一：<类型>值
<Type>value

// 语法二：值 as 类型
value as Type
```



如果没有声明变量类型，let 命令声明的变量，会被类型推断为 TypeScript 内置的基本类型之一；const 命令声明的变量，则被推断为值类型常量。

```typescript
// 类型推断为基本类型 string
let s1 = 'JavaScript';

// 类型推断为字符串 “JavaScript”
const s2 = 'JavaScript';
```

  
`as const`断言可以用于整个对象，也可以用于对象的单个属性，这时它的类型缩小效果是不一样的。`as const`会将字面量的类型断言为不可变类型，缩小成 TypeScript 允许的最小类型。

```typescript
const v1 = {
  x: 1,
  y: 2,
}; // 类型是 { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
}; // 类型是 { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const; // 类型是 { readonly x: 1; readonly y: 2; }
```



