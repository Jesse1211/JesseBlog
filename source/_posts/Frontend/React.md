---
title: React
categories:
  - Front-End
  - React
---

### 1. What is React

- CORE: Design by Component; 组合 & 嵌套 来构成整体页面
- 数据流单向响应, 更安全更快

### 2. State & Props

- state & setState 在 `constructor` 中初始化, State 更新后会调用`render`. 组件内部管理
- props: 组件的 input parameters, from its parent class, 由外部传入
- Immutable props:
  - 更改 Props 不会 trigger `render`
  - Re-rendering child component by sending new props from outside

### 3. super() & super(props)

- `super(name) == sup.prototype.constructor.call(this.name)`
- 先有`super(props)`, 才有`this.props`, `this.parentProps`
- if `super()`, then `this.props = undefined`

### 4. Components

#### 4.1 Class Component

State & lifecycle methods manageable

- `this.props` is constant unless `this` changes

```ts
class classComponent extends React.Component {
	constructur(props){
		super(props);
		...
	}

	// Life cycle methods
	componentDidMount() {}
	...

	render () { return (); } // return in jsx style
}

const res = new classComponent(props);
```

#### 4.2 Function Component

- Manage State & lifecycle methods by hooks (return react component)
  - 适合无状态 / use Hooks
  - 不存在生命周期, 因为 hook 都继承于`R.C`
  - `useEffect` manage 生命周期 -> `componentDidMount` & `componentWillUnmount`
  - 需要 `useState` 管理状态
- `React.createElement(..);`: 创建 Component, 用 babel 转化成 `React.createClass`
- Do not have `this`: `props` is constant

```ts
function functionComponent(props: Props) {
  return <div>Hello from a function component!</div>;
}
const res = functionComponent(props);
```

### 5. 受控组件 Controlled Components & 非受控组件 UnControlled Components

- 受控组件
  - `const [value, setValue] = useState('');`
  - 组件状态相应外部数据: `onChange`. 比如 input 表单数据
  - 适合: 做验证, 条件渲染...
- 非受控组件
  - 表单元素的值由 DOM 自己控制
  - 不需要`state`
  - `const inputRef = useRef(null);`
  - 适合: 不需要频繁监控数值: 提交, 上传...

### 6. React Event - 合成事件 SyntheticEvent

- 跨浏览器包装器: React 模拟原生 DOM 事件所有能力的事件对象
  - 冒泡: React 沿着 Fiber Tree 向上查找有没有注册过对应事件的组件
  - 并没有接触到浏览器: 事件注册, 合成, 冒泡, 派发
