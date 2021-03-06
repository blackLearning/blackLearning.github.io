# TypeScirpt

## 类型系统(type system)

- 结构类型(structural typing)
- 命名类型(nominal typing)

当比较两种数据的时候, 如果比较的是他们的类型结构, 则是结构类型, 也就是说即使两个参数的类型不同，但是如果他们的类型结构是一样，我们就认为它们的类型是兼容的


比如 Java, C++, Swift则主要是以命名类型为主:

```java
class Foo {
  method(input: string): number { ... }
}
class Bar {
  method(input: string): number { ... }
}
let foo: Foo = new Bar(); // ERROR!!
```

比如 ts, Haskell, Elm则主要是以结构类型为主:

```ts
class Foo {
  
method(input: string): number { ... }
}
class Bar {
  
method(input: string): number { ... }
}
let foo: Foo = new Bar(); // Okay.
```

## 基础类型:

-   Boolean
-   Number
-   String
-   symbol
-   Array
-   Object: 表示所有的非原始类型，也就是引用类型。
-   Null
-   Undefined

    默认Null和undefined可以复制给任意的其他类型。

    可以通过`strictNullChecks`来限制这种行为，让null和undefined只能复制给他们自身和void类型。

-   Tuple: 可以知道数组内不同元素的类型的数组.
-   Enum: 定义一组数值
-   Any: 任意类型，主要用于兼容第三方的代码,

    可以使用`noImplitAny`来限制any类型的使用。

-   Void: 没有任何类型，例如函数没有返回值
-   Never: 表示永远不存在的类型，是任意类型的子类型，也可以赋值给任意类型。但是其他所有的类型都不能赋值给never类型。

## Interfaces

接口定义了一些类型的成员和对应的类型，它是只在编译阶段存在，不能包含成员的具体实现细节。

所有的相同的 Interface 的声明都会最终合并在一起，因此如果你想在一个已经声明了的 Interface 上继续添加属性的话，可以继续声明该 Interface 即可。

-   optional properties

`x?: Number;`

-   readonly properties: 属性使用 readonly, 变量使用`const`

`readyonly x: Number;`

只读数组，任何调用数组上可变方法(push, pop)等方法都会导致编译错误。
`let ro: ReadonlyArray<number> = a;`

-   function types

表示一个可被调用的类型注解

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

-   class Types, implements, extends


当声明一个class的同时，也相当于声明了一个拥有类的成员的同名的一个类类型。

与C#或Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。

```ts
interface ClockInterface {
  currentTime: Date;
  setTime(time :Date) :void;
}

class Clock implements ClockInterface {
  private currentTime: Date;
  readonly time: Date;
  constructor(h: number, m: number) {
    this.currentTime = new Date();
  }
  public setTime() {}
}
```

除了类可以互相继承之外，接口也可以相互继承。 这让我们能够从一个接口里复制成员到另一个接口里，可以更灵活地将接口分割到可重用的模块里。

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

// 多重继承，接口也可以继承类类型
interface Square extends Shape, PenStroke, Clock {
    sideLength: number;
}

// 类型断言
let square = <Square>{};
let square = {} as Square;
square.color = 'blue';
square.sideLength = 10;
square.penWidth = 5.0;
```

## Namespaces and Modules

namespaces是一种过时的ts模块化方案。

```ts
namespace foo {
    export var x = 10;
    export var y = 20;
}

// =>

var foo;
(function (foo) {
    foo.x = 10;
    foo.y = 20;
})(foo || (foo = {}));

```

但namespaces可以用于ts的代码组织，例如一些顶层API比如react, 但是有大量的类型和方法，可以用namespace将其包含在一个顶级的命名空间中，而不是大量的export，常常用于一些顶级变量例如jquery, react, angular等.d.ts声明文件的编写。

相同名字的namespace会互相合并，并且namespace是全局作用域的。

例如： 

```ts
export = React;
export as namespace React;
// 用于声明react 类型的.d.ts，使用namespace来组织代码
declare namespace React {
    type ReactText = string | number;
    type ReactChild = ReactElement | ReactText;

    interface ReactNodeArray extends Array<ReactNode> {}
    type ReactFragment = {} | ReactNodeArray;
    type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;

