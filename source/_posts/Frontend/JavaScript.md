---
title: JavaScript
categories:
  - Front-End
  - JavaScript
---

# JavaScript

- 单线程: 在浏览器 & 非浏览器环境中同样. 单线程也就意味着 JavaScript 在同一时间只能进行一项任务, 如果有多项任务的话, 需要对任务进行排队, 完成一个才能继续下一个.

### 数据类型 Type

- 值类型(基本类型)：字符串(String), 数字(Number), 布尔(Boolean), 空(Null), 未定义(Undefined), Symbol

- 引用数据类型(对象类型)：对象(Object), 数组(Array), 函数(Function)

  - 还有两个特殊的对象：正则(RegExp)和日期(Date)

- `typeof ...`: 检测机器码后三位数字 - 少 null 多 function (_判断基础数据类型 except null_)
  - 返回 object: 因为 object does implement Call => 实例化后的对象
  - 返回 function: 因为 object does not implement Call
- `A instanceof B`: 只会 return boolean - (_借助 proto 判断类型_)
  - 检测 A 是否时由 B 对象实例化产生
  - 顺着原型链寻找 - `[] instanceof Object = true`

### 严格模式 use strict

- 保留关键字 - 关键字无法用作其他名字 `var let = 10;`
- 禁止 this 关键字指向全局对象
  - 非严格模式的全局指向 = window
  - 严格模式下 = undefined
- 不允许对只读属性赋值
  - 不能更改 `const`
- 不允许删除函数
  - `delete func`
- 不允许删除变量或对象
  - `delete variable`
- 不允许使用未声明的变量等
  - `v = 10;`

### **this**

- 在方法中, this 表示该方法所属的对象

  ```javascript
  const obj = {
    name: "Alice",
    greet: function () {
      console.log(this.name); // 输出: Alice
    },
  };
  obj.greet(); // this 指向 obj
  ```

- 如果单独使用, this 表示全局对象

  `console.log(this); // window`

- 在函数中

  - 非严格模式: this 表示全局对象

    ```javascript
    function sayHello() {
      console.log(this); // 浏览器中: window
    }

    sayHello();
    ```

  - 未定义的(undefined) (防止意外访问 / 修改全局变量)

    ```javascript
    "use strict";
    function sayHello() {
      console.log(this); // 输出: undefined
    }

    sayHello();
    ```

- 在事件中, this 表示接收事件的元素

  - 在事件处理程序中，this 指向触发事件的元素
    ```javascript
    const button = document.getElementById("myButton");
    button.addEventListener("click", function () {
      console.log(this); // 输出: <button id="myButton"></button>
    });
    ```
  - 如果使用箭头函数，this 不会指向触发事件的元素，而是继承自其定义时的上下文。
    ```javascript
    button.addEventListener("click", () => {
      console.log(this); // 在全局上下文中，箭头函数的 this 是 window
    });
    ```

- call(), apply(), bind()

  - 改变函数的 this 指向 (function 对象自带的方法)
  - 区别：
    1. call、apply 直接调用原函数, bind 返回一个新函数
    2. call、bind 直接传递参数, apply 以数组形式传递参数

  ```javascript
  const person = {
    name: "Bob",
  };

  function greet() {
    console.log(this.name);
  }

  greet.call(person, ...); // 输出: Bob
  greet.apply(person, [...]); // 输出: Bob
  const p = greet.bind(person, ...);
  p(); // 输出: Bob
  ```

### Promise

为了避免`回调地狱`的异步解决方案, 目的是更加优雅地书写复杂的异步任务.

- 回调地狱: 为了实现顺序, 嵌套很多层的回调函数

#### Promise 创建

```javascript
const promise = new Promise(function (resolve, reject) {
  // 异步操作代码
  if (/* 操作成功 */) {
    resolve('成功的结果'); // 状态变为 fulfilled
  } else {
    reject('失败的原因'); // 状态变为 rejected
  }
});
```

- resolve 和 reject 的作用域只有起始函数, 不包括 then 以及其他序列 (不影响外部代码);
- resolve 和 reject 不能终止函数运行, 直到 return.

#### Promise 特点:

