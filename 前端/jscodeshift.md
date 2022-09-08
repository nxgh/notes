[jscodeshift](https://github.com/facebook/jscodeshift) 是 Fb 开源的一款基于 ast 代码自动化重构工具
配套使用

- [babel](https://babeljs.io/)
- [recast](https://github.com/benjamn/recast) jscodeshift 使用的解析器
- [AST explorer](http://astexplorer.net/) 语法树可视化工具
- [ast-types ](https://github.com/benjamn/ast-types)jscodeshift 所有 ast 相关操作

## 与 babel 的区别

**babel**

- `babel`的 `transform api` 是`visitor`风格，也就是声明对什么 `ast` 做什么处理，然后在遍历过程中被调用，这种不和具体遍历方式耦合的写法是一种设计模式（访问者模式）
- 处理复杂的场景优势，处理简单场景啰嗦

**jscodeshift**

- `collection`风格，类似 `jquery`，主动查找 `ast`，放到集合中操作，适合处理简单场景
  > 为什么不是`babel` > `babel`的目的是对代码向下兼容的，会进行代码转换，而且即使不做任何修改，输出的代码和原本的也有区别，比如空格，空行，注释位置会变化，而`jscodeshift`会保留没发生修改的代码的原有格式。

## api.jscodeshift

`api.jscodeshift`将字符串源文件`file.source`转换为一个可遍历/操作的`Collection`

```javascript
module.exports = (file, api) => {
  const j = api.jscodeshift;
  const root = j(file.source);

  return root.toSource();
};
```

`api.jscodeshift` 是 `jscodeshift` 提供的一个 `Collection`构造函数，该函数对象上绑定了 `ast-types` 的 `namedTypes `和 `builders`。
其中 `namedTypes`为大写字母开头，例如代码中的 `j.MemberExpression` ，它就是一个 `Type`对象。向该构造函数提供源码字符串或 `AST` 就可以通过 `new`关键字构造出一个 `Collection` 对象

## AST 节点

AST 节点的类型是固定的，可以参考 [@babel/types](https://babeljs.io/docs/en/babel-types) 文档

- ** NodePath**

`ast-types` 的 `NodePath`类代表了 `AST` 的一个节点。通过 `NodePath` 类的 `node` 属性可以得到以该节点为根节点的子树结构，可以很方便的获得该节点及其子树节点的各种值

- `Collection`类

`Collection` 类是 `jscodeshift` 对 `ast-types` 节点的封装
它提供了 `AST` 的遍历、查找和操作的方法，这些方法的都以一个新的 `Collection` 对象作为返回值，这样能方便的进行链式调用，让代码显得更简洁。

- `Collection.__paths`

`__paths` 属性是一个 `NodePath` 对象的数组，包含了当前操作的 AST 树节点的集合

## 常用方法

| 类别 | api                                                    |
| ---- | ------------------------------------------------------ |
| 查找 | `find`, `filter`                                       |
| 修改 | `replaceWith`, `insertBefore`, `insertAfter`, `remove` |
| 其他 | `forEach`, `map`, `size`, `at`                         |

### find

```typescript
find(type: recast.NamedTypes, filter): Collection type
```

`find` 和 `filter` 主要用于查找和定位我们需要修改的节点。
`find` 方法接受两个参数：`type` 和 `option`（可选）。`type` 即是上文所说的 `Type` 对象，`option` 是附加的过滤条件，例如 `root.find(j.Identifier, { name: 'get' })` 就表示找到名称为`"get"`的`Identifier` 节点

### filter

`filter` 方法其实就是调用了 `Collection` 类 `__paths` 属性的值（一个 `NodePath` 对象数组）的 `filter` 方法，该方法接受一个布尔返回值的回调函数，这个回调函数的参数就是遍历的 `NodePath` 对象

### forEach

`forEact` 和 `map`也对应 `NodePath`对象数组的同名方法

### findVariableDeclarators

对`find`的封装

```javascript
findVariableDeclarators: function(name) {
    const filter = name ? {id: {name: name}} : null;
    return this.find(VariableDeclarator, filter);
}
```

### root.toSource

`root.toSource` 将整个 AST 转化为源码并返回=

## 执行转换

---

```bash
node jscodeshift -t ./codemod.js file ./test.js

# 当前文件夹下执行
node ./node_modules/jscodeshift/bin/jscodeshift.js  -t ./codemod.js file ./test.js

# ts 文件指定
--parser=ts --extensions=js,ts

# tsx 文件指定
--parser=tsx --extensions=tsx
```

## 示例

---

### 替换变量

```javascript
module.exports = function (fileInfo, api) {
  return api
    .jscodeshift(fileInfo.source)
    .findVariableDeclarators("foo")
    .renameTo("bar")
    .toSource();
};
```

### 使用 jscodeshift 在文件开头插入一行

```javascript
export default (file, api) => {
  const j = api.jscodeshift;
  const s = '"use strict";';
  const root = j(file.source);

  root.get().node.program.body.unshift(s);

  return root.toSource();
};
```

### 在`use strict`声明之后添加`import`

```javascript
export default (file, api) => {
  const j = api.jscodeshift;
  const s = '"use strict";';
  const root = j(file.source);
  const imports = root.find(j.ImportDeclaration);
  const n = imports.length;

  if (n) {
    //j(imports.at(0).get()).insertBefore(s); // before the imports
    j(imports.at(n - 1).get()).insertAfter(s); // after the imports
  } else {
    root.get().node.program.body.unshift(s); // begining of file
  }

  return root.toSource();
};
```