- `e.nativeEvent`: 原生 DOM 事件
  `<Button onClick = {(e) => console.log(e.nativeEvent)} />

#### 6.1 事件监听器 - 核心机制: Event Delegation

- 组件/DOM node: 没有被绑定 Event listener (`onClick`, `onChange`)
  - **统一事件监听**: 把所有事件绑定到结构最外层容器 (`document` / `React root`)
- Process:
  1.  Initialize React: register event listener on `document`
  2.  Built Map：`Event -> Component Handler`
      - **自动清理**: Update map when component mount / unmount
  3.  当事件冒泡到 `document`，React 捕获事件，根据 Map 进行分发

#### 6.2 事件处理

- 在事件的冒泡阶段执行
- native event 会比 React 的 SyntheticEvent 触发得更早

1. 捕获阶段 (Capture Phase)
   1. window
   2. document
      - trigger native event listener (`document.addEventListener('click', ..., true)`)
   3. React Container
   4. button (target)
2. 目标阶段 (Target Phase)
   - 事件到达实际被点击/触发的元素
3. 冒泡阶段 (Bubble Phase) - **被 React 和浏览器分阶段处理**
   - 事件从目标元素向上传递回 `window`
   - **React 在 `document` 或根节点拦截到事件**
     - 创建一个 SyntheticEvent
     - 用自己的 Map 派发处理器
     - 模拟“冒泡”, 调用每一层组件的事件处理器 (`onClick`, `onChange`, ...)
       - 从事件源组件往上查找父组件的事件处理器并依次执行
   - 通过冒泡增加性能, 给 parent 绑定委托
     - `e.target.nodeName == "child"`

#### 6.3 冒泡 - Native first, then react

向上传播 - `Child -> Parent -> Root (document)`

1. 触发 child 组件的事件处理器
   - React SyntheticEvent
     - if `e.stopPropagation()`: 父组件不会收到这个事件
     - 冒泡流程中断
   - Native DOM
     - `e.nativeEvent.stopImmediatePropagation();`
     - `e.nativeEvent.stopPropagation();`
2. 冒泡到 parent 组件
3. 冒泡到 document (or React Root)

#### 6.4 EventListener

- capture: 监听器会在时间捕获阶段传播到 event.target 时触发
- passive: 监听器不会调用 preventDefault()
- once: 监听器只会执行一次，执行后移除
- signal: 调用 abort()移除监听器

```ts
element.addEventListener("click - capture type - 监听器", listener function 执行, signal - 调用来移除监听);
```

### 7. CSS usage

- 局部 css - 防止污染

#### 7.1 组件内 - style

`<div style = {{backgroundColor: "red"}} />`

- 无冲突, 写太多了容易混乱

#### 7.2 import .css

`import './App.css';`

- 全局生效, 样式之间会互相影响

#### 7.3 import .module.css

`import './App.module.css';`

- 引入的 css 不会影响后代组件
- 需要配置`modules:true` in `webpack`配置文件
- 不方便动态修改样式, 需要用内联方式编写

#### 7.4 CSS in JS - 由 JS 生成的的 CSS

- 第三方库: MUI

### 8. React lifecycle 不同阶段

#### 8.1 创建

- `constructor`
  - get parent props
  - 初始化 state, 挂载方法到 this, ...
- `getDerivedStateFromProps`
  - static
  - 比较`props` & `state`添加限制条件, 防止无用 state 更新
  - 返回新的对象作为 state 或者 null 表示不需要更新
- `render`
  - 渲染 **VDOM**
  - 不能用`setState`
  - `ReactDOM.render()`返回的组件实例
    - if 渲染组件, return 组件实例
    - if 渲染 dom, return 具体 dom 节点
- `componentDidMount`
  - 挂载到真实 DOM 后执行
  - 多用于执行数据获取, 事件监听...

#### 8.2 更新

- `getDerivedStateFromProps`
- `shouldComponentUpdate`
  - boolean: 通过比较`props` & `state`
  - 不建议深层比较, 影响效率
  - 不能调用`setState` - loop
- `render`
- `getSnapshotBeforeUpdate`
  - DOM 还没更新
  - 获取组件更新前的信息: 组件的滚动位置...
  - 返回的`Snapshot` value 被作为参数传入`componentDidUpdate`
- `componentDidUpdate`
  - 更新结束后
  - 比较`props` & `state`变化做对应操作 - 修改数据

#### 8.3 卸载

`componentWillUnmount`

- 卸载前, 清理监听事件, 取消订阅网络请求...
- 无法被再次挂载, 只能重新创建

### 9. 组件通信

**单向数据流**

#### 9.1 Parent -> Child

```ts
<EmailInput email="123@gmail.com" />
```

#### 9.2 Child -> Parent

使用函数回调: `onSet` OR `bind`

```ts
// parent
<Child onSetData = {setData} />
<Child getData = {this.setState.bind(this)} />

// child
<button onClick = {onSetData(100)} />
<button onClick = {this.getData.bind(this, 100)} />
```

#### 9.3 Between Siblings

```ts
// parent
<SiblingA count = {this.state.count} />
<SiblingB count = {this.state.count} />
```

#### 9.4 Parent -> descendant

- `React.createContext` - 看 Hook 部分

#### 9.5 Between un-connected components

使用 redux 进行全局资源管理

### 10. 看懂什么是 this

- def: Scope / Context of its use

#### 10.1 Global scope

- 顶层声明:
  - `this = undefined (strict)` OR `this = global scope`
  - 绝对不会自动指向定义的 object.

```ts
// this = Window
// this = {frames: Window, postMessage: ƒ, blur: ƒ, focus: ƒ, …}
this.alert("test") = Window.alert("test");
```

#### 10.2 Function scope

`this` 取决于 function 如何被 call

- **Explicitly set**:

  - `call`, `apply`, `bind` the value of `this` in a function (include Arrow function)

  ```ts
  const object = { a: 5, b: 7 };
  const f = function (c, d) {
    return this.a + this.b + c + d;
  };

  // call
  f.call(object, 12, 4); // 5 + 7 + 12 + 4

  // apply
  f.apply(object, [12, 4]); // 5 + 7 + 12 + 4

  // bind
  const bindF = f.bind(object);
  bindF(12, 4); // 5 + 7 + 12 + 4
  ```

  - Arrow function
    - 无法修改 this 绑定 - 本质是把 f 放到 constructor 中
    - `const f = () => { return this.a + this.b }`
    - `constructor() { this.f = () => { ... }; }`
    - global scope
      - this 和 object 无关:
        - OK: `const f = () => { return object }`
        - NO: `const f = () => { return this.object }`
        - `this.object = undefined`
        - 因为箭头函数不会创建自己的 this, 它的 this 取决于定义时的词法作用域
          - object: local variable (block scoped)
    - class scope
      - `this` 在内部默认指向实例

- **Not set**:
  - defaults: `this` = global context
  - strict mode: this = it's explicit value
    - if not set, `this = undefined`
  ```ts
  const f = function () {
    return this;
  }; // Window
  const f = function () {
    "use strict";
    return this;
  }; // undefined
  ```

#### 10.3 Object context

this refer to the object to itself.

```ts
const object = {
  a: 2,
  b: 3,
  f: function () {
    return this.a + this.b;
  },
};
object.f(); // returns 2 + 3
```

#### 10.4 事件绑定 - this

bind 会改变 this 指向

```jsx
// slow: re-bind while re-rendering
<div onClick={this.handleClick.bind(this)} />
<div onClick={e => this.handleClick(e)} />

 // fast: bind一次, re-render不会重新执行constructor