1. 对象的状态不受外界影响. Promise 对象代表一个异步操作, 有三种状态：

- pending: 初始状态, 不是成功或失败状态.
- fulfilled: 意味着操作成功完成.
- rejected: 意味着操作失败.

2. 一旦状态改变, 就不会再变, 任何时候都可以得到这个结果.

#### Promise 方法：

- then：用于处理 Promise 成功状态的回调函数. `resolve` 传递的结果
- catch：用于处理 Promise 失败状态的回调函数. `reject` 传递的结果
- finally：无论 Promise 是成功还是失败, 都会执行的回调函数. 清理操作

#### 什么时候适合使用 Promise 而不是传统回调函数？

答：当需要多次顺序执行异步操作的时候, 例如, 如果想通过异步方法先后检测用户名和密码, 需要先异步检测用户名, 然后再异步检测密码的情况下就很适合 Promise.

### var, let, const

- **Redefinition**: 改 type
  - `var`: Allowed
  - `let`, `const`: Not allowed
- **Reassignment**: 变 content
  - `var`, `let`: Allowed
  - `const`: Not allowed (except for object/array properties)
- **Hoisting**: 提升变量
  - `var`: Hoisted
    - `console.log(x); var x = 5; // Undefined, due to hoisting. `
  - `let`, `const`: Not hoisted (Temporal Dead Zone)
    - `console.log(y); let y = 10; // Error, no hoisting with let.`
- **Scope**:

  - `var`: Function-scoped
  - `let`, `const`: Block-scoped

  ```javascript
  function testLetConst() {
    if (true) {
      var x = 10;
      let y = 20;
      const z = 30;
    }
    console.log(x); // 10，因为 `var` 在函数作用域内有效
    console.log(y); // ReferenceError: y is not defined
    console.log(z); // ReferenceError: z is not defined
  }

  testLetConst();
  ```

- **Loops by while / for**:
  - `var`, `let`: Supported
  - `const`: Supported for arrays/objects, not primitive values.

### 数据存储形式 - 堆栈

- 栈
- 计算机为*原始类型*开辟的一块内存空间 - string, number
- `var a = 1; var b = a, b = 1 // ab不同`
- 堆
- 计算机为*引用类型*开辟的一块内存空间 - object - 给堆提供一大块空间, 通过*reference*找
- `var a = {key:1}; var b = a; b.key = 2 // ab相同`

### 深浅拷贝 - 取决于是否遍历到原始类型 - recursion

如果属性是一个基本数据类型, 拷贝就是基本类型的值, 如果属性是引用类型, 拷贝的就是内存地址

- 浅拷贝
  - 遍历赋值
  - 将原对象或原数组的引用直接赋给新对象, 新数组, 新对象只是对原对象的一个引用, 而不复制对象本身, 新旧对象还是共享同一块内存
- 深拷贝
  - JSON.parse() 和 JSON.stringify()
    - `var a = JSON.parse(JSON.stringify(obj))`
  - 拷贝“值”而不是“引用”

### 类型转换

- 隐式转换 `if (var) {}`
  - `NaN, 0, undefined, null, ""` 默认转换为 false. 其他都是 true
  - operator &&, || 在隐式情况的取值
  - `(0 || 5) => 5`
  - `(0 && 5) => 0`
- == and === diff
  - `==`比较值
  - `===`比较 type

### Async & Await

`async function name([param[, param[, ... param]]]) { statements }`

- async 函数返回一个 Promise 对象, 可以使用 then 方法添加回调函数
- Await 只能在 Async function 中使用

### null & undefined

- undefined 表示一个变量没有被声明, 或者被声明了但没有被赋值(未初始化) , 一个没有传入实参的形参变量的值为 undefined, 如果一个函数什么都不返回, 则该函数默认返回 undefined.

  - Javascript 将未赋值的变量默认值设为 undefined ; Javascript 从来不会将变量设为 null

  ```javascript
  // 变量被声明但未赋值
  let a;
  console.log(a); // 输出: undefined

  // 对象属性不存在
  const obj = { name: "Alice" };
  console.log(obj.age); // 输出: undefined

  // 函数没有返回值
  function foo() {}
  console.log(foo()); // 输出: undefined

  // 访问数组中不存在的索引
  const arr = [1, 2, 3];
  console.log(arr[5]); // 输出: undefined

  // 函数调用时未传入参数
  function greet(name) {
    console.log(name); // 输出: undefined
  }
  greet();
  ```

