## JSX

```jsx
const element = <h1 title="foo">Hello</h1>

const container = document.querySelector('#root')

ReactDom.render(element, container)		
```

JSX 转换为 JS 代码的过程由 Babel 之类的构建工具来完成

```bash
yarn add @babel/core @babel/cli @babel/plugin-transform-react-jsx -D
```

```bash
 .\node_modules\.bin\babel --plugins @babel/plugin-transform-react-jsx index.jsx
```

输出

```js
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
);
```

> `@babel/plugin-transform-react-jsx` 中可以显示的指定需要将 JSX 编译为什么函数调用
>
> preact 中会编译为一个名为 `h` 的函数调用



## createElement

1. `createElement` 作用根据传入的参数创建 `ReactElement` 对象

### ReactElement

`ReactElement` 对象是一个用来承载信息的容器，它挂载了节点的以下信息

1. `type`类型，用于判断如何创建节点
2. `key`和`ref`这些特殊信息
3. `props`新的属性内容
4. `$$typeof`用于确定是否属于`ReactElement`

### 实现

在一个 React Element 中，我们只需关注 `type` 和 `props` 两个属性

- `type` 指代 `ReactElement` 的类型
- `props` 是另一个对象，具有 JSX attributes 中所有键值对, 还有一个特殊的属性 `children`, 通常是一个包含更多 elements 的数组

```js
const createElement = (type, props, ...children) => {
  return {
    type,
    props: { ...props, children },
  }
}	
```

处理非对象类型

> 这里为了简化代码，在没有 `children` 时创建了一个空数组，React 中不会对原始类型进行包装

```js
const createTextElement = text => ({
  type: 'TEXT_ELEMENT',
  props: { nodeValue: text, children: [] },
})

const createElement = (type, props, ...children) => {
  return {
    type,
    props: { 
        ...props,
        children: children.map(child => (typeof child === 'object' ? child : createTextElement(child))) 
    },
  }
}
```

现在实现 `render` 函数，只需要关系向 DOM 中添加内容

```js
function render(element, container) {
  const dom = element.type == 'TEXT_ELEMENT' ? document.createTextNode('') : document.createElement(element.type)

  const isProperty = key => key !== 'children'

  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })

  element.props.children.forEach(child => render(child, dom))

  container.appendChild(dom)
}

```

##  重构 异步可中断

```js
function render(element, container) {
  // ...
  element.props.children.forEach(child => render(child, dom))
  // ...
}

```

这里的递归使得渲染一旦开始，在整棵 element tree 渲染完成之前程序是不会停止。如果这棵 element tree 过于庞大，它有可能会阻塞主进程太长时间

**我们将渲染工作分成几个小部分，在完成每个单元后，如果需要执行其他操作，我们将让浏览器中断渲染**

```js
let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}

requestIdleCallback(workLoop)

function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

> [`requestIdleCallback`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 
>
> `window.requestIdleCallback()`方法插入一个函数，这个函数将**在浏览器空闲时期被调用**，可以把 `requestIdleCallback` 当作是一个 `setTimeout`

为了组织各个工作单元，我们需要一个新的数据结构：Fiber tree

### Fiber Tree

为每一个 element 分配一个 fiber，而每个 fiber 将成为一个工作单元

假如有以下结构

```jsx
 <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>
```

对应的 Fiber Tree 结构

<img src="D:\Workspace\build-your-own-react\docs\images\无标题-2022-02-28-1411.png" alt="对应的 Fiber Tree 结构" style="zoom: 20%;" />

设计这个数据结构的目标之一是：使查找下一个工作单元变得更加容易

**每一个 Fiber 都会链接到其第一个子节点 `child`，下一个兄弟姐妹节点`sibling`和其父节点`parent`**

将创建 DOM node 的部分代码抽离处理

```js
function createDom(fiber) {
  const dom = fiber.type == 'TEXT_ELEMENT' ? 
        document.createTextNode('') : 
        document.createElement(fiber.type)

  const isProperty = key => key !== 'children'
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })

  return dom
}
```

在 `render` 中设置 Fiber Tree 的根节点

```js
let nextUnitOfWork = null

function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}

```

当浏览器准备好的时候, 将会调用 `workLoop` 函数，从根节点开始执行 `performUnitOfWork` 

```js
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}

requestIdleCallback(workLoop)

```

`performUnitOfWork` 将为每个 fiber 执行三件事

1. 将 element 添加至 DOM
2. 为 element 的 children 创建 fiber
3. 选出下一个工作单元

```js
function performUnitOfWork(fiber) {
  // 1. 创建一个 node 节点然后将其添加至 DOM
  // 将这个 DOM node 保存在 fiber.dom 属性中以持续跟踪
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }

  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
  // 2. 为每一个chid创建一个新的 fiber
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null

  while (index < elements.length) {
    const element = elements[index]

    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
    // 将其添加到 Fiber Tree 中，它是 child 还是 sibling ，取决于它是否是第一个 child。
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }

    prevSibling = newFiber
    index++
  }
  // 选出下一个工作单元
  // 首先寻找 child ,其次 sibling ,然后是 parent 的 sibling
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