    //
    // Top Level API
    // ----------------------------------------------------------------------

    // DOM Elements
    function createFactory<T extends HTMLElement>(
        type: keyof ReactHTML): HTMLFactory<T>;
}

```

modules 也就是es2015的模块方案，使用export 和 import 来导入导出模块。

declare 用于声明已经实现了的函数或者模块的类型。

```ts
// 声明一个已经存在的buffer模块的方法类型
declare module "buffer" {
    export const INSPECT_MAX_BYTES: number;
    const BuffType: typeof Buffer;

    export type TranscodeEncoding = "ascii" | "utf8" | "utf16le" | "ucs2" | "latin1" | "binary";

    export function transcode(source: Buffer | Uint8Array, fromEnc: TranscodeEncoding, toEnc: TranscodeEncoding): Buffer;

    export const SlowBuffer: {
        /** @deprecated since v6.0.0, use Buffer.allocUnsafeSlow() */
        new(size: number): Buffer;
        prototype: Buffer;
    };

    export { BuffType as Buffer };
}

```

### 模块解析

遵循NodeJs的模块解析策略

```ts
// root/src/moduleA.ts
import { b } from "./moduleB"
```

- /root/src/moduleB.ts
- /root/src/moduleB.tsx
- /root/src/moduleB.d.ts
- /root/src/moduleB/package.json (如果指定了"types"属性)
- /root/src/moduleB/index.ts
- /root/src/moduleB/index.tsx
- /root/src/moduleB/index.d.ts

### 三斜线指令

`/// <reference path="..." />`指令是三斜线指令中最常见的一种。 
它用于声明文件间的依赖。
告诉编译器在编译过程中要引入的额外的文件。

大多数常用的第三方模块都可以在`DefinitelyTyped`这个库中找到对应的声明文件。

默认在编译时会将所有的node_modules中的@types文件下的声明文件引入进来。
可以通过`typeRoots`和`types`配置项来改变默认行为。

自动引入只在你使用了全局的声明（相反于模块）时是重要的。 如果你使用 import "foo"语句，TypeScript仍然会查找node_modules和node_modules/@types文件夹来获取foo包。

## Generics

泛型是在定义一个类型或者接口的时候，我们不知道其中某些参数的具体类型，但是又想对其进行某种约束，这时候就可以用泛型(常常是一个大写字母)来代替表示该类型。

当你使用简单的泛型时，泛型常用 T、U、V 表示。如果在你的参数里，不止拥有一个泛型，你应该使用一个更语义化名称，如 TKey 和 TValue （通常情况下，以 T 做为泛型前缀也在如 C++ 的其他语言里做为模版。）

-   泛型函数

```ts
function identity(arg: T): T {
    return arg;
}

// 可选：把类型作为参数，从而指定该函数的参数和返回值都必须是该类型。
function identity<T>(arg: T): T {
    return arg;
}

// 传入string作为类型，从而指定该函数的参数和返回值都必须是字符串。
let output = identity<string>('myString');
```

-   泛型 interface

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

// 把类型参数提到interface层面可以当参数传入
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

有时在使用泛型参数的时候，也可以省略该参数，让ts为我们自动推导出T的类型。

泛型参数同样可以有默认值。

```ts
interface GenericIdentityFn<T = string> {
    (arg: T): T;
}
```

泛型约束：也可以规定该泛型必须具有某些类型, 这里可以规定T必须是number或者string类型。

```ts
function identity<T extends number | string>(arg: T): T {
    return arg;
}
```

-   泛型类

泛型类和泛型 interface 类似：