- null 则表示"什么都没有", 即"空值".

  - 是程序员手动赋予的值，用来表明变量或属性有意设置为空。

  ```javascript
  // 手动赋值为 null
  let b = null;
  console.log(b); // 输出: null

  // 清空对象引用
  let obj = { name: "Alice" };
  obj = null;
  console.log(obj); // 输出: null

  // 表示空对象
  let user = null; // 表示当前没有分配用户
  ```

### 装箱拆箱

- 装箱 `new Number()`
- 拆箱 `obj.valueOf()`

  - 如果有 value 那就返回, 否则返回对象本身

`toPrimitive(input, type)`

1.  如果 input = 原始类型的值 - 直接返回
2.  `input.valueOf()` = 原始类型, 直接返回
3.  `input.toString()` = string = 原始类型 并且返回
    - 如果是 object - 返回`[object type]`
4.  报错

- `console.log([] + {}) = [object Object]`
- `console.log([] + []) = ""`

### 事件

事件是文档和浏览器窗口中发生的特定的交互瞬间

- 直接在标签内直接添加执行语句,
- 定义执行函数.

#### addeventlistener 监听事件

事件类型分两种：事件捕获、事件冒泡.

- 事件捕获：由外往内, 从事件发生的顶点开始, 逐级往下查找, 一直到目标元素.

- 事件冒泡：由内往外, 从具体的目标节点元素触发, 逐级向上传递, 直到根节点.

- 事件委托：利用事件冒泡, 把子元素的事件都绑定到父元素上. 如果子元素阻止了事件冒泡, 那么委托也就没法实现了

- 自定义事件

### 防抖 & 节流

- 防抖：指触发事件后在 n 秒内函数只能执行一次, 如果在 n 秒内又触发了事件, 则会重新计算函数执行时间.

  - setTimeout

- 节流：是指连续触发事件但是在 n 秒中只执行一次函数
  - 时间戳 Timestamp / 定时器 Timers

### 垃圾回收机制和内存机制

- 垃圾回收机制：自动内存管理机制, 垃圾收集器会定期的找出那些不在继续使用的变量, 然后释放内存.
- 标记清除：最常用的垃圾回收算法之一. 当变量进入环境(函数中声明一个变量) 时, 它被标记为进入环境. 当变量离开环境时(函数执行结束) , 它被标记为离开环境. 垃圾回收器会定期检查所有变量, 并清除被标记为“离开环境”的变量
- 引用计数：跟踪每个值被引用次数, 当一个值的引用次数变为零时, 说明没有任何代码在使用它, 垃圾回收会释放该值所占内存. 容易出现循环引用问题, 导致内存泄漏
  - **内存泄漏** 不再使用的变量, 它们所占用的内存 不去清除的话就会造成内存泄漏

### JS 延迟加载的方式

1. 把 JS 放在页面的最底部
2. script 标签的 defer 属性：脚本会立即下载但延迟到整个页面加载完毕再执行. 该属性对于内联脚本无作用 (即没有 `src` 属性的脚本).
3. 是在外部 JS 加载完成后, 浏览器空闲时, Load 事件触发前执行, 标记为 async 的脚本并不保证按照指定他们的先后顺序执行, 该属性对于内联脚本无作用(即没有 `src` 属性的脚本).
4. 动态创建 script 标签, 监听 dom 加载完毕再引入 js 文件

### 同步 & 异步

同步往往会阻塞, 没有数据过来, 我就等着; 异步则不会阻塞, 没数据来我干别的事, 有数据来去处理这些数据.

- 同步：上一件事情没有完成, 继续处理上一件事情, 只有上一件事情完成了, 才会做下一件事情. 下面代码会等待上面 同步代码执行完毕
  - for 循环语句, alert(),console.log()等
