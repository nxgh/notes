- vue2/3、react16/18、angular、solidjs、svelte
- 组件基础
  - 模板、jsx、html
  - 条件渲染
  - 列表 & key
  - 表单输入
- 数据变更检测
  - virtual dom & diff
  - 双向绑定
  - 脏检查
  - 响应式 与 watch
  - 不可变数据(immer)
  - 单项数据流与双向绑定
  - 计算属性实现
- 生命周期
  - 初始化
  - 组件挂载、卸载
  - 组件更新
- 组件、逻辑复用
  - HOC
  - composition api & hooks
  - slot & props.children
- 组件通信
  - props
  - emit
  - 依赖注入
    - 父子通信、子父通信、兄弟通信、全局通信、透传
- refs
- 组件样式
- 事件处理
- 哲学概念

- 路由
- 状态管理
- typescript

## 组件基础

### 模板语法与 JSX

Vue 推荐使用一种基于 HTML 的模板语法

```vue
<template>
  <h1>Hello World</h1>
</template>
```

Vue 同时支持 JSX 及手写渲染函数, 但这将不会享受到和模板同等级别的编译时优化

> [渲染函数 & JSX](https://cn.vuejs.org/guide/extras/render-function.html#jsx-tsx)

```js
import { h } from 'vue'
const vnode = h(
  'div', // type
  { id: 'foo', class: 'bar' }, // props
  [
    /* children */
  ]
)
```

```jsx
const vnode = <div id={dynamicId}>hello, {userName}</div>
```

React 推荐使用名为 JSX 的 JavaScript 的语法扩展

```jsx
const Hello = () => {
  return <h1>Hello World</h1>
}
```

React 中每个 JSX 元素只是调用 React.createElement(component, props, ...children) 的语法糖。

> [babel：jsx 编译](https://babeljs.io/repl)

```js
const Hello = () => {
  return React.createElement('h1', null, 'Hello World')
}
```

---

### 常用的模板/jsx 绑定操作

#### 文本插值

```vue
<span>Message {{ msg }}</span>
```

```jsx
<span>Message {msg}</span>
```

#### 原始 HTML

> **渲染任意 HTML 是非常危险的，因为这非常容易造成 XSS 漏洞。请仅在内容安全可信时再使用 v-html，并且永远不要使用用户提供的 HTML 内容。**

```vue
<span v-html="rawHtml"></span>
```

```jsx
const Element = <div dangerouslySetInnerHTML={{ __html: 'First &middot; Second' }} />
```

#### Attribute 绑定

```vue
<div v-bind:id="dynamicId"></div>
<div :id="dynamicId"></div>
```

绑定多个

```js
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
}
```

```vue
<div v-bind="objectOfAttrs"></div>
```

```jsx
const Element = <div id={dynamicId}></div>
```

#### 使用 JavaScript 表达式

```vue
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div :id="`list-${id}`"></div>
<span :title="toTitleDate(date)">
  {{ formatDate(date) }}
</span>
```

```jsx
const Element = (
  <>
    {number + 1}
    {ok ? 'YES' : 'NO'}
    {message.split('').reverse().join('')}
    <div id={`list-${id}`}></div>
    <span title={toTitleDate(date)}>{formatDate(date)}</span>
  </>
)
```

#### 条件渲染

##### if-else

```vue
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

```jsx
const Element <h1>{awesome ? 'React is awesome!' : 'Oh no 😢'}</h1>
```

##### else-if

```vue
<div v-if="type === 'A'"> A </div>
<div v-else-if="type === 'B'"> B </div>
<div v-else-if="type === 'C'"> C </div>
<div v-else> Not A/B/C </div>
```

```jsx
const Element = <div>{type === 'A' ? 'A' : type === 'B' ? 'B' : type === 'C' ? 'C' : 'Not A/B/C'}</div>
```

##### v-show

```vue
<h1 v-show="ok">Hello!</h1>
```

```jsx
<h1 style={{display: ok ? '': 'none'}}">Hello!</h1>
```

#### 样式绑定

```js
const isActive = ref(true)
const hasError = ref(false)
```

```vue
<div class="static" :class="{ active: isActive, 'text-danger': hasError }">
</div>
```

渲染结果

```vue
<div class="static active"></div>
```

```jsx
const Element = <div className={`static ${isActive && 'active'} ${hasError && 'text-danger'}`}></div>
```

使用 classnames.js

```jsx
var classNames = require('classnames')

const Element = <div className={classNames('static', { active: isActive, 'text-danger': hasError })}></div>
```

渲染多个 class

```vue
<!--
const activeClass = ref('active')
const errorClass = ref('text-danger')
 -->
<div :class="[activeClass, errorClass]"></div>
```

渲染的结果是：

```vue
<div class="active text-danger"></div>
```

```jsx
const Element = <div className={`${activeClass} ${errorClass}`}></div>
```

条件渲染

```vue
<div :class="[isActive ? activeClass : '', errorClass]"></div>
<div :class="[{ active: isActive }, errorClass]"></div>
```

```jsx
const Element = <div className={`${isActive && active} errorClass`}></div>
```

#### 绑定 style

```js
const activeColor = ref('red')
const fontSize = ref(30)
const styleObject = reactive({
  color: 'red',
  fontSize: '13px',
})
```

```vue
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<div :style="styleObject"></div>
```

```jsx
const Element = <div style={{ color: activeColor, fontSize: fontSize + 'px' }}></div>
```

#### 组件内使用

```vue
<!-- MyComponent -->
<template>
  <p class="foo bar">Hi!</p>
</template>
```

使用

```vue
<MyComponent class="baz boo" />
```

渲染

```vue
<p class="foo bar baz boo">Hi</p>
```

指定元素接收

```vue
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>
```

```vue
<MyComponent class="baz" />
```

```vue
<p class="baz">Hi!</p>
<span>This is a child component</span>
```

```jsx
const MyComponent = ({ className }) => {
  return <p className={`${className}`}>Hi</p>
}
```

#### 列表渲染

#### for

```js
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
```

```vue
<li v-for="(item, index) in items">
  {{ item.message }} - {{ index}}
</li>
```

可以使用 of 作为分隔符来替代 in

```vue
<li v-for="(item, index) of items">
  {{ item.message }} - {{ index}}
</li>
```

```jsx
const Element = [{ message: 'Foo' }, { message: 'Bar' }].map((item, index) => (
  <li>
    {item.message} - {index}
  </li>
))
```

解构

```vue
<li v-for="({ message }, index) in items">
  {{ message }} {{ index }}
</li>
```

```jsx
const Element = [{ message: 'Foo' }, { message: 'Bar' }].map(({ message }, index) => (
  <li>
    {message} - {index}
  </li>
))
```

for 遍历对象

```js
const myObject = {
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2016-04-10',
}
```

```vue
<ul>
  <li v-for="(value, key, index) in myObject" key="value">{{ key }}:  {{ value }} </li>
</ul>
```

```jsx
const Element = (
  <ul>
    {Object.entries(myObject).map(([key, value], index) => (
      <li key={key}>
        {key}:{value}
      </li>
    ))}
  </ul>
)
```

```vue
<span v-for="n in 10" key="n">{{ n }}</span>
```

```jsx
const Element = [...Array(10).key()].map(n => <span key={n}>{n + 1}</span>)
```

渲染一个包含多个元素的块

```vue
<ul>
  <template v-for="item in items" key="item.msg">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

```jsx
const Element = (
  <ul>
    {items.map(item => (
      <Fragment key={item.msg}>
        <li>{item.msg}</li>
        <li classNames="divider" role="presentation"></li>
      </Fragment>
    ))}
  </ul>
)
```

for 与 if

vue 不推荐同时使用 v-if 和 v-for ,当它们同时存在于一个节点上时，v-if 比 v-for 的优先级更高

```vue
<!-- 这会抛出一个错误，因为属性 todo 此时 没有在该实例上定义 -->
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

解决

```vue
<template v-for="todo in todos" key="todo.name">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

```jsx
const Element = todos.map(todo => (
  <Fragment key={todo.name}>
    (todo.isComplete ? <li>{todo.name}</li> : <></>)
  </Fragment>
))
```

key 

- [vue -key](https://cn.vuejs.org/api/built-in-special-attributes.html#key)
- [react-key](https://zh-hans.reactjs.org/docs/lists-and-keys.html)


#### 事件处理

```vue
<a v-on:click="doSomething"> ... </a>
<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

```JSX
const Element = <a onClick={doSomething}> ... </a>
const Element = <a onClick={() => {}}> ... </a>
```


---