```js
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

为了让泛型可以工作在一些特殊的情况，引入泛型约束。

```js
// 约束该泛型必须有length属性
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  // 这样在使用length属性的时候，ts就不会提示错误
  // 否则ts就会由于不知T是什么类型，从而提示length属性错误
  console.log(arg.length);
  return arg;
}
```

## 类型进阶

### 类型推断

一些编程语言要求变量声明的时候必须指定它的类型, 比如C和java, 一些编程语言可以在变量声明的时候自动推断变量的类型, 例如haskell, typescript.

- 定义时推断

```ts
let foo = 123
let bar = 'Hello'
foo = bar // Error: cannot assign `string` to a `number`
```

- 返回值推断

```ts
// 返回值推断
function add(a: number, b: number) {
    return a + b
}
let foo = add(1, 2) // foo: number
```

- 条件推断

```ts
function getString(a :string | number) :string {
  if (typeof a === 'number') {
    return String(a) // a: number
  }
  return a
}
```

### 类型断言：

```ts
let pet = getSmallPet();
// 当不能确定一个变量的类型，需要对变量类型进行断言，这样ts才不会抛出错误
if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
} else {
    (<Bird>pet).fly();
}
```

然而，当你在 JSX 中使用 <Fish> 的断言语法时，这会与 JSX 的语法存在歧义：

因此可以使用`(pet as Fly).fly()`的 as 语法来断言。

###  类型守护：

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(' ') + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

### 类型别名

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    } else {
        return n();
    }
}
```

## 类型操作符和类型工具：

- readonly, ?

- extends 和条件类型

```ts
T extends U ? X : Y
```

如果 T 类型可以认为是继承于U类型（要么 T 和 U 是同一种基础类型，要么 T 类型的代表范围 小于 U类型，也就是 T 是 U 的子集，U 是 T 的超集），则取 X, 否则取 Y;

- typeof: 从变量中读出其类型(通常由 ts 推断得出)

这允许你告诉编译器，一个变量的类型与其他类型相同

```ts
let foo = 123;
let bar: typeof foo; // 'bar' 类型与 'foo' 类型相同（在这里是： 'number'）

bar = 456; // ok
bar = '789'; // Error: 'string' 不能分配给 'number' 类型
```

- keyof: 索引类型查询操作符，对于任何类型 T， keyof T的结果为 T上已知的公共属性名的联合类型

keyof 操作符能让你捕获一个类型的键。例如，你可以使用它来捕获变量的键名称，在通过使用 typeof 来获取类型之后：

```ts
const colors = {
    red: 'red',
    blue: 'blue'
};

type Colors = keyof typeof colors;

let color: Colors; // color 的类型是 'red' | 'blue'
```

- T[K]: 索引访问操作符 & in: 映射类型

```ts
interface colors {
  red: string,
  blue: number
};

// in可以将T中所有的属性转为新的类型中的属性
// T[k] 则可以取到原类型中的某个属性的类型
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}

type A = Map<colors>
```
-   as: 类型推断(当你比 ts 编译器更懂该变量的类型时，可以强制指定该变量的类型，防止编译报错)


--- 
- ConstructorParameters: a tuple of class constructor's parameter types
- Exclude: exclude a type from another type
- Extract: select a subtype that is assignable to another type
- InstanceType: the instance type you get from a newing a class constructor
- NonNullable: exclude null and undefined from a type
- Parameters: a tuple of a function's parameter types
- Partial: Make all properties in an object optional
- Readonly: Make all properties in an object readonly
- ReadonlyArray: Make an immutable array of the given type
- Pick: A subtype of an object type with a subset of its keys
- Record: A map from a key type to a value type
- Required: Make all properties in an object required
- ReturnType A function's return type


