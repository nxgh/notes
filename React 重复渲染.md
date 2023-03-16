<!--
tags:
  - react
search:
  - useCallback
  - useMemo
  - memo
  - render
  - rerender
 -->

- re-render 的原因，state?props?context?
- 如何避免：useCallback、useMemo、memo
- 关于 ref

## 更新与重新渲染

一般来说

- 「更新」指 `render`、`Reconcilation` 和 `Commit` 整个过程
- 「重新渲染」，指的是 `render` 阶段。

## React 中的 rerender

> 在不使用 `this.forceUpdate()` 的情况下，状态改变是 React 重新渲染的唯一触发条件。

状态改变在 React 中表现为:

1. `useState` 的回调或 `useEffect`
2. 父组件 rerender
3. `context` provider 中的值改变导致所有依赖这个值的组件 rerender

未被 `memo` 包裹的组件 `props` 变化并不是 rerender 的根因，它是父组件 rerender 的一种表现形式

## 当一个状态发生改变时，React 会重新渲染整个组件树吗？

并不会，React 只会更新依赖这个状态的组件，以及它的子组件。

在下边的 🌰 中，`Bar` 的状态发生了改变，但是 `Foo` 并没有 rerender。

在 React 中，数据是自上而下单向传递的(「单向数据流」, The Data Flows Down)

同时也能看到，`Child` 组件依然发生了更新，即使它没有依赖任何 props, 即：

当一个组件更新时，react 会更新这个组件的所有子组件，即使这些子组件没有依赖任何 props。

<!-- <Preview.React> -->

```tsx App.tsx
function Foo() {
  return <div style={{ border: '1px solid #eee' }}>Foo: {Date.now()}</div>
}
function Bar() {
  const [count, setCount] = React.useState(0)
  return (
    <div style={{ border: '1px solid #eee' }}>
      Bar: {Date.now()} <button onClick={() => setCount(count + 1)}>+</button>
      <Child />
    </div>
  )
}

function Child() {
  return <div>Child: {Date.now()}</div>
}

export function Demo() {
  return (
    <div style={{ border: '1px solid #eee' }}>
      <Foo />
      <Bar />
    </div>
  )
}
```

<!-- </Preview.React> -->

> 理想状态下，每一个 react 组件都应该是一个纯函数，它的输入是 props，输出是 UI，它不应该有任何副作用，也不应该依赖外部的状态。
> 可实际上，我们经常会写出一个类似上边例子的组件，它的状态是在组件内部维护的。

## memo

`memo` 是一个高阶组件，它可以用来避免组件的重新渲染。

还是上方的 🌰，我们可以使用 `memo` 来避免 `Child` 组件的重新渲染。

<!-- <Preview.React> -->

```tsx App.tsx
function Foo() {
  return <div style={{ border: '1px solid #eee' }}>Foo: {Date.now()}</div>
}
function Bar() {
  const [count, setCount] = React.useState(0)
  return (
    <div style={{ border: '1px solid #eee' }}>
      Bar: {Date.now()} <button onClick={() => setCount(count + 1)}>+</button>
      <Child />
    </div>
  )
}

const Child = React.memo(() => {
  return <div>Child: {Date.now()}</div>
})

export function Demo() {
  return (
    <div style={{ border: '1px solid #eee' }}>
      <Foo />
      <Bar />
    </div>
  )
}
```

<!-- </Preview.React> -->

### 为什么 React 不默认 `memo` 所有组件呢？

React 组件更新的开销没有想象中的那么大

如果一个组件接受很多复杂的 prop，有可能渲染这个组件并对比 Virtual DOM 的性能开销甚至小于等于浅比较所有 prop 的开销。

绝大部分时候，React 是足够快的。因此，只有当一个 纯组件 有大量纯的子组件、或者这个 纯组件 内部有很多复杂计算时，我们才需要将其包裹在 memo 中。

另外一个 React 默认不 memo 所有组件的原因是：让 React 在 Runtime 中判断子组件的全部依赖、以跳过子组件的不必要更新，是非常困难、非常不现实的。计算子组件依赖的最好时机是编译期间

## useCallback & useMemo

除了 `memo`，还有 `useCallback` 和 `useMemo` 可以用来避免组件的重新渲染。

- `useCallback` 一般需要配合 `memo` 使用。
- `useCallback` 一般用来避免函数的重新创建。
- `useMemo` 一般用来避免值的重新计算。
- `useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`

在实践中，如果想通过 缓存 `props` 的方式来避免组件的重新渲染，需要保证:

1. 所有传递给子组件的 `props` 的引用都不变，(比如通过 `useMemo`)
2. 子组件使用 `React.memo` 或者 `React.PureComponent`

## useCallback

1.  `useCallback` 可以用来解决创建函数造成的性能问题?

并不能，单从组件角度来看，`useCallback` 只会更慢，因为函数无论如何都会被创建，还增加了内部对于依赖变化的检测。

`useCallback` 的真正目的还是在于缓存了每次渲染时 `inline callback` 的实例， 这样方便配合上子组件的 `React.memo`(或者`shouldComponentUpdate` ) 起到减少不必要的渲染的作用。

`useCallback` 需要与 `React.memo` 配合使用，否则没有任何意义。无意义的浅比较也会消耗性能。

## useMemo

`useMemo` 可以直接缓存组件的返回值, 只有当依赖发生变化时，才会重新 render。

```tsx
function App({ todo, tab }) {
  const visibleTodo = useMemo(() => filterTodos(todos, tab), [todos, tab])
  return useMemo(() => <Todo data={visibleTodo} />, [visibleTodo])
}

function Todo({ data }) {
  return <p>{data}</p>
}
```