this.handleClick = this.handleClick.bind(this) // in constructor
<div onClick={this.handleClick} />
```

### 11. Axios

- HTTP requests: from `node.js` or `XMLHttpRequests` from the browser
  - supports Promise API. `.then().catch()`
- Interception: HTTP requests and responses
  - enables client-side protection against XSRF
- Can cancel requests

```ts
axios
  .get("url")
  .then((response) => {
    // Code for handling the response
  })
  .catch((error) => {
    // Code for handling the error
  });
```

### 12. Ref

- Ref: assigned to constructed component's `ref`, so they can be referenced throughout the component
  - like `document.querySelector`
  - 绕过 React “声明式编程”, 进行“命令式操作”
- 不适合
  - 可以通过 refs 更新组件 (推荐用 props / state)
  - 过多使用 refs 会破坏封装性
- 适合
  - 对 DOM 元素的焦点控制 `input.focus()`, 内容选择
  - 对 DOM 元素的操作和对组件实例的操作
- `.current`: DOM 节点或者子组件实例
- 原理
  - `<MyComponent ref={someRef} />`
  - React 会尝试把 `someRef.current` 设置为：
  1.  if 类组件 -> 就是类的实例 `this`
  2.  if DOM 元素 -> 就是 HTML 元素节点 `<div>`, `<input>`..
  3.  if `forwardRef` 函数组件 -> 是你手动暴露的 DOM 或实例
  4.  else -> 不支持，会报错或为 `null`

#### 12.1 传入 String (Deprecated)

通过 this.refs.传入的 string 的格式获取对应元素

```jsx
render() {
	return <div ref="theRef" />;
}
```

#### 12.2 传入对象 `createRef()` - use in Class

创建 refs 后, 将 ref 属性加入元素中

```jsx
class Component1 extends React.Component {
  theRef: React.RefObject<HTMLDivElement | null>;

  constructor(props) {
    super(props);
    this.theRef = React.createRef();
  }

  componentDidMount(): void {
    this.theRef.current.innerHTML = "Hello"; // content
  }

  render() {
    return <div ref={this.theRef} />;
  }
}
```

#### 12.3 传入函数 (more flexible)

ref 传入函数时, 在 render 过程中, callback 会传入一个元素对象, 然后通过实例将对象保存

```jsx
class Component1 extends React.Component {
  render() {
    return (
      <div
        ref={
          (e) => {
            this.theRef = e;
            console.log(this.theRef);
          } // execute if rendered
        }
      />
    );
  }
}
```

#### 12.4 传入 Hook - `useRef`

不能使用 ref 属性, 因为 component 本身没有实例, ref 对象接收到的是组件的挂载实例

```ts
function f(props) {
  const theRef = useRef(null);
  return (
    <div
      ref={theRef}
      onClick={() => {
        console.log(theRef);
      }}
    >
      {" "}
      // 获取ref属性
    </div>
  );
}
```

### 13. 高阶组件 Higher-order function

- Higher-order function: 满足一个条件
  - input at least one function
  - output a function
- 类似装饰者模式
- `const EnhancedComponent = highOrderComponent(WrappedComponent);`
- React 组件不会直接映射为 DOM 节点, 只有原生元素或包含原生元素的组件才会变成真实 DOM
  - DOM 中显示的是 A: `B = HOC(A); <B />`
  - 除非: `HOC`添加了 DOM elements

#### 13.1 约定

- props 一致
  - for encapsulation and scalability
- render()中不能使用高阶组件
  - HOC 是在组件定义阶段使用的, 而不是在渲染时动态创建
  - render 创建一个新的组件类型导致 React diff 失败 (认为是全新的组件, 强制卸载/重新挂载)
- 用 compose 组合高阶组件
  - 避免嵌套地狱
  - `const Enchanced = compose(withAuth, withLogger, withRouter)(BaseComponent);`
- **Ref**

  - ref -> 最外层容器组件实例, 不是被包裹的组件
    - 挂载在 DOM 上的是外层 HOC, 不是 `WrappedComponent`
  - function component 获得 ref: `React.forwardRef`

    - (因为没有实例)
    - ref 无法和 props 一样传递

    ```jsx
    const MyComponent = React.forwardRef((props, ref) => {
      return <div ref={ref}>Hello</div>;
    });

    // 使用时：
    const myRef = useRef();
    <MyComponent ref={myRef} />;
    ```

#### 13.2 应用

- 提高代码复用性 & 灵活性
- 适用于多模块间相同功能 - 权限控制, 日志记录, 异常处理...

```jsx
function f(WrappedComponent) {
	return class extends Component {
		componentWillMount() {
			this.setState({localStorage.getItem('data')});
		}

		render() {
			return <WrappedComponent data = {this.state.data} />
		}
	}
}

