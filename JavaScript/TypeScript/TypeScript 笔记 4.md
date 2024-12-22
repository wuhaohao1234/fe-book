## keyof
keyof 是一个单目运算符，接受一个对象类型作为参数，返回该对象类型的所有键名组成的联合类型。

```typescript
type MyObj = {
  foo: number,
  bar: string,
};

type Keys = keyof MyObj; // 'foo'|'bar'

type KeyT = keyof object;  // never
```



如果对象属性名采用索引形式，keyof 会返回属性名的索引类型。

```typescript
// 示例一
interface T {
  [prop: number]: number;
}

// number
type KeyT = keyof T;

// 示例二
interface T {
  // JavaScript 属性名为字符串时，包含了属性名为数值的情况
  // 因为数值属性名会自动转为字符串
  [prop: string]: number;
}

// string|number
type KeyT = keyof T;
```



对于联合类型，keyof 返回成员共有的键名，对于交叉类型，keyof 返回所有键名

```typescript
type A = { a: string; z: boolean };
type B = { b: string; z: boolean };

// 返回 'z'
type KeyT1 = keyof (A | B);

// 返回 'a' | 'x' | 'b' | 'y'
type KeyT2 = keyof (A & B);

// 相当于
keyof (A & B) ≡ keyof A | keyof B
```



keyof 取出的是键名组成的联合类型，如果想取出键值组成的联合类型，可以像下面这样写

```typescript
type MyObj = {
  foo: number,
  bar: string,
};

type Keys = keyof MyObj;

type Values = MyObj[Keys]; // number|string
```

:::info
`Keys`是键名组成的联合类型，而`MyObj[Keys]`会取出每个键名对应的键值类型，组成一个新的联合类型，即`number|string`

:::



keyof 运算符往往用于精确表达对象的属性类型

```typescript
function prop<T, K extends keyof T>(obj: T, key: K):T[K] {
  return obj[key];
}
```



keyof 的另一个用途是用于属性映射，即将一个类型的所有属性逐一映射成其它值

```typescript
type Writalbe<T> = {
  -readonly [Prop in keyof T]: T[Prop];
}

type MyObj = {
  readonly n: number;
}

type NewObj = Writalbe<MyObj>;
```

## in
在 JavaScript 中 in 用来判断对象中是否包含某属性，在 Typescript 中 in 用来取出（遍历）联合类型的每一个成员类型

```typescript
type U = 'a' | 'b' | 'c';

type Foo = {
  [Prop in U]: number;
};

// 等同于
type foo = {
  a: number;
  b: number;
  c: number;
};
```

## [] 运算符
[] 运算符用来取出对象的属性类型，T[K] 返回对象 T 的属性 K 的类型

```typescript
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

type AgeType = Person['age']; // number
type T = Person['age'|'name']; // number|string
```

[] 运算符的参数也可以是属性名的索引类型

```typescript
type Obj = {
  [key: string]: number;
};

type T = Obj[string]; // number

const MyArray = ['a', 'b', 'c']; // {[key: number]: string}
type T = MyArray[number]; // string
```

## T extends U ? X : Y
类似 JavaScript 的三元表达式，T extends U ? X : Y 根据 T 是否为 U 的子类型来返回不同的类型

```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

// number
type T1 = Dog extends Animal ? number : string;

// string
type T2 = RegExp extends Animal ? number : string;
```

如果需要判断的类型是一个联合类型，那么条件运算符会展开这个联合类型

```typescript
(A|B) extends U ? X : Y

// 等同于

(A extends U ? X : Y) |
(B extends U ? X : Y)
```

## infer
infer 关键字用来定义范型里面推断出来的类型参数，而不是外部传入的类型参数，通常和条件运算符一起使用，用在 extends 关键字后面的父类型之中

```typescript
type Flatten<Type> =
  Type extends Array<infer Item> ? Item : Type;

// 等同于

type Flatten<Type, Item> =
  Type extends Array<Item> ? Item : Type;
```

使用`infer`，推断函数的参数类型和返回值类型

```typescript
type ReturnPromise<T> = 
  T extends (...args: infer A) => infer R
  ? (...args: A) => Promise<R>
  : T;
```

如果`T`是函数，就返回这个函数的 Promise 版本，否则原样返回。`infer A`表示该函数的参数类型为`A`，`infer R`表示该函数的返回值类型为`R`。如果不使用`infer`，就不得不把`ReturnPromise<T>`写成`ReturnPromise<T, A, R>`，这样就很麻烦，相当于开发者必须人肉推断编译器可以完成的工作。

## is
函数返回布尔值的时候，可以使用`is`运算符，限定返回值与参数之间的关系。

```typescript
function isFish(pet: Fish|Bird):pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

`is`运算符还有一种特殊用法，就是用在类（class）的内部，描述类的方法的返回值。

```typescript
class Teacher {
  isStudent():this is Student {
    return false;
  }
}

class Student {
  isStudent():this is Student {
    return true;
  }
}
```

## 模版字符串
TypeScript 允许使用模板字符串构建类型，模板字符串的最大特点就是内部可以引用其他类型。模板字符串里面引用的类型，如果是一个联合类型，那么它返回的也是一个联合类型，即模板字符串可以展开联合类型。

```typescript
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

type T = 'A'|'B';
type U = `${T}_id`; // "A_id" | "B_id"
```

## 工具类
### Partial<T>
```typescript
type Partial<T> = {
  [P in keyof T]: T[P];
};
```

### Require<T>
```typescript
type Required<T> = {
  [P in keyof T]-?: T[P];
};
```

### Readonly<T>
```typescript
type Readonly<T> = {
  readonly [P in keyof P]: T[p];
};
```

### Pick<T, K>
```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P]; 
};
```

### Exclude<T, U>
`Exclude<T, U>` 从类型 `T` 中排除那些可以赋值给 `U` 的类型。

```typescript
type Exclude<T, U> = T extends U ? never : T;

type ResultType = Exclude<'a' | 'b' | 'c', 'a' | 'c'>; // 'b'
```

### Extract<T, U>
`Extract<T, U>` 提取那些可以赋值给 `U` 的类型。

```typescript
type Extract<T, U> = T extends U ? T : never;
```

### Omit<T, K>
```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

### Record<K, T>
`Record<K, T>` 构造一个以 `K` 中的键为键，值为 `T` 类型的对象类型。

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

### NonNullable<T>
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

### ReturnType<T>
```typescript
type ReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R
  ? infer R
  : any;
```

`ReturnType<T>` 通常用于类型推断和类型系统的自动化，这在构建复杂系统时尤为有用。例如，当你有一组函数，并希望确保某些操作基于这些函数的返回类型时，`ReturnType<T>` 可以帮你自动推断和处理这些类型。