- 异步： 规划要做一件事情,如果是异步事情, 不是当前立马去执行这件事情, 需要等一定的时间, 这样的话, 我们不会等着他执行, 而是继续执行下面的操作. 下面代码不会等待上面异步代码执行完毕
  - 所有定时器, ajax 异步请求, 所有的事件绑定都是异步;

### setTimeout 与 setInterval 区别与机制：

setTimeout()和 setInterval()经常被用来处理延时和定时任务.

- setTimeout() ：方法用于在指定的毫秒数后调用函数或计算表达式
- setInterval()：则可以在每隔指定的毫秒数循环调用函数或表达式, 直到 clearInterval 把它清除.

### require 与 import 的区别和使用(CommonJS 规范和 es6 规范)

1. import 是 ES6 中的语法标准也是用来加载模块文件的, import 函数可以读取并执行一个 JavaScript 文件, 然后返回该模块的 export 命令指定输出的代码. export 与 export default 均可用于导出常量、函数、文件、模块, export 可以有多个, export default 只能有一个.

- import 是静态加载，编译阶段就会解析并加载模块，提高了性能。

```javascript
export default function greet(name) {
  console.log(`Hello, ${name}`);
}

// 导入
import greet from "./module.js";
```

2. require 定义模块：module 变量代表当前模块, 它的 exports 属性是对外的接口. 通过 exports 可以将模块从模块中导出, 其他文件加载该模块实际上就是读取 module.exports 变量, 他们可以是变量、函数、对象等. 在 node 中如果用 exports 进行导出的话系统会系统帮您转成 module.exports 的, 只是导出需要定义导出名.

- require 在代码运行时动态加载模块，因此它可以出现在条件语句或函数内部。

```javascript
module.exports = { a, add };

// 导入
const { a, add } = require("./module.js");
```

### require 与 import 的区别

- require 是 CommonJS 规范的模块化语法, import 是 ECMAScript 6 规范的模块化语法;

- require 是运行时加载, import 是编译时加载;

- require 可以写在代码的任意位置, import 只能写在文件的最顶端且不可在条件语句或函数作用域中使用;

- require 通过 module.exports 导出的值就不能再变化, import 通过 export 导出的值可以改变;

- require 通过 module.exports 导出的是 exports 对象, import 通过 export 导出是指定输出的代码;

- require 运行时才引入模块的属性所以性能相对较低, import 编译时引入模块的属性所所以性能稍高.

### target 和 currentTarget 区别

都是事件对象上的属性

- event.target：返回触发事件的元素
- event.currentTarget：返回绑定事件的元素(相当于事件中 this)

```javascript
document.getElementById("parent").addEventListener("click", function (event) {
  console.log("target:", event.target); // 实际触发点击的子元素
  console.log("currentTarget:", event.currentTarget); // 绑定事件的父元素
});
```

### prototype 和 proto 的关系是什么

- prototype: 所有 function / class 都会有一个 prototype 属性, 它就是函数的原型对象
- proto: 所有实例对象上都会有一个 proto 属性, 它等同于函数的原型对象

```javascript
function Person() {}
const person = new Person();

console.log(Person.prototype); // 构造函数的原型对象
console.log(person.__proto__); // 实例对象的 __proto__

// 关系
console.log(Person.prototype === person.__proto__); // true
```

### 原型和原型链

- **原型** portoType 这个属性就是函数的原型
- **原型链**
  1. 所有对象都有原型, 而原型本身就是对象, 所以原型也有自己的原型对象, 就形成原型链
  2. 如果对象本身没有属性, 则就会去原型链上去找. Object 原型对象的原型值为 null

### new 操作符具体做了什么？

- 在内存创建一个新对象
- 把构造函数中 this 指向新建的对象
- 会在新对象上添加一个**proto**属性,指向函数的原型对象 prototype
- 判断函数返回值,如果值是引用类型就直接返回值; 否则返回 this(创建的新对象)

```javascript
function Person(name) {
  this.name = name;
  return { age: 30 }; // 返回引用类型
}

const person = new Person("Alice");

console.log(person); // { age: 30 }
console.log(person.name); // undefined (this 被覆盖)
```

### 什么是 AJAX？如何实现？

