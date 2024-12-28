---
title: React
categories:
  - Front-End
  - React
---

## React

#### 底层逻辑

##### JSX

总结:

- ![[Screenshot 2024-08-23 at 11.52.58 AM.png|400]]
- 用于描述 component 内容
- 是 JS 的语法扩展, 类似模版语言但是充分具备 JS 的能力
  - 被编译为`React Element : createElement(type, config, children)`
  - 因为是扩展: 需要用 Babel 使 JSX 支持浏览器
- Babel
  - 工具链, 将 ECMAScript 代码转换为向后兼容的 JS 语法来防止不兼容
- createElement 参数中介 - 数据格式化 => `ReactElement`:
  - ![[Screenshot 2024-08-23 at 11.47.38 AM.png|400]]
  - ![[Screenshot 2024-08-23 at 11.48.03 AM.png|400]]
- `ReactElement`:
  - 组装
  - ![[Screenshot 2024-08-23 at 11.49.34 AM.png|400]]
  - 被装到虚拟 DOM, 然后`ReactDOM.render` 来填补到真实 DOM: `ReactDOM.render(element, container, [callbck])`

##### React 生命周期

- 灵魂 `render`
  - 渲染工作流: 从组件数据改变到组件实际更新发生的过程
  - `render`不会操作真实 DOM, 只是把需要渲染的内容返回出来
  - `componentWillReceiveProps()`由父组件的更新触发的
  - `shouldComponentUpdate()`的返回值决定之后的生命周期 & 是否进行 re-render
  - ![[Screenshot 2024-08-23 at 12.51.43 PM.png|500]]
- 躯干
- Fiber
  - 把同步变成异步渲染 `async render`
  - 任务拆解: 把大任务拆解到小任务
  - 可打断: `render`可以被打断, 并且 by priority 防止饿死

#### Promises

To handle asynchronous operations

- asynchronous: they don’t block the main thread of execution
  A Promise is an object that represents the eventual result of an asynchronous operation. It has three states: **_pending_, *resolved* (_fulfilled_), and *rejected*.** The core idea is to make asynchronous code look and feel more like synchronous code, making it more readable and maintainable.
- `Promise.resolve(value)`:This method returns a Promise that is resolved with the given `value`.
- `Promise.reject(reason)`:Returns a Promise that is rejected with the given `reason`.
- `Promise.all(iterable)`:This method returns a Promise that is resolved when all Promises in the iterable have resolved, or rejected when any of them rejects.
- `Promise.race(iterable)`:Returns a Promise that is resolved or rejected as soon as one of the Promises in the iterable resolves or rejects.

#### Axios

make HTTP requests from node.js or XMLHttpRequests from the browser and it supports the Promise API that is native to JS ES6. It can be used intercept HTTP requests and responses and enables client-side protection against XSRF. It also has the ability to cancel requests.

```javascript
axios
  .get("url")
  .then((response) => {
    // Code for handling the response
  })
  .catch((error) => {
    // Code for handling the error
  });
```

#### Hooks

##### useState

```javascript
const [count, setCount] = useState(0);
```

##### useEffect

**Each Effect in your code should represent a separate and independent synchronization process.**

- A component *mounts* when it’s added to the screen.
- A component *updates* when it receives new props or state, usually in response to an interaction.
- A component *unmounts* when it’s removed from the screen.
- example
  - components:
    1. `ChatRoom` mounted with `roomId` set to `"general"`
    2. `ChatRoom` updated with `roomId` set to `"travel"`
    3. `ChatRoom` updated with `roomId` set to `"music"`
    4. `ChatRoom` unmounted
  - Effect:
    1. Your Effect connected to the `"general"` room
    2. Your Effect disconnected from the `"general"` room and connected to the `"travel"` room
    3. Your Effect disconnected from the `"travel"` room and connected to the `"music"` room
    4. Your Effect disconnected from the `"music"` room

```javascript
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);

  connection.connect();

  return () => {
    // clean up function to stop synchronizing effect (dependencies changes)
    connection.disconnect();
  };
}, [roomId]);
```

##### createContext

1. Create the context
   ```javascript
   export const PortfolioContext = createContext<{
   	data: AllDataType;
   	$locale: LocalType;
   	onLocaleChange: (locale: LocalType) => void;
   }>({
   	$locale: "en-US",
   	data: allData,
   	onLocaleChange: () => {},
   });
   ```
   ```javascript
   export const PortfolioProvider: FC<{ children: ReactNode }> = ({
     children,
   }) => {
     const [$locale, setLocale] = useState < LocalType > "en-US";
     const onLocaleChange = (locale: LocalType) => {
       setLocale(locale);
     };
     return (
       <PortfolioContext.Provider
         value={{ data: allData, $locale, onLocaleChange }}
       >
         {children}
       </PortfolioContext.Provider>
     );
   };
   ```
2. Provide the context (in parent)

   - **Wrap children with a context provider** to provide the `PortfolioProvider`

   ```javascript
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

##### useReducer

高级版本的 useState

- 管理复杂的状态逻辑
- 支持合并多个更新操作，从而减少不必要的重新渲染

1. interface

   ```javascript
   export interface CacheState {
   	tab: string
   	busy: boolean
   }

   export type CacheAction =
   | { type: "SET"; tab: string }
   | { type: "ADD"; busy?: boolean }
   ```

2. implementation
   - 接收新的状态&动作, 这样就可以更新状态

```javascript
export const reducer = (state: CacheState, action: CacheAction): CacheState => {
  switch (action.type) {
    case "SET":
      return { ...state, tab: action.tab };
    case "ADD":
      return { ...state, busy: action.busy };
  }
};
```

3. usage
   - useReducer
     - Argument: reducer function, initial state
     - Return: newState, dispatch function (to update state & re-render)

```javascript
const [{ tab, busy }, dispatch] = useReducer(reducer, {
  tab: "Sample",
  busy: true,
});

dispatch({ type: "SET", tab: "newTab" });
```

##### useMemo

`useMemo` cache the result of a calculation between re-renders.

- `calculateValue`: the Value to cache. Executes for the initial render, then wait until dependencies changes
- `dependencies`: The list of all reactive values referenced inside body.

```javascript
import { useMemo } from "react";
function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
}
```

##### useCallback

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
  ```javascript
  const [number, setNumber] = useState(1);
  const getItems = () => {
    return [number, number + 1];
  };
  ```
  ```javascript
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

##### useRef

###### Ref

ref 是一种用于获取对特定元素或组件实例的引用的方式: React.createRef()

- 它通常用于访问特定元素，如输入框或自定义组件的实例，或者访问组件上的方法以调用该方法与组件进行交互。

###### Hook

Useful to persist values across renders - 在 current 属性中保存一个可变的”盒子“
`const refContainer = useRef(initialValue);`

- `refContainer.current = initialValue`
- 更新 current 不会 re-render