class ComponentTest extends Component {
	render () {
		return <div>{this.props.data}</div>
	}
}

const newF = f(ComponentTest);
```

### 14. Catch Error - Error Boundaries Component

1. Catch Error from ANY nodes in the sub-tree
2. Print error
3. Display fallback UI (代替渲染的备用 UI), not render the crashed sub-tree

#### 14.1 条件

- 使用 `static getDerivedStateFromError()`
  - 渲染备用 UI
- 使用 `componentDidCatch()`
  - 打印错误信息
- 无法捕获的
  - 事件处理, 异步代码

### 15. **组件间过渡动画**

### 16. setState 执行机制

- 使用`setState`更新 state 状态, 然后重新执行 render, 导致页面更新
- 直接更改 state 不会触发 rerender

#### 16.1 原因

- 必须通过 setState 方法告诉 React state 已经改变

#### 16.2 异步更新

在组件 lifecycle 周期 / react 合成事件中, setState 是异步

```ts
this.state.message = "HelloWorld"
change() {
	setState({
		message: "Hello"
	}); // setState需要时间, 所以先执行下一行了
	console.log(this.state.message); // Hello World
}
```

#### 16.2 同步更新

在`setTimeout` / 原生 dom 事件中, setState 是同步

```ts
this.state.message = "HelloWorld"
// ex1
change() {
	setState({
		message: "Hello"
	}, () => {
		console.log(this.state.message); // Hello
	});
}

// ex2
change() {
	setTimeout(() => {
		setState({
			message: "Hello"
		});
		console.log(this.state.message); // Hello
	}, 0)
}

// ex3
componentDidMount() {
	document.getElementById("btn").addEventListener('click', () => {
		this.setState({
			message: "Hello"
		});
		console.log(this.state.message); // Hello
	})
}
```

#### 16.3 批量更新 (异步)

```ts
handleClick = () => {
  setState(count + 1);
  console.log(count); // 1

  setState(count + 1);
  console.log(count); // 1

  setState(count + 1);
  console.log(count); // 1
};
```

- 打印 1, count 值更新为 2
- 对同一个值进行多次 setState, 它的批量更新策略会自动覆盖, 取最后一次执行结果

### 17. Render

- 编写 jsx 后, 被 babel 编译后转化成 js 格式
- `createElement`制造的 element 是虚拟 DOM tree 的节点
  - 3 个参数: type (标签), attributes (标签属性), children
- 最终虚拟 DOM 渲染成真实 DOM
  - React 把 render 返回的树和旧版本的树进行比较 diff, 决定如何更新 DOM
- Fiber
  - 把同步变成异步渲染 `async render`
  - 任务拆解: 把大任务拆解到小任务
  - 可打断: `render`可以被打断, 并且 by priority 防止饿死

```jsx
return (
  <div className="cn">
    <Header> hello </Header>
    <div> start </div>
    Right Reserve
  </div>
);