## Render and Commit Phases

当我们在处理一个 React element 时，我们都会添加一个新的节点到 DOM 中，而浏览器在渲染完成整个树之前可能会中断我们的工作。在这种情况下，用户将会看不到完整的 UI。

所以我们需要删除那部分对 DOM 进行修改的代码：

```js 
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
 // if (fiber.parent) {
 //   fiber.parent.dom.appendChild(fiber.dom)
 // }
 //...
}
```

跟踪 Fiber Tree 的根节点

```js
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}

let nextUnitOfWork = null
let wipRoot = null
```

一旦完成所有工作（直到没有 `nextUnitOfWork` ），我们便将整个 Fiber Tree 交给 DOM。

```js
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}

function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}

let nextUnitOfWork = null
let wipRoot = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }

  if (!nextUnitOfWork && wipRoot) {
    commitRoot()
  }

  requestIdleCallback(workLoop)
}

requestIdleCallback(workLoop)
```

## 更新或删除节点

将在 `render` 函数上接收到的 `elements` 与我们提交给 DOM 的最后一棵 Fiber Tree 进行比较

在完成 commit 之后，我们需要对「最后一次 commit 到 DOM 的一棵 Fiber Tree」 的引用进行保存。我们称它为 `currentRoot` 。同时我们也对每个 Fiber 添加了一个 `alternate` 属性。这个属性是对旧 Fiber 的链接，这个旧 Fiber 是我们在在上一个 commit phase 向 DOM commit 的 Fiber

```js
function commitRoot() {
  commitWork(wipRoot.child)
	// currentRoot: 最后一次 commit 到 DOM 的一棵 Fiber Tree
  currentRoot = wipRoot
  wipRoot = null
}

function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  }
  nextUnitOfWork = wipRoot
}

let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null
```

## Reconcile

把 `performUnitOfWork` 中用来创建新 Fiber 的部分代码抽离成一个新的函数 `reconcileChildren` 

在这里我们将会旧 Fibers 与新 elements进行 `reconcile` 

1. 我们使用 `type` 属性对它们进行比较：

2. - 如果老的 Fiber 和新的 element 拥有相同的 type，我们可以保留 DOM 节点并仅使用新的 Props 进行更新。这里我们会创建一个新的 Fiber 来使 DOM 节点与旧的 Fiber 保持一致，而props 与新的 element 保持一致。

   - - 我们还向 Fiber 中添加了一个新的属性 `effectTag` ，这里的值为 `UPDATE` 。为稍后我们将在 commit 阶段使用这个属性。

3. - 如果两者的 `type` 不一样并且有一个新的 element，这意味着我们需要创建一个新的 DOM 节点。

   - - 在这种情况下，我们会用 `PLACEMENT` effect tag 来标记新的 Fiber。

4. - 如果两者的 `type` 不一样，并且有一个旧的 Fiber，我们需要删除旧节点。

   - - 在这种情况下，我们没有新的 Fiber，所以我们会把 `DELETION`effect tag 添加到旧 Fiber 中。

```js
function reconcileChildren(wipFiber, elements) {
  let index = 0
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child
  let prevSibling = null

  while (index < elements.length || oldFiber != null) {
    const element = elements[index]
    let newFiber = null

    const sameType = oldFiber && element && element.type == oldFiber.type

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: 'UPDATE',
      }
    }
    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: 'PLACEMENT',
      }
    }
    if (oldFiber && !sameType) {
      oldFiber.effectTag = 'DELETION'
      deletions.push(oldFiber)
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling
    }

    if (index === 0) {
      wipFiber.child = newFiber
    } else if (element) {
      prevSibling.sibling = newFiber
    }

    prevSibling = newFiber
    index++
  }
}

```

## 处理不同类型的变化

```js
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

处理前面定义的各种 `effectTags`

```js
function commitWork(fiber) {
  if (!fiber) {
    return
  }

  let domParentFiber = fiber.parent
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent
  }
  const domParent = domParentFiber.dom

  if (fiber.effectTag === 'PLACEMENT' && fiber.dom != null) {
    domParent.appendChild(fiber.dom)
  } else if (fiber.effectTag === 'UPDATE' && fiber.dom != null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props)
  } else if (fiber.effectTag === 'DELETION') {
    commitDeletion(fiber, domParent)
  }

  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

- `PLACEMENT`：这个 DOM 节点添加到父 Fiber  的节点上
- `DELETION`：删除这个 child
- `UPDATE` ：使用最新的 props 来更新现有的 DOM 节点

```js
function updateDom(dom, prevProps, nextProps) {
  //Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(key => !(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2)
      dom.removeEventListener(eventType, prevProps[name])
    })

  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = ''
    })

  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })

  // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2)
      dom.addEventListener(eventType, nextProps[name])
    })
}
```