ajax 是一种能够实现网页局部刷新的技术, 可以使网页异步刷新.

1. 创建核心对象 XMLhttpRequest;

2. 利用 open 方法打开与服务器的连接;

3. 利用 send 方法发送请求; ("POST"请求时, 还需额外设置请求头)

4. 监听服务器响应, 接收返回值.

### Javascript 作用域链?

如果当前作用域没有找到属性或方法, 会向上层作用域查找, 直至全局函数, 这种形式就是作用域链

```javascript
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";

    console.log(innerVar); // 查找到 inner 函数作用域
    console.log(outerVar); // 查找到 outer 函数作用域
    console.log(globalVar); // 查找到全局作用域
  }

  inner();
}
outer();
```

### 为什么 JS 是单线程, 而不是多线程 [常考]

单线程是指 JavaScript 在执行的时候, 有且只有一个主线程来处理所有的任务. 目的是为了实现与浏览器交互.

- 如果 JavaScript 是多线程的, 现在我们在浏览器中同时操作一个 DOM, 一个线程要求浏览器在这个 DOM 中添加节点, 而另一个线程却要求浏览器删掉这个 DOM 节点, 那这个时候浏览器就会很郁闷, 他不知道应该以哪个线程为准. 所以为了避免此类现象的发生, 降低复杂度, JavaScript 选择只用一个主线程来执行代码, 以此来保证程序执行的一致性.

### 请说出三种减低页面加载时间的方法

- 压缩 css、js 文件
- 合并 js、css 文件, 减少 http 请求(精灵图)
- 外部 js、css 文件放在最底下
- 减少 dom 操作, 尽可能用变量替代不必要的 dom 操作

### session 与 cookie 的区别

- 保存位置
  - session 保存在服务器, 客户端不知道其中的信息;
  - cookie 保存在客户端, 服务器能够知道其中的信息.
- 保存内容
  - session 中保存的是对象
  - cookie 中保存的是字符串.
- 路径
  - session 不能区分路径, 同一个用户在访问一个网站期间, 所有的 session 在任何一个地方都可以访问到.
  - cookie 中如果设置了路径参数, 那么同一个网站中不同路径下的 cookie 互相是访问不到的.

#### cookies 是干嘛的, 服务器和浏览器之间的 cookies 是怎么传的, httponly 的 cookies 和可读写的 cookie 有什么区别, 有无长度限制 ?

- cookies 是一些存储在用户电脑上的小文件. 它是被设计用来保存一些站点的用户数据, 这样能够让服务器为这样的用户定制内容, 后者页面代码能够获取到 cookie 值然后发送给服务器. 比如 cookie 中存储了所在地理位置, 以后每次进入地图就默认定位到改地点即可.

### 请描述一下 cookies, sessionStorage 和 localStorage 的区别

- 共同点
  - 都是保存在浏览器端, 且同源的.
- 区别
  - 数据传递
    - cookie 数据始终在同源的 http 请求中携带(即使不需要) , 即 cookie 在浏览器和服务器间来回传递.
    - sessionStorage 和 localStorage 不会自动把数据发给服务器, 仅在本地保存.
  - 路径
    - cookie 数据还有路径(path) 的概念, 可以限制 cookie 只属于某个路径下. 存储大小限制也不同, cookie 数据不能超过 4k, 同时因为每次 http 请求都会携带 cookie, 所以 cookie 只适合保存很小的数据, 如会话标识.
    - sessionStorage 和 localStorage 虽然也有存储大小的限制, 但比 cookie 大得多, 可以达到 5M 或更大.
  - 有效期
    - sessionStorage：仅在当前浏览器窗口关闭前有效, 自然也就不可能持久保持;
    - localStorage：始终有效, 窗口或浏览器关闭也一直保存, 因此用作持久数据;
    - cookie 只在设置的 cookie 过期时间之前一直有效, 即使窗口或浏览器关闭.
  - 作用域
    - sessionStorage 在不同的浏览器窗口中不共享, 即使是同一个页面;
    - cookie 和 localStorage 在所有同源窗口中都是共享的.

### 验证码是干嘛的, 是为了解决什么安全问题.

