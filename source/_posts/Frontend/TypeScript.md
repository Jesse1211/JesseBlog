---
title: TypeScript
categories:
  - Front-End
  - TypeScript
---

## TypeScript

### 1. What is TypeScript

- `TypeScript` is super-set of JavaScript
  - ES6, OOP...
- static type check, find error when compelling
- 自动编译: TypeScript -> JavaScript

```ts
// TS
const hello: string = "Hello world!";
console.log(hello);

// JS
const hello = "Hello world!";
console.log(hello);
```

#### 1.1 Feature

- 类型批注 & 编译时类型检查 (可选)
  - 通过类型批注提供编译时启动类型检查的静态类型
  - `function Add(left: number, right: number): number {return 1;}`
- 类型推断
  - 发生在初始化变量
  - `let stc = "string";`
  - 如果无法推断, 默认`any`
- 接口: 描述对象的类型
  - `let tom : Person = {}`
- Generic programming `T`
  - 使用一些之后才会指定的类型
- 对象类型
  - `interface`只能定义对象类型
  - `type`可以定义对象, 也可以定义交叉, 联合, 原始类型...

### 2. 高级类型

#### 2.1 交叉类型

`T & U`: 合并多个类型, 包含所有 Property

```ts
function merge<T, U>(first: T, second: U): T & U {
  return { ...a, ...b };
}

const res = merge({ name: "Jesse" }, { age: 25 });
let res: T & U = {
  name: "Jesse",
  age: 25,
};
```

#### 2.2 联合类型

`T | U`: 多个类型中的一个

```ts
function formater(cmd: string[] | string) {
  if (typeof cmd === "string") {
    return cmd;
  }
  return cmd.join(" ").trim();
}
```

#### 2.3 类型别名

- `type SomeName = someValidTypeAnnotation`: 起个新名字
- Example 1

```ts
type some = boolean | string;

const a: some = true; // OK
const b: some = "hi"; // OK
const c: some = 123; // NO
```

- Example 2

```ts
type Container<T> = { value: T };

const a: Container<boolean> = { value: true };
```

#### 2.4 类型索引 `keyof`

类似于`Object.keys`

```ts
interface Button {
	type: string
	text: string
};

keyof Button === "type" | "text";
```

#### 2.5 类型约束 `extend`

- 添加约束

```ts
type BaseType = string | number | boolean;

function copy<T extends BaseType>(arg: T): T {
  return arg;
}
```

- work with `keyof`

```ts
function getValue<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

getValue({ a: 1 }, "a");
```

#### 2.6 映射类型 `in`

```ts
type Readonly<T> = {
	readonly [P in keyof T]: T[P]; // key: 'a' | 'b'
}

interface Obj {
	a: string;
	b: string;
}

type ReadonlyObj === Readonly<Obj>;

// interface ReadonlyObj {
//	readonly a: string; readonly b: string
// }
```

#### 2.7 条件类型

`T extends U ? X : Y`

- if T is subset of U, then X; else Y

### 3. 接口 interface

```ts
interface Child extends Parent1, Parent2, Parent3 {
	name: string;
	age!: number; // number | undefined
	readonly isMale: boolean; // 不可改
	say: (words: string) => string; // 函数
	[propName: string]: any; // 索引签名
}
```

### 4. Function Overload

- 声明: 多个 function with different input type
- 实现: 使用`|`包含所有可能的类型

```ts
function add(arg1: string, arg2: string): string
function add(arg1: number, arg2: number): number

function add(arg1: string | number, arg2: string | number) {
	if (typeof arg1 === 'string' &&& typeof arg2 === 'string') {
		return arg1 + arg2;
	}
	if (typeof arg1 === 'number' &&& typeof arg2 === 'number') {
		return arg1 + arg2;
	}
}
```

### 5. Generic Programming

- 不预先定义好具体的类型
- 适用于: function, interface, class

```ts
// function
function returnItem<T>(para: T): T {
	return para;
}

funciton swap<T, U>(tuple: [T, U]): [U, T] {
	return [tuple[1], tuple[0]];
}
swap([7, 'seven']);

// interface
interface ReturnItemFn<T> {
	(para: T): T;
}
const returnItem: ReturnItemFn<number> = para => para;

// class
class Stack<T> {
	private arr: T[] = [];
	public push(item: T) {
		this.arr.push(item);
	}
	public pop() {
		this.arr.pop();
	}
}
const stack = new Stack<number>();

// 约束泛型 extends
type Params = string | number;
class Stack<T extends Params> {
	private arr: T[] = [];
	public push(item: T) {
		this.arr.push(item);
	}
	public pop() {
		this.arr.pop();
	}
}
const stack = new Stack<boolean>();

// 索引泛型 keyof
function getVal<T extends object, U extends keyof T>(obj: T, key: U) {
	return obj[key];
}

// 多类型约束
interface Child extends Parent1, Parent2 {
}
```