// after compiling to js
return React.createElement(
  "div",
  {
    className: "cn",
  },
  React.createElement("Header", null, "hello"),
  React.createElement("div", null, "start"),
  "Right Reserve"
);
```

#### 17.2 Trigger

- class component: 更新 state by using `setState`
  - 触发 render
- function component: 通过`useState` hook 修改状态
  - 判断 state 有无改变, 触发 render

### 18. Real DOM & Virtual DOM

- Real DOM
  - 频繁重排和重绘
  - 损耗: RDOM (CRUD + 排版 + 重绘)
- Virtual DOM
  - 不会进行排版 (layout) & 重绘 (repaint)
  - 损耗: VDOM (CRUD) + RDOM (diff CRUD + 排版 + 重绘)
  - 一次流程 - store **ALL** diff to local js object, then attach to DOM
  - 跨平台
  - 首次渲染大量 DOM, 慢

#### 18.1 Real DOM

- 文件对象类型, 结构化文本的抽象, 在页面渲染出的每一个节点都是真实 DOM 结构
  - `<div id='root'> </div>`

#### 18.2 Virtual DOM

- JavaScript Object: 对 DOM 的描述
- PropertyMap: virtual to real DOM
- `ReactDOM.render`: insert virtual DOM node to read node & complete render
  - `ReactDOM.render(<h1>hello</h1>, document.getElementById("root"));`
  - 把 h1 标签渲染到 root 节点
- JSX
  - compiled to VDOM by Babel, return a VDOM
  - `VDOM = React.createElement()`

#### 18.3 JSX -> Real DOM

1. Babel 转换 jsx 代码:

   - 标签首字母:
     - 小写: `div` - 原生 DOM, `createElement(string, ...)`
     - 大写: `Hello` - 自定义 DOM, `createElement(object, ...)`

   ```jsx
   <div>
   	<img src='image.png' className='profile' />
   	<Hello />
   </div>

   React.createElement(
   	"div",
   	null,
   	React.createElement("img, {
   		src: "image.png",
   		className: "profile"
   	}),
   	React.createElement(Hello, null)
   );
   ```

2. `createElement()` 对信息进行处理, 构成 VDOM object
3. `RenderDOM.render(element, container, [callback])`挂载
   - `RenderDOM.render(<App />, document.getElementById("root"))`
   - `container`: 真实 DOM 节点
   - `callback`: 完成渲染并将内容挂到 DOM 上后, 自动执行
4. First time: update ALL in container
5. 使用`React diff`进行高效更新 Real DOM
   - callback: 组件被渲染 / 更新后执行
   - 转换成 RDOM

#### 18.4 加载过程

1. 发起请求
   1. 浏览器输入 url
   2. DNS 域名解析
   3. 找到 IP
   4. 向服务器发送请求
2. 接收请求
   1. decode binary to html - index.html
   2. 使用 HTML 解析器构建 DOM Tree
      1. 从 Document 开始, 解析标签 - `</div>`
      2. 生成 node - `HTMLDivElement`
      3. 生成 Tree, 和 html 一一对应
      - 遇到 link 的外部 css, 加载 CSS **(新线程)**
        - 解析 -> 构建 CSSOM Tree
        - CSSStyleSheet -> CSSRule -> Selector & Declaration(语法, 描述)
      - 遇到 script: 先执行 js 内容, 完成后构建 DOM (不开新线程) - 这就是为什么 script 加到底部
        - async 加载(fetch)完成后立即执行 (execution)，因此可能会阻塞 DOM 解析
        - defer 加载(fetch)完成后延迟到 DOM 解析完成后才会执行(execution)
   3. 构建 Render Tree = DOM Tree + CSSOM Tree
   4. 布局 layout & 绘制 paint:
      - 计算大小, 距离, 坐标..., 通过 UI 绘制
      - _reflow 回流_: 当元素属性改变 并且 影响布局 (width, height, margins...) = 刷新页面
      - _repaint 重绘_: 当元素属性发生改变 不影响布局 (color, font...) = 动态更新

### 19. Promises

- Handle asynchronous operations
  - asynchronous: not block main thread of execution
- An object: represents the result of an asynchronous operation.
  - look like synchronous code, more readable and maintainable.
  - `Promise.resolve(value)`: returns a Promise that is **resolved** with the given value.
  - `Promise.reject(reason)`: returns a Promise that is **rejected** with the given reason.
  - `Promise.all(iterable)`: returns a Promise that is resolved when **all** Promises have resolved, or rejected when any of them rejects.
  - `Promise.race(iterable)`: returns a Promise that is resolved or rejected as soon as one of the Promises resolves or rejects.

### 20. FIBER

React 和浏览器共同合作, 实现渲染

#### 20.1 Stack Reconciler - 旧

- 互斥: JavaScript 和页面渲染两个 process 无法同时执行, 无法中断 React 渲染组件
- 同步更新: js process 持续占用, 直到 VDOM 计算完成, 才可以渲染 (大组件卡顿)

#### 20.2 Fiber - 重写后的 Stack Reconciler

JavaScript object - element data, 更新操作队列, 类型....

- Doubly Linked List
  - child, sibling, return (父节点)
  - **可中断, 可恢复, 异步优先级更新**
  - 代表“任务调度单”
- 步骤
  1.  一个组件/DOM = 一个 Fiber Node
  2.  Traverse nodes
      - 异步: use `requestIdleCallback()`:
        - 检查有没有时间执行, 如果时间不够就中断, 浏览器空闲了再恢复执行
        - Priorities: 高可以中断低任务, 不需要重新执行
  3.  构建新的 Fiber Tree
  4.  one-time Commit: 提交更新 DOM

#### 20.3 Fiber 双队列 (Current vs Work-In-Progress Tree)

- Fiber changed DOM Diff Tree to Linked List (像是带指针的树)
- 1 DOM = 2 Fiber node (in current & work-in-progress)
  - Current fiber Tree (已渲染的)
  - Work-in-progress fiber tree (正在构建新的)
    - 中断时, 记住位置，恢复时继续遍历

### 21. Key & Diff

`Each child in a list should have a unique "key" prop.`

- diff 算法
  - 用 key 判断元素是新创建的还是被移动的, 减少不必要的元素渲染
- 优化
  - 要: 唯一且稳定的 key
  - 不要: no random key value
  - 不要: index 作为 key 有可能不会对性能产生优化

#### Diff

- 暴力解 -> 优化: O(n^3) -> O(n)
- 优化后的三个层级
  1.  tree 层级策略
      - 对相同层级的节点比较, Delete, Create, NO Move
  2.  component 层级策略
      - 比较同类文件, 删除不同类 & 创建新 component
  3.  element 层级策略
      - Check `key` between nodes which in same level
      - Insert, Move, Remove by `key`
- 渲染列表
  - 不使用 key 会更好
  - 插入最前面会导致整个 component 重新渲染

### 23. Hooks

- hooks 属于 functional programming, 包裹功能在函数中, 提高 code readability
- 自定义 hook 有助于封装我们的功能

#### 23.1 useState

```ts
const [count, setCount] = useState(0);
```

#### 23.2 useEffect

**Each Effect is a separate and independent synchronization process.**

- A component *mounts* when it’s added to the screen
- A component *updates* when it receives new props or state, usually in response to an interaction
- A component *unmounts* when it’s removed from the screen
- example
  - components
    1. `ChatRoom` mounted with `roomId` set to `"general"`
    2. `ChatRoom` updated with `roomId` set to `"travel"`
    3. `ChatRoom` updated with `roomId` set to `"music"`
    4. `ChatRoom` unmounted
  - Effect:
    1. Your Effect connected to the `"general"` room
    2. Your Effect disconnected from the `"general"` room and connected to the `"travel"` room
    3. Your Effect disconnected from the `"travel"` room and connected to the `"music"` room
    4. Your Effect disconnected from the `"music"` room

```ts
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();

  return () => {
    // clean up function to stop synchronizing effect (dependencies changes)
    connection.disconnect();
  };
}, [roomId]);
```

#### 23.3 createContext

1. Create the context

   ```jsx
   export const PortfolioContext = createContext<{
   	data: AllDataType;
   	$locale: LocalType;
   	onLocaleChange: (locale: LocalType) => void;
   }>({
   	$locale: "en-US",
   	data: allData,
   	onLocaleChange: () => {},
   });

   export const PortfolioProvider: FC<{ children: ReactNode }> = ({
   	children,
   }) => {
   	const [$locale, setLocale] = useState<LocalType>("en-US");
   	const onLocaleChange = (locale: LocalType) => {
   		setLocale(locale);
   	};
   	return (
   	<PortfolioContext.Provider
   		value={{ data: allData, $locale, onLocaleChange }}>
   		{children}
   	</PortfolioContext.Provider>
   	);
   };
   ```

2. Provide the context (in parent)

   - **Wrap children with a context provider** to provide the `PortfolioProvider`

   ```ts
   import { LevelContext } from "./LevelContext.js";

   export default function Section({ level, children }) {
     return (
       <section className="section">
         <PortfolioProvider value={level}>{children}</PortfolioProvider>
       </section>
     );
   }
   ```

3. Use the context (in children)
   - `useContext` tells React that the component X wants to read the `LevelContext`.
     `const level = useContext(LevelContext);`

#### 23.4 useReducer

高级版本的 useState

- 管理复杂的状态逻辑
- 支持合并多个更新操作，从而减少不必要的重新渲染

1. interface

   ```ts
   export interface CacheState {
     tab: string;
     busy: boolean;
   }

   export type CacheAction =
     | { type: "SET"; tab: string }
     | { type: "ADD"; busy?: boolean };
   ```

2. implementation
   - 接收新的状态&动作, 这样就可以更新状态

```ts
export const reducer = (state: CacheState, action: CacheAction): CacheState => {
	switch (action.type) {
		case "SET":
			return { ...state, tab: action.tab }
		case "ADD":
			return { ...state, busy: action.busy }
```

3. usage
   - useReducer
     - Argument: reducer function, initial state
     - Return: newState, dispatch function (to update state & re-render)

```ts
const [{ tab, busy }, dispatch] = useReducer(reducer, {
  tab: "Sample",
  busy: true,
});

dispatch({ type: "SET", tab: "newTab" });
```

#### 23.5 useMemo

`useMemo` cache the result of a calculation between re-renders.

- `calculateValue`: the Value to cache. Executes for the initial render, then wait until dependencies changes
- `dependencies`: The list of all reactive values referenced inside body.

```ts
import { useMemo } from "react";
function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
}
```

#### 23.6 useCallback

for referential equality problem

- 防止因为父组件的非相关渲染而导致的子组件的不必要重新渲染。
- 想象这个场景：
  - 子组件接受一个父组件传递的函数作为 prop
  - 如果父组件重新渲染，而且这个函数是在父组件的函数体内定义的，那么每次父组件渲染时，都会为子组件传递一个新的函数实例。
  - 这可能会导致子组件不必要地重新渲染，即使该函数的实际内容没有任何变化。
- 使用场景
  - 子组件的性能优化：当你将函数作为 prop 传递给已经通过 React.memo 进行优化的子组件时，使用 useCallback 可以确保子组件不会因为父组件中的函数重建而进行不必要的重新渲染。
  - Hook 依赖：如果你正在传递的函数会被用作其他 Hook（例如 useEffect）的依赖时，使用 useCallback 可确保函数的稳定性，从而避免不必要的副作用的执行。
  - 复杂计算与频繁的重新渲染：在应用涉及很多细粒度的交互，如绘图应用或其它需要大量操作和反馈的场景，使用 useCallback 可以避免因频繁的渲染而导致的性能问题。
  - 在这个情况中, 除去 number 变化, 重建它是没有意义的. 只有在 number 变化的时候, 才会重建
  ```ts
  const [number, setNumber] = useState(1);
  const getItems = () => {
    return [number, number + 1];
  };
  ```
  ```ts
  const getItems = useCallback(() => {
    return [number, number + 1];
  }, [number]);
  ```
- Note
  - useCallback 适用对象是一个 function, 一个被 execute 之后才会 return 的 function
    - `useCallback(fn, deps)`
  - 我们用 useCallback 把函数包起来之后，在父组件中只有当 deps 变化的时候，才会创建新的 getItems 实例，子组件才会跟着 reRender
  - useMemo 保存的是一个 object, 一个已经被 return 了的 function
    - `useMemo(() => fn, deps)`

#### 23.7 useRef

###### Ref

ref 是一种用于获取对特定元素或组件实例的引用的方式: React.createRef()

- 它通常用于访问特定元素，如输入框或自定义组件的实例，或者访问组件上的方法以调用该方法与组件进行交互。

###### Hook

Useful to persist values across renders - 在 current 属性中保存一个可变的”盒子“
`const refContainer = useRef(initialValue);`

- `refContainer.current = initialValue`
- 更新 current 不会 re-render

#### 23.8 `useHistory`

```ts
const history = useHistory();
return <button onClick={() => history.push("/")} />;
```

#### 23.9 `useParams`

```ts
const {name} = useParams();
return (
	{name !== "name" ? <Redirect to="/" /> : null}
)
```

#### 23.10 `useLocation`

当前 URL 的 location object

```ts
const { pathname } = useLocation();
return <h1>{pathname}</h1>;
```

### 24. 提高效率

- 搭配 React 和 Diff, 尽量做到 DOM 对小粒度的更新
  - 尽量拆分组件
- 当 parent 渲染, 避免无谓的 child 渲染
- 共可以从三个层面分析 - 代码层, 工程层, 框架机制层

#### 24.1 `bool shouldComponentUpdate()`

React LifeCycle 中, 通过深比较 state 和 props 确定是否 re-render

#### 24.2 `PureComponent`

通过 state 和 props 的浅比较 - `shallowEqual`

#### 24.3 `React.Memo`

- 缓存组件的渲染
- 只能用于 function component
- 如果需要深层比较, 传入两个参数

```ts
function Button(props) {}

export default memo(Buttom) // 浅
export default memo(Buttom,
					(prevProps, nextProps) => prevProps === nextProps)) // 深
