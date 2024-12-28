---
title: JavaScript
categories:
  - Front-End
  - JavaScript
---

## JavaScript

#### 数据类型

##### var, let, const

- **Redefinition**: 改 type
  - `var`: Allowed
  - `let`, `const`: Not allowed
- **Reassignment**: 变 content
  - `var`, `let`: Allowed
  - `const`: Not allowed (except for object/array properties)
- **Hoisting**:
  - `console.log(x); var x = 5; // Undefined, due to hoisting. `
  - `console.log(y); let y = 10; // Error, no hoisting with let.`
    - `var`: Hoisted
    - `let`, `const`: Not hoisted (Temporal Dead Zone)
- **Scope**:
  - `var`: Function-scoped
  - `let`, `const`: Block-scoped
- **Loops by while / for**:
  - `var`, `let`: Supported
  - `const`: Supported for arrays/objects, not primitive values.

##### Type

- undefined, boolean, number, string, Object, null
- `typeof ...`: 检测机器码后三位数字 - 少 null 多 function (_判断基础数据类型 except null_)
  - 返回 object: 因为 object does implement Call => 实例化后的对象
  - 返回 function: 因为 object does not implement Call
- `A instanceof B`: 只会 return boolean - (_借助 proto 判断类型_)
  - 检测 A 是否时由 B 对象实例化产生
  - 顺着原型链寻找 - `[] instanceof Object = true`

##### 数据存储形式 - 堆栈

- 栈
  - 计算机为*原始类型*开辟的一块内存空间 - string, number
  - `var a = 1; var b = a, b = 1 // ab不同`
- 堆
  - 计算机为*引用类型*开辟的一块内存空间 - object - 给堆提供一大块空间, 通过*reference*找
  - `var a = {key:1}; var b = a; b.key = 2 // ab相同`

##### 深浅拷贝 - 取决于是否遍历到原始类型 - recursion

- 遍历赋值
- Object.create
  - 大部分用于 浅拷贝
- JSON.parse() 和 JSON.stringify()
  - 深拷贝
  - `var a = JSON.parse(JSON.stringify(obj))`
  - object 转成 string, 然后转回去

##### 类型转换

- 隐式转换 `if (var) {}`
  - `NaN, 0, undefined, null, ""` 默认转换为 false. 其他都是 true
- operator &&, || 在隐式情况的取值
  - `(0 || 5) => 5`
  - `(0 && 5) => 0`
- == and === diff
  - `==`比较值
  - `===`比较 type

##### 装箱拆箱

- 装箱 `new Number()`
- 拆箱 `obj.valueOf()` - 如果由 value 那就返回, 否则返回对象本身
- `toPrimitive(input, type)`
  1.  如果 input = 原始类型的值 - 直接返回
  2.  `input.valueOf()` = 原始类型, 直接返回
  3.  `input.toString()` = string = 原始类型 并且返回
      - 如果是 object - 返回`[object type]`
  4.  报错
  - `console.log([] + {}) = [object Object]`
  - `console.log([] + []) = ""`

#### JS DOM

##### DOM 加载过程

1. 发起请求
   1. 浏览器输入 url
   2. DNS 域名解析
   3. 找到 IP
   4. 向服务器发送请求
2. 接收请求
   1. decode binary to html - index.html
   2. 构建 DOM, 使用 HTML 解析器
      1. 从 Document 开始, 解析标签 - `</div>`
      2. 生成 node - `HTMLDivElement`
      3. 生成树的结构, 和 html 一一对应
   - 遇到 link 的外部 css, 加载 CSS _(新线程)_
     - 解析 -> 构建 css 树
     - CSSStyleSheet -> CSSRule -> Selector & Declaration(语法, 描述)
   - 遇到 script: *先*执行 js 内容, 完成后构建 DOM _(不开新线程)_ - 这就是为什么 script 加到底部
     - async 加载(fetch)完成后立即执行 (execution)，因此可能会阻塞 DOM 解析
     - defer 加载(fetch)完成后延迟到 DOM 解析完成后才会执行(execution)
   1. 构建 render 树 = DOM 树 + CSS 树
   2. 布局 layout & 绘制 paint:
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

- capture。监听器会在时间捕获阶段传播到 event.target 时触发。
- passive。监听器不会调用 preventDefault()。
- once。监听器只会执行一次，执行后移除。
- signal。调用 abort()移除监听器。

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
