参考：[https://wangdoc.com/typescript](https://wangdoc.com/typescript/)

+ <font style="color:rgb(74, 74, 74);">TypeScript 规定，变量只有赋值后才能使用，否则就会报错。</font>
+ <font style="color:rgb(74, 74, 74);">TypeScript 的类型检查只是编译时的类型检查，而不是运行时的类型检查</font>
+ <font style="color:rgb(74, 74, 74);">“类型”是针对“值”的，可以视为是后者的一个元属性。每一个值在 TypeScript 里面都是有类型的。	</font>
+ <font style="color:rgb(74, 74, 74);">TypeScript 代码只涉及类型，不涉及值。所有跟“值”相关的处理，都由 JavaScript 完成。</font>
+ <font style="color:rgb(74, 74, 74);">TypeScript 项目里面，其实存在两种代码，一种是底层的“值代码”，另一种是上层的“类型代码”。前者使用 JavaScript 语法，后者使用 TypeScript 的类型语法。</font>
+ <font style="color:rgb(74, 74, 74);">never类型的一个重要特点是，可以赋值给任意其他类型。空集是任何集合的子集。</font>
+ TypeScript 有两个“顶层类型”（any和unknown），但是“底层类型”只有never
+ 所有类型的名称都是小写字母，首字母大写的Number、String、Boolean等在 JavaScript 语言中都是内置对象，而不是类型名称
+ 除了undefined和null这两个值不能转为对象，其他任何值都可以赋值给Object类型
+ 空对象{}是Object类型的简写形式，所以使用Object时常常用空对象代替
+ 小写的object类型代表 JavaScript 里面的狭义对象，即可以用字面量表示的对象，只包含对象、数组和函数，不包括原始类型的值
+ <font style="color:rgb(74, 74, 74);">任何其他类型的变量都可以赋值为undefined或null</font>
+ <font style="color:rgb(74, 74, 74);">TypeScript 规定，单个值也是一种类型，称为“值类型”。</font>
+ <font style="color:rgb(74, 74, 74);">由于编译时不会进行 JavaScript 的值运算，所以TypeScript 规定，typeof 的参数只能是标识符，不能是需要运算的表达式。</font>
+ <font style="color:rgb(74, 74, 74);">TypeScript 允许使用方括号读取数组成员的类型 </font>

```typescript
type Names = string[];
type Name = Names[0]; // string
```

+ <font style="color:rgb(74, 74, 74);">TypeScript 允许声明只读数组，方法是在数组类型前面加上readonly关键字。 </font>
+ <font style="color:rgb(74, 74, 74);">由于只读数组是数组的父类型，所以它不能代替数组。这一点很容易产生令人困惑的报错。</font>
+ <font style="color:rgb(74, 74, 74);">数组的成员类型写在方括号外面（number[]），元组的成员类型是写在方括号里面（[number]）</font>

```typescript
// 数组
let a:number[] = [1];

// 元组
let t:[number] = [1];
```

+ <font style="color:rgb(74, 74, 74);">元组成员的类型可以添加问号后缀（</font><font style="color:rgb(218, 16, 57);background-color:rgb(245, 245, 245);">?</font><font style="color:rgb(74, 74, 74);">），表示该成员是可选的。问号只能用于元组的尾部成员，也就是说，所有可选成员必须在必选成员之后</font>
+ <font style="color:rgb(74, 74, 74);">扩展运算符（...）用在元组的任意位置都可以，它的后面只能是一个数组或元组</font>

```typescript
type t1 = [string, number, ...boolean[]];
type t2 = [string, ...boolean[], number];
type t3 = [...boolean[], string, number];
```

+ 函数类型中参数名是必需的，但实际函数实现参数名可以不一致

```typescript
type MyFunc = (string, number) => number; // X
type MyFunc = (x: string, y: number) => number; // Y
```

+ <font style="color:rgb(74, 74, 74);">TypeScript 允许函数传入的参数不足</font>
+ 任何需要类型的地方，都可以使用 typeof 运算符从一个值获取类型
+ <font style="color:rgb(74, 74, 74);">function rest 参数类型声明</font>

```typescript
// rest 参数为数组
function joinNumbers(...nums:number[]) {
  // ...
}

// rest 参数为元组
function f(...args:[boolean, number]) {
  // ...
}

function multiply(n:number, ...m:number[]) {
  return m.map((x) => n * x);
}

function repeat(...[str, times]: [string, number]):string {
  return str.repeat(times);
}
```

+ void 类型表示函数没有返回值，void 类型允许返回 undefined 或 null，如果打开了`strictNullChecks`编译选项，那么 void 类型只允许返回 undefined
+ 函数**参数**声明返回值是 void，实际使用可以是返回任意值的函数，只要不使用返回值即可
+ 函数的运行结果如果是抛出错误，也允许将返回值写成 void（大部分时候使用 never）
+ never 表示函数不可能有返回值
+ never 是 TypeScript 的唯一一个底层类型，所有其他类型都包括了never。从集合论的角度看，number|never 等同于 number
+ TypeScript 对于“函数重载”的类型声明方法是，逐一定义每一种情况的类型。为了降低复杂性，一般来说，如果可以的话，应该优先使用联合类型替代函数重载

```typescript
// 写法一
function len(s:string):number;
function len(arr:any[]):number;
function len(x:any):number {
  return x.length;
}

// 写法二
function len(x:any[]|string):number {
  return x.length;
}
```

+ <font style="color:rgb(74, 74, 74);">构造函数的类型写法，就是在参数列表前面加上</font><font style="color:rgb(218, 16, 57);background-color:rgb(245, 245, 245);">new</font><font style="color:rgb(74, 74, 74);">命令</font>

```typescript
class Animal {
  numLegs: number = 4;
}
type AnimalConstructor = new () => Animal;

type F = {
  new (s: string): object;
};
```