```

#### 24.4 Avoid 内联函数

render 时需要重新创建新的 function instance

```jsx
// slow
<input
  onClick={(e) => {
    this.setState({ inputValue: e.target.value });
  }}
/>;

// fast
f = (e) => {
  this.setState({ inputValue: e.target.value });
};
<input onClick={this.f} />;
```

#### 24.5 Fragments 避免额外标记

如果需要 return 多个标签, 使用 Fragments 不会引入额外标记

```jsx
return (
  <>
    <h1 />
    <h1 />
  </>
);
```

#### 24.6 Put `bind` in Constructor

#### 24.7 `Immutable`

无需进行深度比较, 提升`shouldComponentUpdate`效率

#### 24.8 懒加载

动态加载: `Suspense`, `lazy`实现代码拆分

```jsx
const comp = React.lazy(() => import("./"));
export const asyncComp = (props) => (
  <React.Suspense>
    <comp {...props} />
  </React.Suspense>
);
```

#### 24.9 服务端渲染

1. 后端将 Root component 转换成 string, 传输到前端
2. 前端使用 render 生成 html

### 26. React Router

- 实现 SPA 应用
- 无刷新的条件下切换显示不同页面
- URL 改变时, 页面根据 URL 变化, 但是页面**不会刷新**

#### 26.1 方法

- `<Route path="/" component={<渲染组件 />} render={() => <渲染内容/>} exact/>`
- `<Redirect to="/" />`
- switch - 匹配第一个组件时使用, 之后不会匹配
  - `<Switch> <Route /> <Route /> </Switch>`
- useHistory
- useParams
- useLocation

#### 26.2 参数传递

- 动态路由 `<Route path="/:id"/>`
  - 可以匹配 `/abc`, `/123`
- search 传递参数

  ```ts
  <NavLink to="/detail2?name=why&age=18 />

  console.log(props.location.search)
  ```

- to 传入对象

  ```ts
  <NavLink
    to={{
      pathname: "",
      query: { name: "", age: 0 },
      state: { height: 1, address: "" },
      search: "?apikey=123",
    }}
  />;

  console.log(props.location);
  ```

#### 26.3 Hash 模式 - HashRouter

改变 hash 不会导致浏览器向服务器发请求, 也不会刷新页面

- 触发`window.hashchange` event, 操作 DOM 来模拟页面跳转

#### 26.4 History 模式 - BrowserRouter

通过`BrowserRouter`传入的值: props.path, context.pathname 进行匹配, 决定渲染文件

### 27. Redux

- 数据状态管理 - State Container
- UI 组件 --(dispatch: action)--> Store --(reducer)--> 生成新 State 并存储

#### 27.1 `react-redux`

- 统一管理: 通过 redux 存储 state 到 store, 使用 dispatch action 给 store
- `mapStateToProps`: 当前组件需要什么 Redux 的 state 来映射到它的 props 里。
- `mapDispatchToProps`: 当前组件需要什么 dispatch 操作来触发状态更新。

```jsx
// Provider: 注入
<Provider store = {store}>
	<App />
