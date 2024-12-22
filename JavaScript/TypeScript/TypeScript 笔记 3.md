## import export
import 在一条语句中，可以同时输入类型和正常接口，这样很不利于区分类型和正常接口，容易造成混淆。为了解决这个问题，TypeScript 引入了两个解决方法。

```typescript
// a.ts
export interface A {
  foo: string;
}

export let a = 123;

// b.ts
import { A, a } from './a';

import { type A, a } from './a';
import type { A } from './a';
```

同样的，export 语句也有两种方法，表示输出的是类型。

```typescript
type A = 'a';
type B = 'b';

// 方法一
export {type A, type B};

// 方法二
export type {A, B};
```



CommonJS 是 Node.js 的专用模块格式，与 ES 模块格式不兼容，TypeScript 有两种 import 方式

```typescript
import * as fs from 'fs';
import fs = require('fs');
```

## namesapce
namespace 是一种将相关代码组织在一起的方式，它出现在 ES 模块诞生之前，作为 TypeScript 自己的模块格式而发明的。但自从有了 ES 模块，官方已经不推荐使用 namespace 了。

namespace 本身也可以使用`export`命令输出，供其他文件使用。

```typescript
// shapes.ts
export namespace Shapes {
  export class Triangle {
    // ...
  }
  export class Square {
    // ...
  }
}

// 写法一
import { Shapes } from './shapes';
let t = new Shapes.Triangle();

// 写法二
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle();
```

  
多个同名的 namespace 会自动合并，这一点跟 interface 一样。合并命名空间时，命名空间中的非`export`的成员不会被合并，但是它们只能在各自的命名空间中使用。

```typescript
namespace Animals {
  export class Cat {}
}
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Dog {}
}

// 等同于
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Cat {}
  export class Dog {}
}
```

## declare
declare 关键字用来告诉编译器，某个类型是存在的，当前文件可以使用其他文件声明的类型。declare 关键字的重要特点是，它只是通知编译器某个类型是存在的，不用给出具体实现。比如，只描述函数的类型，不给出函数的实现，如果不使用`declare`这是做不到的。

declare 关键字可以给出外部函数的类型描述。

```typescript
declare function sayHello(name:string):void;
```

  
如果想把变量、函数、类组织在一起，可以将 declare 与 module 或 namespace 一起使用。如果类型声明文件名为`index.d.ts`，且在项目的根目录中，那就不需要在`package.json`里面注明了

```typescript
declare namespace AnimalLib {
  class Animal {
    constructor(name:string);
    eat():void;
    sleep():void;
  }

  type Animals = 'Fish' | 'Dog';
}

// 或者 declare 一个 package
declare module AnimalLib {
  class Animal {
    constructor(name:string);
    eat(): void;
    sleep(): void;
  }

  type Animals = 'Fish' | 'Dog';
}
```



declare module 和 declare namespace 里面，加不加 export 关键字都可以。

```typescript
declare namespace Foo {
  export var a: boolean;
}

declare module 'io' {
  export function readFile(filename:string):string;
}

```



如果要为 JavaScript 引擎的原生对象添加属性和方法，可以使用`declare global {}`语法。

```typescript
export {};

declare global {
  interface String {
    toSmallString(): string;
  }
}

String.prototype.toSmallString = ():string => {
  // 具体实现
  return '';
};

declare global {
  interface Window {
    myAppConfig:object;
  }
}
```



类型声明文件里面，变量的类型描述必须使用`declare`命令，否则会报错，因为变量声明语句是值相关代码。

```typescript
declare let foo:string;
```

  
interface 类型有没有`declare`都可以，因为 interface 是完全的类型代码。

```typescript
interface Foo {} // 正确
declare interface Foo {} // 正确
```

## 类型文件
当前模块如果包含自己的类型声明文件，可以在 package.json 文件里面添加一个types字段或typings字段，指明类型声明文件的位置。

```typescript
{
  "name": "awesome",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts"
}
```



如果类型声明文件的内容非常多，可以拆分成多个文件，然后入口文件使用三斜杠命令，加载其他拆分后的文件。入口文件是`main.d.ts`，里面的接口定义在`interfaces.d.ts`，函数定义在`functions.d.ts`。那么，`main.d.ts`里面可以用三斜杠命令，加载后面两个文件。

```typescript
/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />
```

:::info
三斜杠命令（`///`）是一个 TypeScript 编译器命令，用来指定编译器行为。它只能用在文件的头部，如果用在其他地方，会被当作普通的注释。

:::

## 注释指令
`// @ts-nocheck`告诉编译器不对当前脚本进行类型检查，可以用于 TypeScript 脚本，也可以用于 JavaScript 脚本。

```typescript
// @ts-nocheck

const element = document.getElementById(123);
```



`// @ts-ignore`告诉编译器不对下一行代码进行类型检查，可以用于 TypeScript 脚本，也可以用于 JavaScript 脚本。

```typescript
let x:number;

x = 0;

// @ts-ignore
x = false; // 不报错
```

## JSDoc
TypeScript 直接处理 JS 文件时，如果无法推断出类型，会使用 JS 脚本里面的 JSDoc 注释。

```typescript
/**
 * @param {string} [x="bar"]
 */
function foo(x) {}

/**
 * @return {boolean}
 */
function foo() {
  return true;
}
```

  