将一串随机产生的数字或符号, 生成一幅图片, 图片里加上一些干扰象素(防止 OCR) , 由用户肉眼识别其中的验证码信息, 输入表单提交网站验证, 验证成功后才能使用某项功能. 验证码一般是防止批量注册的, 人眼看起来都费劲, 何况是机器. 像百度贴吧未登录发贴要输入验证码大概是防止大规模匿名回帖的发生. 目前, 不少网站为了防止用户利用机器人自动注册、登录、灌水, 都采用了验证码技术.

### 性能优化

web 前端是应用服务器处理之前的部分, 前端主要包括：HTML、CSS、javascript、image 等各种资源, 针对不同的资源有不同的优化方式.

#### 内容优化

- 减少 HTTP 请求数. 这条策略是最重要最有效的, 因为一个完整的请求要经过 DNS 寻址, 与服务器建立连接, 发送数据, 等待服务器响应, 接收数据这样一个消耗时间成本和资源成本的复杂的过程. 常见方法：合并多个 CSS 文件和 js 文件, 利用 CSS Sprites 整合图像, Inline Images (使用 data：URL scheme 在实际的页面嵌入图像数据 ), 合理设置 HTTP 缓存等.
- 减少 DNS 查找
- 避免重定向
- 使用 Ajax 缓存
- 延迟加载组件, 预加载组件
- 减少 DOM 元素数量. 页面中存在大量 DOM 元素, 会导致 javascript 遍历 DOM 的效率变慢.
- 最小化 iframe 的数量. iframes 提供了一个简单的方式把一个网站的内容嵌入到另一个网站中. 但其创建速度比其他包括 JavaScript 和 CSS 的 DOM 元素的创建慢了 1-2 个数量级.
- 避免 404. HTTP 请求时间消耗是很大的, 因此使用 HTTP 请求来获得一个没有用处的响应(例如 404 没有找到页面) 是完全没有必要的, 它只会降低用户体验而不会有一点好处.

#### 服务器优化

- 使用内容分发网络(CDN) . 把网站内容分散到多个、处于不同地域位置的服务器上可以加快下载速度.
- GZIP 压缩
- 设置 ETag：ETags(Entity tags, 实体标签) 是 web 服务器和浏览器用于判断浏览器缓存中的内容和服务器中的原始内容是否匹配的一种机制.
- 提前刷新缓冲区
- 对 Ajax 请求使用 GET 方法
- 避免空的图像 src

#### Cookie 优化

- 减小 Cookie 大小
- 针对 Web 组件使用域名无关的 Cookie

#### CSS 优化

- 将 CSS 代码放在 HTML 页面的顶部
- 避免使用 CSS 表达式
- 使用 < link> 来代替 @import
- 避免使用 Filters

#### javascript 优化

- 将 JavaScript 脚本放在页面的底部.
- 将 JavaScript 和 CSS 作为外部文件来引用. 在实际应用中使用外部文件可以提高页面速度, 因为 JavaScript 和 CSS 文件都能在浏览器中产生缓存.
- 缩小 JavaScript 和 CSS
- 删除重复的脚本
- 最小化 DOM 的访问. 使用 JavaScript 访问 DOM 元素比较慢.
- 开发智能的事件处理程序
- javascript 代码注意：谨慎使用 with, 避免使用 eval Function 函数, 减少作用域链查找.

#### 图像优化

- 优化图片大小
- 通过 CSS Sprites 优化图片
- 不要在 HTML 中使用缩放图片
- favicon.ico 要小而且可缓存

#### 什么是 CDN？

- CDN(Content Delivery Network) 是内容分发网络的缩写, 它是一组位于全球不同地理位置的服务器群集, 旨在更快、更可靠地传输互联网上的内容给用户. CDN 的主要目的是通过将内容缓存在离用户更近的服务器上, 从而提高网站的性能和用户体验.

- 在 CDN 中, 当用户请求访问一个网站时, CDN 会根据用户的地理位置, 选择距离用户最近的服务器来响应请求. 这样做有以下几个优点：

- 加速网站访问速度：CDN 可以缓存网站的静态资源(如图片、视频、CSS、JavaScript 等) , 使这些资源可以更快地加载, 从而提升网站的访问速度.