</Provider>

// connection: 使用getState, dispatch
connect(mapStateToProps, mapDispatchToProps)(MyComponent);

// 映射数据
const mapStateToProps = (state) => {
	return {
		prop: state.xxx // 映射
	}
}

const mapDispatchToProps = (dispatch) => {
	return {
		onClick: () => {
			dispatch({
				type: 'increatment'
			})
		}
	}
}

// 使用
class Foo extends Component {
	constructor(props) {...}

	render() {
		return (
			<div>this.props.foo</div> // 渲染的是redux中的数据
			<button> onClick={this.props.onClick} />
		)
	}
}
Foo = connect()(Foo);
export default Foo;
```

#### 27.2 Redux Middleware

- dispatch & 分发 action 的过程: 异步, 错误处理, 监控...
- Action -> Mid, Mid, Mid -> Store (Reducer) -> UI
- 共享: 应用系统和系统软件之间, 使用系统软件的功能衔接应用系统的应用
- 中间件: `applyMiddlewares`注册, 把中间件组成数组, 依次执行
  - redux-thunk: 异步操作
  - redux-logger: log 记录 `logger = createLogger()`
  - `const store = createStore(reducer, applyMiddleware(thunk, logger));`

#### 27.3 核心原则

##### 1. 单一数据源 (`State`, `Store`)

任何地方都可以获取 state

- State example: DomainState: 服务器端数据. UI State: UI 状态

##### 2. 只能通过`action (dispatch)`来修改 state

decouple 显示&修改

- Action: (plain object)，至少包含 `type`，用来表示将要发生什么
- 维护, 可控: 将修改状态的逻辑都放到 action/reducer 中

##### 3. 使用`reducer`修改 state

- 本质: `(prevState, action) => newState`
- 根据 action 的类型和附带的数据, 计算出新的状态

### 28. Immutable

- Persistent data structure
  - 更新 immutable 会返回新的 immutable object
  - structural sharing: new object 会尽可能不浪费内存
    - no deep copy
    - 共享未修改的节点
- 性能优化, 减少渲染次数
  - `shouldComponentUpdate()`通过 is 比较 Immutable, not deep compare
- Collection, List, Map, Set

```ts
import { Map, is } from "immutable";

