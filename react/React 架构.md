[React Fiber架构原理剖析](https://segmentfault.com/a/1190000042271919)
[React 技术揭秘](https://react.iamkasong.com/preparation/oldConstructure.html#react15%E6%9E%B6%E6%9E%84)
[图解 React 的 diff 算法：核心就两个字 —— 复用](https://mp.weixin.qq.com/s/gaASCIVTuUfsM-cVP2teGw)


- v16 为什么需要重构架构/v15 架构的缺点

 


## 源码结构

- React Core
	React Core **只包含定义组件必要的 API**
- Renderer 渲染器
	**渲染器用于管理一棵 React 树，使其根据底层平台进行不同的调用。**
- Reconcilers 协调算法
	用来更新并且渲染DOM树的算法, v16之前是 `stack reconciler`, v16 之后为 `fiber reconciler`
- Fiber Reconciler




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