## 如何避免 Context 导致的重复渲染

众所周知，当 Context 的 value 发生改变的时候，所有 `<Context.Provider />` 的子组件都会更新, 这本质上还是因为状态的改变

<!-- <Preview.React> -->

```tsx App.tsx
const Context = React.createContext()

function Header() {
  return <h1>Hello World{Date.now()}</h1>
}
function Content() {
  const { theme, setTheme } = React.useContext(Context)
  return (
    <div>
      <p style={{ color: theme === 'red' ? 'red' : '#000' }}>Lorem {Date.now()}.</p>
      <button onClick={() => setTheme('dark')}>Dark Theme</button>
      <button onClick={() => setTheme('red')}>Red Theme</button>
    </div>
  )
}
function Footer() {
  return <div>{Date.now()}</div>
}
export function App() {
  const [theme, setTheme] = React.useState('dark')
  return (
    <div className='App'>
      <Context.Provider value={{ theme, setTheme }}>
        <Header />
        <Content />
        <Footer />
      </Context.Provider>
    </div>
  )
}
```

<!-- </Preview.React> -->

当 `Context.Provider` 重新渲染的时候，它所有的子组件都被重新渲染了, 那么我们可以将 `Context.Provider` 抽离出来，这样就不会导致 `App` 组件的重新渲染了

<!-- <Preview.React> -->

```tsx App.tsx
const Context = React.createContext()

function Header() {
  return <h1>Hello World{Date.now()}</h1>
}
function Content() {
  const { theme, setTheme } = React.useContext(Context)
  return (
    <div>
      <p style={{ color: theme === 'red' ? 'red' : '#000' }}>Lorem {Date.now()}.</p>
      <button onClick={() => setTheme('dark')}>Dark Theme</button>
      <button onClick={() => setTheme('red')}>Red Theme</button>
    </div>
  )
}
function Footer() {
  return <div>{Date.now()}</div>
}
function Provider(props) {
  const [theme, setTheme] = React.useState('dark')
  return <Context.Provider value={{ theme, setTheme }}>{props.children}</Context.Provider>
}
export function App() {
  return (
    <div className='App'>
      <Provider>
        <Header />
        <Content />
        <Footer />
      </Provider>
    </div>
  )
}
```

<!-- </Preview.React> -->

### 避免由多个不同状态更新导致的重复渲染

当存在多个不同的状态更新时，订阅上下文的组件都将重复渲染，而不管这些状态更新是否需要。

这个例子中，无论是 `count` 还是 `theme` 发生改变，订阅上下文的子组件都将重新渲染

<!-- <Preview.React> -->

```tsx App.tsx
const Context = React.createContext()

function Header() {
  const { count, setCount } = React.useContext(Context)
  return (
    <>
      <h1>
        count: {count} {Date.now()}
      </h1>
      <button onClick={() => setCount(count => (count += 1))}>Red Theme</button>
    </>
  )
}

function Content() {
  const { theme, setTheme } = React.useContext(Context)
  return (
    <div>
      <p style={{ color: theme === 'red' ? 'red' : '#000' }}>Lorem {Date.now()}.</p>
      <button onClick={() => setTheme('dark')}>Dark Theme</button>
      <button onClick={() => setTheme('red')}>Red Theme</button>
    </div>
  )
}

function Provider(props) {
  const [theme, setTheme] = React.useState('dark')
  const [count, setCount] = React.useState(0)
  return <Context.Provider value={{ theme, setTheme, count, setCount }}>{props.children}</Context.Provider>
}
export function App() {
  return (
    <div className='App'>
      <Provider>
        <Header />
        <Content />
      </Provider>
    </div>
  )
}
```

<!-- </Preview.React> -->

一个常见的解决方案是使用嵌套上下文

<!-- <Preview.React> -->

```tsx App.tsx focus=1,2,28:35,39,40,43,44
const CountContext = React.createContext()
const ThemeContext = React.createContext()

function Header() {
  const { count, setCount } = React.useContext(CountContext)
  return (
    <>
      <h1>
        count: {count} {Date.now()}
      </h1>
      <button onClick={() => setCount(count => (count += 1))}>Red Theme</button>
    </>
  )
}

function Content() {
  const { theme, setTheme } = React.useContext(ThemeContext)
  return (
    <div>
      <p style={{ color: theme === 'red' ? 'red' : '#000' }}>Lorem {Date.now()}.</p>
      <button onClick={() => setTheme('dark')}>Dark Theme</button>
      <button onClick={() => setTheme('red')}>Red Theme</button>
    </div>
  )
}

function CountProvider(props) {
  const [count, setCount] = React.useState(0)
  return <CountContext.Provider value={{ count, setCount }}>{props.children}</CountContext.Provider>
}
function ThemeProvider(props) {
  const [theme, setTheme] = React.useState('dark')
  return <ThemeContext.Provider value={{ theme, setTheme }}>{props.children}</ThemeContext.Provider>
}
export function App() {
  return (
    <div className='App'>
      <CountProvider>
        <ThemeProvider>
          <Header />
          <Content />
        </ThemeProvider>
      </CountProvider>
    </div>
  )
}
```

<!-- </Preview.React> -->

嵌套上下文常与 `useReducer` 配合使用，`useReducer` 返回的 `dispatch` 自带 memoize, 不会在多次渲染时改变

## Redux 呢？

## 参考

[React 组件到底什么时候 render 啊](https://mp.weixin.qq.com/s/rTQNpWBC7re9ykpwba8IAg)
[React 为什么重新渲染](https://blog.skk.moe/post/react-re-renders-101/#Rang-Wo-Men-Tan-Tan-Context)