const m1 = Map({ a: 1 });
const m2 = Map({ a: 1 });
m1 === m2; // false
Object.is(m1, m2); // false
is(m1, m2); // true, 比较两个object
```

### 29. React Server-side rendering (SSR)

- 服务端运行 React 生成 HTML -> 发送到浏览器 -> 加载 JS 文件 (前端绑定事件&状态)
- 加速首屏加载

#### Server

```jsx
export const App = () => (
  <div>
    <h1>Hello, React SSR!</h1>
    <p>This content is rendered on the server.</p>
  </div>
);

app.get("*", (req, res) => {
  // Render the React component to a string
  const appString = ReactDOMServer.renderToString(<App />);

  // Embed the rendered app into a basic HTML template
  const html = `
    <!DOCTYPE html>
    <html>
      <head>
        <title>React SSR Example</title>
      </head>
      <body>
        <div id="root">${appString}</div>
        <!-- Your bundled client-side code to hydrate the app -->
        <script src="/bundle.js"></script>
      </body>
    </html>
  `;
  res.send(html);
});
```

#### Client

bundler like `Webpack` to bundle client-side JavaScript (`client.js`) into a file `bundle.js` from the `public` directory

- attaching event listeners, and making the app interactive
- Recursively analyze dependency graph from the entry point
  - transform JavaScript and JSX syntax into code that can run in all target browsers.
  - browsers do not natively understand JSX or some newer JavaScript features.

```jsx
// Client-Side Hydration: Hydrate the app to attach event listeners, attach the React tree to the existing HTML
// client.js
ReactDOM.hydrate(<App />, document.getElementById("root"));

// Bundling the Client Code
// bundle.js
const path = require("path");

module.exports = {
  // The entry point for the client-side code
  entry: "./client.js",

  // The output configuration where the bundle will be placed
  output: {
    path: path.resolve(__dirname, "public"),
    filename: "bundle.js",
  },

  // Module rules to define how different file types are handled
  module: {
    rules: [
      {
        test: /\.jsx?$/, // Targets both .js and .jsx files
        exclude: /node_modules/, // Skips transformation for node_modules
        use: "babel-loader", // Uses Babel to transform modern JS/JSX code
      },
      // Additional rules can be added here for handling CSS, images, etc.
    ],
  },

  // Resolve file extensions so that imports don't need to include them explicitly
  resolve: {
    extensions: [".js", ".jsx"],
  },

  // Mode can be 'development' or 'production'
  mode: "development", // Change to 'production' for production builds
};
```
