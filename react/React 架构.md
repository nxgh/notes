[React Fiber架构原理剖析](https://segmentfault.com/a/1190000042271919)
[React 技术揭秘](https://react.iamkasong.com/preparation/oldConstructure.html#react15%E6%9E%B6%E6%9E%84)
[图解 React 的 diff 算法：核心就两个字 —— 复用](https://mp.weixin.qq.com/s/gaASCIVTuUfsM-cVP2teGw)


- jsx 通过 babel 编译为 `React.createElement`
- 在 Reconciler 阶段, `React Element` 对象会转为 `fiber` 对象
	- element 类型与转化成 fiber 的 tag 类型的对应关系
- `fiber` 对象通过 `sibling`、`return`、`child` 将每个 `fiber` 对象联系起来

















## 问题
- v16 为什么需要重构架构/v15 架构的缺点

## v16 为什么需要重构架构/v15 架构的缺点

> “stack” reconciler 是 React 15 及更早的解决方案

React15架构可以分为两层：

- *Reconciler*（协调器）—— 负责找出变化的组件
- *Renderer*（渲染器）—— 负责将变化的组件渲染到页面上

React15架构中
在 **Reconciler** 中，`mount`的组件会调用[mountComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，`update`的组件会调用[updateComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会递归更新子组件。

由于递归执行，所以更新一旦开始，中途就无法中断。

如果遇到应用中组件数量比较庞大，那么VirtualDOM 的层级就会比较深，带来的结果就是主线程被长期占用(递归更新时间超过了16ms)，进而阻塞渲染、造成卡顿现象

Fiber 架构就是为了支持 **可中断渲染** 

## Fiber Reconciler



## React  15 架构

React v15 架构可分为两层：

- ==Reconciler== （协调器）—— 负责找出变化的组件
- ==Renderer==（渲染器）—— 负责将变化的组件渲染到页面上

每当有更新发生时，**Reconciler**会做如下工作：

- 调用函数组件、或class组件的`render`方法，将返回的JSX转化为虚拟DOM
- 将虚拟DOM和上次更新时的虚拟DOM对比
- 通过对比找出本次更新中变化的虚拟DOM
- 通知**Renderer**将变化的虚拟DOM渲染到页面上

在每次更新发生时，**Renderer**接到**Reconciler**通知，将变化的组件渲染在当前宿主环境。

在**Reconciler**中，`mount`的组件会调用[mountComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，`update`的组件会调用[updateComponent](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会递归更新子组件。

由于递归执行，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16ms，用户交互就会卡顿。

## React 16 架构

React16架构可以分为三层：

- **==Scheduler==**（调度器）—— 调度任务的优先级，高优任务优先进入**Reconciler**
- Reconciler（协调器）—— 负责找出变化的组件, 并打上不同的Flags
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上

### Scheduler

由于以下因素，React 放弃了使用 `requestIdleCallback` 作为调度器：

- 浏览器兼容性
- 触发频率不稳定，受很多因素影响

`React`实现了功能更完备的`requestIdleCallback`polyfill。除了在空闲时触发回调的功能外，**Scheduler**还提供了多种调度优先级供任务设置。

### Reconciler

[React16的Reconciler ](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1673)更新工作从递归变成了可以中断的循环过程, `shouldYield`判断当前是否有剩余时间。

> ==异步可中断更新==可以理解为：更新在执行过程中可能会被打断（浏览器时间分片用尽或有更高优任务插队），当可以继续执行时恢复之前执行的中间状态。

```js
/** @noinline */
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

v16 中 **Reconciler**与**Renderer**不再是交替工作。当**Scheduler**将任务交给**Reconciler**后，**Reconciler**会为变化的虚拟DOM打上代表增/删/更新的标记

```js
export const Placement = /*             */ 0b0000000000010;
export const Update = /*                */ 0b0000000000100;
export const PlacementAndUpdate = /*    */ 0b0000000000110;
export const Deletion = /*              */ 0b0000000001000;
```

> 全部的标记见[这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js)

整个**Scheduler**与**Reconciler**的工作都在内存中进行。只有当所有组件都完成**Reconciler**的工作，才会统一交给**Renderer**

### Renderer

**Renderer**根据**Reconciler**为虚拟DOM打的标记，同步执行对应的DOM操作。