更多工具类型参考：[utility-types](https://github.com/piotrwitek/utility-types)

## .d.ts

> 为了描述不是用TypeScript编写的类库的类型，我们需要声明类库导出的API。 由于大部分程序库只提供少数的顶级对象，命名空间是用来表示它们的一个好办法。

> 我们称其为声明是因为它不是外部程序的具体实现。 我们通常在 .d.ts里写这些声明。 如果你熟悉C/C++，你可以把它们当做 .h文件。

一个典型的module类型库的声明文件模板

```ts

export as namespace myLib;
// d3.d.ts
/*~ If this module has methods, declare them as functions like so.
 */
export function myMethod(a: string): string;
export function myOtherMethod(a: number): number;

/*~ You can declare types that are available via importing the module */
export interface someType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ You can declare properties of the module using const, let, or var */
export const myField: number;

/*~ If there are types, properties, or methods inside dotted names
 *~ of the module, declare them inside a 'namespace'.
 */
export namespace subProp {
    /*~ For example, given this definition, someone could write:
     *~   import { subProp } from 'yourModule';
     *~   subProp.foo();
     *~ or
     *~   import * as yourMod from 'yourModule';
     *~   yourMod.subProp.foo();
     */
    export function foo(): void;
}
```

快速忽略第三方库的类型声明:

```ts
// my-typings.ts
declare module "react"; // each of its imports are `any`
```

扩展第三方的类型声明： 

```ts
// react.d.ts
import React from 'react';

declare module "React" {
  interface HTMLAttributes<T> {
    hideFocus?: boolean;
  }
}
```


## React + TS

@types/react, @types/react-dom


基本的Props类型模板:

使用标准的Ts注释方式: `/** comment */`, 方便使用者清楚的知道组件的属性含义.

```ts
type AppProps = {
  message: string;
  count: number;
  disabled: boolean;
  /** array of a type! */
  names: string[];
  /** string literals to specify exact string values, with a union type to join them together */
  status: "waiting" | "success";
  /** any object as long as you dont use its properties (not common) */
  obj: object;
  obj2: {}; // almost the same as `object`, exactly the same as `Object`
  /** an object with defined properties (preferred) */
  obj3: {
    id: string;
    title: string;
  };
  /** array of objects! (common) */
  objArr: {
    id: string;
    title: string;
  }[];
  /** any function as long as you don't invoke it (not recommended) */
  onSomething: Function;
  /** function that doesn't take or return anything (VERY COMMON) */
  onClick: () => void;
  /** function with named prop (VERY COMMON) */
  onChange: (id: number) => void;
  /** alternative function type syntax that takes an event (VERY COMMON) */
  onClick(event: React.MouseEvent<HTMLButtonElement>): void;
  /** an optional prop (VERY COMMON!) */
  optional?: OptionalType;
};
```

---

#### `React.FC<Props>` | `React.FunctionComponent<Props>`

```tsx
const MyComponent: React.FC<Props> = ...
```

#### `React.Component<Props, State>`

```tsx
class MyComponent extends React.Component<Props, State> { ...
```

1. 将props, state设为read-only
2. 为组件的props做注解, 对使用者友好, 因此可以代替prop-types

#### `React.ComponentType<Props>`

```tsx
const withState = <P extends WrappedComponentProps>(
  WrappedComponent: React.ComponentType<P>,
) => { ...
```

#### `React.ComponentProps<typeof XXX>` | `React.ComponentProps<htmlComponent>`

```tsx
type MyComponentProps = React.ComponentProps<typeof MyComponent>;
```

#### `React.ReactElement` | `JSX.Element`

```tsx
const elementOnly: React.ReactElement = <div /> || <MyComponent />;
```

#### `React.ReactNode`

```tsx
const elementOrPrimitive: React.ReactNode = 'string' || 0 || false || null || undefined || <div /> || <MyComponent />;
const Component = ({ children: React.ReactNode }) => ...
```

#### `React.CSSProperties`

```tsx
const styles: React.CSSProperties = { flexDirection: 'row', ...
const element = <div style={styles} ...
```

#### `React.HTMLProps<HTMLXXXElement>`

```tsx
const Input: React.FC<Props & React.HTMLProps<HTMLInputElement>> = props => { ... }

<Input about={...} accept={...} alt={...} ... />
```

#### `React.ReactEventHandler<HTMLXXXElement>`

```tsx
const handleChange: React.ReactEventHandler<HTMLInputElement> = (ev) => { ... } 

<input onChange={handleChange} ... />
```

#### `React.XXXEvent<HTMLXXXElement>`

更加细粒度的事件类型, 比如 `ChangeEvent, FormEvent, FocusEvent, KeyboardEvent, MouseEvent, DragEvent, PointerEvent, WheelEvent, TouchEvent`.

```tsx
const handleChange = (ev: React.MouseEvent<HTMLDivElement>) => { ... }

<div onMouseMove={handleChange} ... />
```


#### defaultProps和props类型推断冲突

- Non-null assertion operator(非空断言语句)
- Component type casting(组件类型重置)
- High order function for defining defaultProps(高阶组件)
- Props getter function(Getter函数)

refrence: [react-typescript-and-defaultprops-dilemma](https://medium.com/@martin_hotell/react-typescript-and-defaultprops-dilemma-ca7f81c661c7)

## Refrence

[React & Redux in TypeScript - Static Typing Guide](https://github.com/piotrwitek/react-redux-typescript-guide)