- 减轻源服务器负载：CDN 可以分担源服务器的负载压力, 因为部分请求会由 CDN 节点来处理, 减少源服务器的压力, 提高网站的稳定性和可靠性.

- 提高全球访问速度：由于 CDN 节点分布在全球各地, 用户无论身处何地, 都可以通过最近的 CDN 节点获取内容, 减少网络延迟, 提高访问速度.

- 增强安全性：CDN 可以提供一定程度的安全防护, 例如 DDoS 攻击防护、SSL 加密等, 保护网站免受恶意攻击.

- 总的来说, CDN 通过在全球各地建立节点, 并缓存网站内容到这些节点上, 实现了更快、更稳定的内容传输, 从而提升了网站的性能和用户体验. 许多大型网站和应用程序都使用 CDN 来改善其服务的性能和可靠性.

#### JS DOM

##### DOM 加载过程

1. 发起请求
1. 浏览器输入 url
1. DNS 域名解析
1. 找到 IP
1. 向服务器发送请求
1. 接收请求
1. decode binary to html - index.html
1. 构建 DOM, 使用 HTML 解析器
   1. 从 Document 开始, 解析标签 - `</div>`
   2. 生成 node - `HTMLDivElement`
   3. 生成树的结构, 和 html 一一对应

- 遇到 link 的外部 css, 加载 CSS _(新线程)_
  - 解析 -> 构建 css 树
  - CSSStyleSheet -> CSSRule -> Selector & Declaration(语法, 描述)
- 遇到 script: *先*执行 js 内容, 完成后构建 DOM _(不开新线程)_ - 这就是为什么 script 加到底部
  - async 加载(fetch)完成后立即执行 (execution), 因此可能会阻塞 DOM 解析
  - defer 加载(fetch)完成后延迟到 DOM 解析完成后才会执行(execution)

1.  构建 render 树 = DOM 树 + CSS 树
2.  布局 layout & 绘制 paint:
    - 计算大小, 距离, 坐标..., 通过 UI 绘制
    - _reflow 回流_: 当元素属性改变 并且 影响布局 (width, height, margins...) = 刷新页面
    - _repaint 重绘_: 当元素属性发生改变 不影响布局 (color, font...) = 动态更新

##### DOM 事件 Event 周期

1. 触发 - 点击
2. 捕获 - 找到被点击的 component
3. 冒泡 - 自下而上触发

- `e.stopPropagation()` 阻止冒泡

- 通过冒泡增加性能, 给 parent 绑定委托
- `e.target.nodeName == "child"`

##### EventListener

- capture. 监听器会在时间捕获阶段传播到 event.target 时触发.
- passive. 监听器不会调用 preventDefault().
- once. 监听器只会执行一次, 执行后移除.
- signal. 调用 abort()移除监听器.

```javascript
element.addEventListener("click - capture type - 监听器", listener function 执行, signal - 调用来移除监听);
```

#### JS BOM

- history - 窗口浏览历史 - `window.history`
- location - 当前页面信息 - `window.location` / `document.location`

#### Function

##### 基本函数

- 匿名函数
  ```javascript
  (function(a, b) {
  	...
  }) (aValue, bValue);
  ```
- 回调函数 callback

  ```javascript
  f1(){
  	...
  }
  f2(func){
  	func()
  }

  f2(f1)
  ```

- 递归函数
  ```javascript
  f(a) {
  	return f(a-1)
  }
  ```
- 构造函数
  ```javascript
  function P(){}
  P var = new P();
  ```

##### 变量和函数提升

1. **put variable at the top, 但是 value assign 保留**
2. **put all functions right after variables**

#### Object Oriented Programming

##### 构造方式

- 函数对象 Function
  - `var a = new Function(...)`
- 普通对象 Object
  - `var = new Object()`

##### Prototype

原型链: 通过`__proto__`属性连接个个 object

- `Type.prototype.constructor === Type`
- `Type.__proto === Type.prototype`
- 属性
  - `__proto__`
  - `constructor`
  - `prototype` 更改属性, 用于 public 使用
    - example: `Type.prototype.property = 0`
