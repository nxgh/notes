## 概念和使用

web-component 由三项主要技术组成

- Custom elements（自定义元素）：一组 JavaScript API，允许您定义 custom elements 及其行为，然后可以在您的用户界面中按照需要使用它们。
- Shadow DOM（影子 DOM）：一组 JavaScript API，用于将封装的“影子”DOM 树附加到元素（与主文档 DOM 分开呈现）并控制其关联的功能。通过这种方式，您可以保持元素的功能私有，这样它们就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。
- HTML templates（HTML 模板）： `<template>` 和 `<slot>` 元素使您可以编写不在呈现页面中显示的标记模板。然后它们可以作为自定义元素结构的基础被多次重用。

```js
class ComponentName extends HTMLElement {
	constructor() {
		super()
	    var shadow = this.attachShadow( { mode: 'closed' } );

		shadow.appenChild()
	}
}
window.customElements.define('component-name', ComponentName);
```
1. 创建一个类或函数来指定 web 组件的功能
2. `customElements.define()` 注册自定义元素
3. `attachShadow()` 将一个 shadow DOM 添加到自定义元素上



## 生命周期

custom element 的构造函数中，可以指定多个不同的回调函数，它们将会在元素的不同生命时期被调用：

- `connectedCallback`：当 custom element 首次被插入文档 DOM 时，被调用。
- `disconnectedCallback`：当 custom element 从文档 DOM 中删除时，被调用。
- `adoptedCallback`：当 custom element 被移动到新的文档时，被调用。
- `attributeChangedCallback`: 当 custom element 增加、删除、修改自身属性时，被调用

