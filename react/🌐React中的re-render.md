# re-render 汇总

> 原文： [React re-renders guide: everything, all at once](https://www.developerway.com/posts/react-re-renders-guide)
>
> 译文: [一份详尽的 React re-render 指南](https://mp.weixin.qq.com/s/SH7N2f5ZhUhysQ7_G2s9rQ)

## 目录

1. 什么是 re-render?
2. 什么时候 React 组件会发生 re-render
3. 使用合成(composition)避免重新渲染
4. 使用 React.memo 避免重新渲染
5. 使用 useMomo/useCallback 提高重新渲染性能
6. 改进列表的重新渲染性能
7. 避免由于上下文(Context)引起的重新渲染

## 1. 什么是 re-render?

在性能方面，我们主要关注两个渲染阶段：

-   initial render：组件第一次在页面上进行渲染
-   re-render：已经在页面上渲染的组件进行第二次或后续多次渲染

当页面更新数据的时候，React 组件就会发生 re-render，比如用户与页面之间产生交互、异步请求数据或者订阅的外部数据更新等这些场景都会导致 re-render。那些没有任何异步数据更新的非交互式应用程序永远不会发生 re-render，因此这种应用场景不需要关心 re-render 的性能优化。

### 必要与非必要的 re-render

-   必要的 re-render：组件发生重新渲染的原因是数据发生了变化，组件要把最新的数据渲染到页面上。
-   不必要的 re-render：由于错误的实现方式，某个组件的 re-render 导致了整个页面全部重新渲染。

非必要的 re-render 本身不存在问题：React 非常快速，通常能够在用户还未注意到的情况下处理它们。然而，如果 re-render 过于频繁或在非常重的组件上进行时，可能会让用户感觉到 “卡顿”，在交互过程中会出现明显延迟，甚至页面完全没有响应。

## 2. 什么时候 React 组件会发生 re-render

-   状态变化
-   父组件 re-render
-   Context 变化
-   Hooks 变化

⛔️ 误区： 当组件的 props 改变时，组件会重新渲染。

未被 memo 包裹的组件 re-render 时，组件的 props 是否发生变化并不重要。组件的 props 即便是发生了改变，也是由父组件来更新它们。
也就是说，父组件的重新渲染触发了子组件的重新渲染，与子组件的 props 是否变化无关。只有那些使用了 React.memo 和 useMemo 的组件，props 的变化才会触发组件的重新渲染。

```jsx
const Child = ({ value }: { value: string }) => {
    console.log("Child re-renders");
    return <></>;
};

const value = "test";

const App = () => {
    const [state, setState] = useState(1);
    const onClick = () => {
        setState(state + 1);
    };
    return (
        <>
            <p>尽管 props 值未曾改变过, 每次点击事件仍会使 Child 重新渲染</p>
            <button onClick={onClick}>click here</button>
            <Child value={value} />
        </>
    );
};
```

### 状态变化

通常发生在回调或 useEffect 中。状态变化是所有重新渲染的根因。

```js
const Component = () => {
    const [state, setState] = useState()

    return ...
}
```

### 父组件重新渲染

```jsx
const Parent = () => {
    return <Child />;
};
```

### Context 变化

当 Context Provider 中的值发生变化时，使用该 Context 的所有组件都要 re-render，即使它们并没有使用发生变化的那部分数据。
这些 re-render 并不能直接通过 memoize 来避免掉，但是可以用一些变通的方法来避免

```jsx
const useValue = useContext(Context)

const Component1 = () => {
    const value = useValue()
    return ...
}

const Component2 = () => {
    const value = useValue()
    return ...
}
```

### Hooks 变化

hooks 中发生的一切都 “属于” 使用它的组件。因此 Context 和 State 的更新规则同样也适用于这里

## 3. 使用合成(composition) 避免重新渲染

### ⛔️ 反模式： 在 render 函数中创建组件

在组件的渲染函数中创建一个组件是一种反模式，很有可能会引起性能问题。在每次重新渲染时，React 都会重新装载这个组件（即销毁它并从头开始重新创建），这比正常的重新渲染要慢得多。除此之外，还会导致以下问题：

-   在重新渲染期间可能出现内容 “闪烁”
-   每次重新渲染时在组件的状态会被重置
-   每次重新渲染时不会触发依赖项的 useEffect
-   如果组件被聚焦，则焦点将丢失

```jsx
// 避免的写法
const Component = () => {
    // Component re-render 会重新生成一个 SlowComponent
    const SlowComponent = () => <Something />;
    return <SlowComponent />;
};
```

```jsx
const SlowComponent = () => <Something />;
const Component = () => {
    return <SlowComponent />;
};
```

### state 下移

-   🙁

```jsx
const Component = () => {
    const [open, setOpen] = useState(false);
    return (
        <>
            <Button onClick={() => setOpen(true)} />
            {isOpen && <ModalDialog />}
            <VerySlowComponent />
        </>
    );
};
```

-   🙂

```jsx
const ButtonWithDialog = () => {
    const [open, setOpen] = useState(false);
    return (
        <>
            <Button onClick={() => setOpen(true)} />
            {isOpen && <ModalDialog />}
        </>
    );
};

const Component = () => {
    return (
        <>
            <ButtonWithDialog />
            <VerySlowComponent />
        </>
    );
};
```

### children 作为 props

与状态下移比较类似，也是将状态变化封装在较小的组件中。 不同之处在于，这里的状态是作用于包裹复杂组件的父组件上，因此无法通过状态下移的方式作用于较小的组件。

一个典型的例子是绑定到组件根元素上 onScroll 或 onMouseMove 回调。在这种场景下，可以将状态管理和使用该状态的组件提取到较小的组件中，并且可以将较复杂的组件作为 children props 传递给它。

从较小的组件角度来看，children 只是 props，因此它们不会受到状态更改的影响，因此不会重新渲染。

-   🙁

```jsx
const Component = () => {
    const [value, setValue] = useState({});
    return (
        <div onScroll={(e) => setValue(e)}>
            <VerySlowComponent />
        </div>
    );
};
```

-   🙂

```jsx
const ComponentWithScroll = ({ children }) => {
    const [value, setValue] = useState({});
    return <div onScroll={(e) => setValue(e)}>{children}</div>;
};
const Component = () => {
    return (
        <ComponentWithScroll>
            <VerySlowComponent />
        </ComponentWithScroll>
    );
};
```

### 组件作为 props (components as props)

与前面的模式基本相同，将状态封装在一个较小的组件中，而较重的组件作为 props 传递给它。

props 不受 state 变化的影响，因此该较重的组件不会被重新渲染。

这个模式适用于那些状态独立的复杂组件，并且不能抽离成一个 children 属性的场景。

-   🙁

```jsx
const Component = () => {
    const [value, setValue] = useState({});
    return (
        <div onScroll={(e) => setValue(e)}>
            <VerySlowComponent1 />
            <Something />
            <VerySlowComponent2 />
        </div>
    );
};
```

-   🙂

```jsx
const ComponentWithScroll = ({ left, right }) => {
    const [value, setValue] = useState({});
    return (
        <div onScroll={(e) => setValue(e)}>
            {left}
            <Something />
            {right}
        </div>
    );
};
const Component = () => {
    return (
        <ComponentWithScroll
            left={<VerySlowComponent1 />}
            right={<VerySlowComponent2 />}
        />
    );
};
```

## 4. 使用 React.memo 避免重新渲染

### 使用 React.momo

使用 React.memo 后的组件只有 props 变更才会触发 re-render

> 为什么这不是默认行为?
>
> 因为作为开发者，我们往往高估了重新渲染的成本。对于 props 很多且没有很多子组件的组件来说，相比 re-render，检查 props 是否变更带来的消耗可能更大。因此，如果对每个组件都进行 React.memo，可能会产生反效果。

```jsx
const ChildMemo = React.momo(Child);
const Parent = () => {
    return <ChildMemo />;
};
```

### 带有 props 的组件

对于非基础数据类型的 props 都要用 React.memo 包装成为 memoize 值。

```jsx
const ChildMemo = React.momo(Child);
const Parent = () => {
    // 1. Parent re-render
    return <ChildMemo value={value} />; // 2.触发 value 变化,  re-render
};
```

```jsx
const ChildMemo = React.momo(Child);
const Parent = () => {
    const value = useMemo(() => ({ value }), []);
    return <ChildMemo value={value} />; // 不会 re-render
};
```

### 作为 props 或 children 的组件

React.memo 可用于作为 props 或 children 的元素。仅

对父组件进行包装是不起作用的：children 和 props 作为对象类型会在每次父组件渲染时都会变化，进而引起子组件重新渲染。

```jsx
const ChildMemo = React.momo(Child);
// Parent re-render 引起 Something 和 GrandChild re-render
const Parent = () => {
    return (
        <ChildMemo left={<Something />}>
            <GrandChild />
        </ChildMemo>
    );
};
```

```jsx
const SomethingMemo = React.momo(Something);
const GrandChildMemo = React.momo(GrandChild);
// Parent re-render 不会引起 Something 和 GrandChild re-render
const Parent = () => {
    return (
        <Child left={<SomethingMemo />}>
            <GrandChildMemo />
        </Child>
    );
};
```

## 5. 使用 useMomo/useCallback 提高重新渲染性能

### ⛔ 无用的 useMemo/useCallback props

将子组件的 props 包装成 memoize 值，是不能避免该子组件重新渲染的。**只要父组件重新渲染，那么子组件就会被重新渲染**，与 props 没有关系

```jsx
const Parent = () => {
    const value  = useMemo(() => ({value}))
    return <Child value={value}>
}
```

### 有用的 useMemo/useCallback

如果子组件被 React.memo 包装，那么它的所有非基础数据类型的 props 也要做 memoize 处理。

```jsx
const ChildMemo = React.memo(Child);

const Parent = () => {
    const value = useMemo(() => ({ value }));
    return <ChildMemo value={value}>
};
```

如果组件使用的 hooks（比如 useEffect、useMemo、useCallback）依赖了非基础数据类型的值，那么要做 memoize 处理。

```jsx
const Parent = () => {
    const value = useMemo(() => ({ value }));

    useEffect(() => {
        //  ...
    }, [value]);
};
```

### 使用 useMemo 处理昂贵的计算

useMemo 的典型场景是对 React 元素做 memoize 处理。在 React 中，挂载和更新组件在大多数情况下都是最昂贵的计算, 与组件更新相比，数组的排序或过滤等 “纯” JavaScript 操作的成本基本可以忽略不计

```jsx
const Component = () => {
    const slowComponent = useMemo(() => {
        return <SlowComponent />;
    }, []);
    return (
        <>
            <Something />
            {slowComponent}
        </>
    );
};
```

### 总结：什么时候应该用 useMemo/useCallback

1. React.memo 过的组件的 props 应该用，因为他们只有 props 变更时才会 re-render，所以反之非 React.memo 过的组件的 props 无需使用，因为都会 re-render
2. useEffect、useMemo、useCallback 中非原始值的依赖应该用
3. 给 Pure Component 的非原始值 props 应该用
4. 重消耗（比如生成渲染树）的部分应该用，反之轻消耗不要用，因为 useMemo/useCallback 本身也有消耗。

### useCallback 语法糖

通常:

-   useMemo 针对 array/object
-   useCallback 针对 function

```ts
React.useCallback(function helloWorld() {}, []);
// 等同于
React.useMemo(() => function helloWorld() {}, []);
```

### dep 更新导致返回的函数更新

```tsx
const Foo = () => {
    const onClick = useCallback(() => setCount(count + 1), [count]);
    return <div onClick={onClick} />;
};
```

```ts
useCallback(() => setCount((prevCount) => prevCount + 1), []);
```

## 6. 改进列表的重新渲染性能

key 的值会影响列表的渲染性能。仅仅设置 key 值并不会提高列表的性能

-   key 的值应为字符串，列表中的元素在每次重新渲染时，这个字符串要保持一致
-   如果列表是静态的, 可以使用数组的 index 作为 key
-   但是在动态列表中使用数组的索引会导致一些问题：
    -   如果列表数据中具有状态或任何不受控制的元素（如表单输入），则会出现错误
    -   如果列表数据包装在 React.memo 中，性能会下降

## 7. 避免由于上下文(Context)引起的重新渲染

### 将 Provider 的值做 memoize 处理

如果 Context Provider 没有在页面的根节点上，那么祖先节点的变化也会导致它被重新渲染

```jsx
const Component = () => {
    return <Context.Provider value={value}>{children}</Context.Provider>;
};
```

所以它的值也应该被 memoize。

```jsx
const Component = () => {
    const memoValue = useMemo(() => ({ value }), []);
    return <Context.Provider value={value}>{children}</Context.Provider>;
};
```

### 对数据和 API 做拆分

拆分 getter 和 setter，这样 getter 变更时依赖 setter 的部分不会 re-render

```jsx
const Component = () => {
    const [state, setState] = useState();
    return (
        <DataContext.Provider value={state}>
            <ApiContext.Provider value={setState}>
                {children}
            </ApiContext.Provider>
        </DataContext.Provider>
    );
};
```

### 对数据拆分

只有变化的数据块的使用者才会重新渲染

```jsx
const Component = () => {
    const [state, setState] = useState();
    const [state1, setState1] = useState();
    return (
        <Data1Context.Provider value={state}>
            <Data2Context.Provider value={state2}>
                {children}
            </Data2Context.Provider>
        </Data1Context.Provider>
    );
};
```

### 伪造 Context selector

对于那些部分使用了 Context 值的组件，即使使用了 useMemo hooks 也无法避免组件的重新渲染。但是，我们可以使用高阶组件和 React.memo 可以伪造出 Context selector。

```jsx
const withSomething = (Component) => {
    const MemoComponent = React.memo(Component);

    return () => {
        const { something } = useSomething();
        return <MemoComponent something={something} />;
    };
};


const Component =  withSomething(({something}) => {
    return ...
})
```

## 避免使用引用值

JavaScript 中有原始值和引用值的区分，由于 props 和 hooks dep 都会做 shadow equal，使用时尽量避免使用引用值，避免不了时需用 useMemo/useCallback 包一下

避免使用引用值:

1. 拍平，比如 `options={{showSideBar:true}}` 改成 `showSideBar=true`
2. 使用原始值子项，比如 `useMemo(fn, [user])` 改成 `useMemo(fn, [user.id])`。

## 定位 re-render 的几个方法。

1. 借助 React Devtools Chrome 插件，在「设置 > Profiler」里开启「Record why each component rendered while profiling」，再通过录制的方式排查，就能知道每个 re-render 的原因
2. 借助外部工具，比如 why-did-you-render 或 tilg。
