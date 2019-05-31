# TypeScirpt 小记

### 基础类型:

-   Boolean
-   Number
-   String
-   symbol
-   Array
-   Object
-   Null
-   Undefined
-   Tuple: 可以知道数组内不同元素的类型的数组.
-   Enum
-   Any
-   Void
-   Never

### Interfaces

所有的相同的 Interface 的声明都会最终合并在一起，因此如果你想在一个已经声明了的 Interface 上继续添加属性的话，可以继续声明该 Interface 即可。

ts的类型是基于结构的类型，也就是说即使两个参数的类型不同，但是如果他们的类型结构是一样，他们也可以互相赋值。

-   optional properties

`x?: Number;`

-   readonly properties: 属性使用 readonly, 变量使用`const`

`readyonly x: Number;`
`ReadonlyArray<T>`

-   function types

表示一个可被调用的类型注解

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

-   class Types

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = 'blue';
square.sideLength = 10;
square.penWidth = 5.0;
```

### Generics

泛型是在定义一个类型或者接口的时候，我们不知道其中某些参数的具体类型，但是又想对其进行某种约束，这时候就可以用泛型(常常是一个大写字母)来代替表示该类型。T 或者任意的大写字母被叫做泛型模板，会在运行时而不是编译时被代替。

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

### 枚举

枚举用来定义一种常量类型，帮助使用这个枚举中的值，可以有效的避免硬编码，
并且当使用了超出枚举范围内的值时，方便的提示错误。

```ts
enum Response {
    No = 0,
    Yes = 1
}
```

反向隐射：

```ts
enum isSuccess {
    0 = 'no',
    1 = 'yes'
}

isSucess[0]; // no
isSucess[no]; // 0
```

常量枚举：

```ts
const enum Tristate {
    False,
    True,
    Unknown
}

const lie = Tristate.False; // goes => const lie = 0;
```

### 类型进阶

-   类型断言：

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

-   类型守护：

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

-   类型别名

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

### 类型工具：

-   extends

```ts
T extends U ? X : Y
```

如果 T 类型可以认为是继承于U类型（要么 T 和 U 是同一种基础类型，要么 U 类型中所有的属性都能在 T 中找到，也就是 T 是 U 的超集，U 是 T 的子集），则取 X, 否则取 Y;

-   typeof: 从变量中读出其类型(通常由 ts 推断得出)

这允许你告诉编译器，一个变量的类型与其他类型相同

```ts
let foo = 123;
let bar: typeof foo; // 'bar' 类型与 'foo' 类型相同（在这里是： 'number'）

bar = 456; // ok
bar = '789'; // Error: 'string' 不能分配给 'number' 类型
```

-   keyof: 捕获键的名称

keyof 操作符能让你捕获一个类型的键。例如，你可以使用它来捕获变量的键名称，在通过使用 typeof 来获取类型之后：

```ts
const colors = {
    red: 'red',
    blue: 'blue'
};

type Colors = keyof typeof colors;

let color: Colors; // color 的类型是 'red' | 'blue'
```

-   Omit: 从类型中移除某个属性
-   Partial: 允许使用类型中的部分类型
-   &: 交叉类型
-   |: 联合类型

从数组中自动生成联合类型：

```ts
// 用于创建字符串列表映射至 `K: V` 的函数
function strEnum<T extends string>(o: Array<T>): { [K in T]: K } {
    return o.reduce((res, key) => {
        res[key] = key;
        return res;
    }, Object.create(null));
}

// 创建 K: V
const Direction = strEnum(['North', 'South', 'East', 'West']);

// 创建一个类型
type Direction = keyof typeof Direction;

// 简单的使用
let sample: Direction;

sample = Direction.North; // Okay
sample = 'North'; // Okay
sample = 'AnythingElse'; // ERROR!
```

-   as: 类型推断(当你比 ts 编译器更懂该变量的类型时，可以强制指定该变量的类型，防止编译报错)
-   declare module: 可以用于为第三方模块添加类型, 或者覆盖第三方的类型

```ts
// my-typings.ts
declare module 'plotly.js' {
    interface PlotlyHTMLElement {
        removeAllListeners(): void;
    }
}

// MyComponent.tsx
import { PlotlyHTMLElement } from 'plotly.js';
import './my-typings';
const f = (e: PlotlyHTMLElement) => {
    e.removeAllListeners();
};
```

更多工具类型参考：[utility-types](https://github.com/piotrwitek/utility-types)

### .d.ts

## React + TS

常用的 React+ts 应用:

```tsx
import React, { Component, useState, useRef } from 'react';
import logo from './logo.svg';
import './App.css';

interface HelloProps {
    message: string;
}

interface AsyncTask {
    (aPromise: Promise<any>): Promise<any>;
}

// custom hook
export function useLoading() {
    const [isLoading, setState] = useState(false);
    const load: AsyncTask = asyncTask => {
        setState(true);
        return asyncTask.finally(() => setState(false));
    };
    // 当一个数组有两种类型的时候，为了避免类型推断，这里显式定义类型。
    return [isLoading, load] as [boolean, AsyncTask];
}

// 语法更冗长，但是没有突出的优点，优先使用普通函数语法。
const HelloWorld: React.FunctionComponent<HelloProps> = props => {
    const [val, toggle] = useState(false);
    const inputRef = useRef<HTMLInputElement | null>(null);

    return (
        <div>
            <input type="text" ref={inputRef} />
            <span onClick={() => inputRef.current && inputRef.current.focus()}>
                {props.message}
            </span>
        </div>
    );
};

// 为app.props声明defaultProps和其他props的联合类型
type AppProps = typeof App.defaultProps & {
    prefix?: string;
};

// 这里forwardRef是一个泛型函数，
// 第一个泛型参数标识接收的组件类型是一个函数式组件，并且它的类型必须是HTMLButtonElement
// 第二个泛型参数限制了该组件的props, propTypes, defaultProps必须是ButtonProps类型
// 两个参数同时限制了返回的组件的props(如果不为空)必须是HTMLButtonElement类型的ref属性和ButtonProps的属性
type ButtonProps = React.PropsWithChildren<{ type: 'submit' | 'button' }>;
const FancyButton = React.forwardRef<HTMLButtonElement, ButtonProps>(
    (props, ref) => (
        <button ref={ref} type={props.type}>
            {props.children}
        </button>
    )
);

class App extends Component<AppProps> {
    private buttonRef = React.createRef<HTMLButtonElement>();

    static defaultProps = {
        name: 'world'
    };

    // 自动bind this, 声明该函数是一个MouseEventHandler, 并且e.target是HTMLAnchorElement
    onClick: React.MouseEventHandler<HTMLAnchorElement> = e => {
        this.setState({});
    };

    render() {
        return (
            <div className="App">
                <header className="App-header">
                    <img src={logo} className="App-logo" alt="logo" />
                    <HelloWorld message="123" />
                    <a
                        className="App-link"
                        onClick={this.onClick}
                        href="https://reactjs.org"
                        target="_blank"
                        rel="noopener noreferrer"
                    >
                        <FancyButton type="button" ref={this.buttonRef}>
                            Learn React
                        </FancyButton>
                    </a>
                </header>
            </div>
        );
    }
}

export default App;
```

### Refrence

[React & Redux in TypeScript - Static Typing Guide](https://github.com/piotrwitek/react-redux-typescript-guide)