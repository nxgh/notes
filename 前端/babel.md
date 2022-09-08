>  babel 取自圣经典故巴别塔

## 用途 

- 转译 `esnext`、`typescript`   等到目标环境支持的  js
- 一些特定用途的代码转换
- 代码的静态分析
> 编译器 Compiler 一般是指高级语言到低级语言的转换工具
> 转译器 (Transpiler) 指对于高级语言到高级语言的转换工具

## babel  的编译流程
babel 是 source to source 的转换，整体编译流程分为三步：

- parse：通过 parser 把源码转成抽象语法树（AST）
- transform：遍历 AST，调用各种 transform 插件对 AST 进行增删改
- generate：把转换后的 AST 打印成目标代码，并生成 sourcemap

总结一下就是：为了让计算机理解代码需要先对源码字符串进行 parse，生成 AST，把对代码的修改转为对 AST 的增删改，转换完 AST 之后再打印成目标代码字符串。

- parse

把源码字符串转换成机器能够理解的 AST，这个过程分为词法分析、语法分析

- transform

对 parse 生成的 AST 的处理，会进行 AST 的遍历，遍历的过程中处理到不同的 AST 节点会调用注册的相应的 visitor 函数，visitor 函数里可以对 AST 节点进行增删改，返回新的 AST（可以指定是否继续遍历新生成的 AST

- generate

generate 阶段会把 AST 打印成目标代码字符串，并且会生成 sourcemap。

## AST 
AST  即抽象语法树（abstract syntax tree）
源码是一串按照语法格式来组织的字符串，人能够认识，但是计算机并不认识，想让计算机认识就要转成一种数据结构，通过不同的对象来保存不同的数据，并且按照依赖关系组织起来，这种数据结构就是抽象语法树
字面量、标识符、表达式、语句、模块语法、class 语法都有各自的 AST
使用时一般通过 [ast 可视化工具查看](https://astexplorer.net/)
### Literal 字面量
字符串字面量 `StringLiteral`
数字字面量 `NumericLiteral`
布尔字面量 `BooleanLiteral`
正则表达式字面量 `RegExpLiteral`
null `NullLiteral`
bigInt `BigintLiteral`
模板字面量 `TemplateLiteral`
### Identifier 标识符
变量名、属性名、参数名等各种声明和引用的名字，都是Identifer
### Statement 语句

- 语句是代码执行的最小单位
- 比如 break、continue、debugger、return 或者 if 语句、while 语句、for 语句，还有声明语句，表达式语句等
```javascript
break -> BreakStatement
continue -> ContinueStatement
return -> ReturnStatement
debugger  ->  DebuggerStatement
throw Error() -> ThrowStatement
{} -> BlockStatement
try {} catch(e) {} -> TryStatement
for() {} ForStatement
for(let k in obj) {} -> ForInStatement
while(true) {} -> WhileStatement
do{} while()  -> DoWhileStatement
switch(v) {} ->  SwitchStatement
label: console.log() -> LabeledStatement
with(a) {} -> WithStatement
```
### Declaration 声明语句
一种特殊的语句，它执行的逻辑是在作用域内声明一个变量、函数、`class`、`import`、`export `等
```javascript
const a = 1;  // VariableDeclaration
function b(){} // FunctionDeclaration
class C {}  // ClassDeclaration

import d from 'e'; // ImportDeclaration

export default e = 1; // ExportDefaultDeclaration
export {e};  // ExportNamedDeclaration
export * from 'e'; // ExportAllDeclaration
```
### Expression 表达式
是执行完以后有返回值
```javascript
[1,2,3]  // ArrayExpression
a = 1     // AssignmentExpression 赋值表达式
1 + 2;    // BinaryExpression 二元表达式
-1;      // UnaryExpression 一元表达式
function(){}; // FunctionExpression
() => {};  // ArrowFunctionExpression
class{};  // ClassExpression
a;        // Identifier 标识符
this;     // ThisExpression 
super;     // Super
a::b;      // BindExpression 绑定表达式
```
### Class
整个 `class`的内容是 `ClassBody`，属性是 `ClassProperty`，方法是`ClassMethod`（通过 `kind` 属性来区分是 `constructor`还是 `method`）
### Modules
es module 是语法级别的模块规范，所以也有专门的 AST 节点。
### Import
```javascript

import {c, d} from 'c'; // ImportSpicifier
import a from 'a';      // ImportDefaultSpecifier
import * as b from 'b'; // ImportNamespaceSpcifier

```
这 3 种语法都对应 ImportDeclaration 节点，但是 specifiers 属性不同。
### Export
```javascript
export { b, d};     // ExportNamedDeclaration
export default a;   // ExportDefaultDeclaration
export * from 'c';  // ExportAllDeclaration 

```
### Program

- `program` 是代表整个程序的节点，它有 `body` 属性代表程序体，存放 `statement` 数组，就是具体执行的语句的集合
### Directive

- 还有 `directives`属性，存放  Directive 节点，比如`"use strict"` 这种指令会使用 `Directive`节点表示
### File
babel 的 AST 最外层节点是 File，它有 program、comments、tokens 等属性，分别存放 Program 程序体、注释、token 等，是最外层节点
### Comment 注释
注释分为块注释和行内注释，对应 `CommentBlock `和 `CommentLine `节点
```javascript
// 行内注释   CommentLine 
/* 块注释 */  CommentBlock 
```
## AST 的公共属性

- `type`： AST 节点的类型
- `start`、`end`：`start` 和` end` 代表该节点对应的源码字符串的开始和结束下标，不区分行列
- `loc `属性是一个对象，有 `line` 和 `column` 属性分别记录开始和结束行列号。
- `leadingComments`、`innerComments`、`trailingComments`： 表示开始的注释、中间的注释、结尾的注释，因为每个 AST 节点中都可能存在注释，而且可能在开始、中间、结束这三种位置，通过这三个属性来记录和 `Comment`的关联。
- `extra`：记录一些额外的信息，用于处理一些特殊情况。比如 `StringLiteral`修改 `value `只是值的修改，而修改 `extra.raw` 则可以连同单双引号一起修改。
## Babel Api
主要是 `@babel/parser`，`@babel/traverse`，`@babel/generator`，`@babel/types`，`@babel/template` 这五个 包 的 api 使用
**parse**

- `@babel/parser`，功能是把源码转成 AST

**transform **

- `@babel/traverse`，可以遍历 AST，并调用 visitor 函数修改 AST
- `@babel/types`  AST 的判断、创建、修改等
- `@babel/template`批量创建 AST 的时候可以使用  来简化 AST 创建逻辑

**generate**

- generate 阶段会把 AST 打印为目标代码字符串，同时生成 sourcemap，需要 `@babel/generator`

中途遇到错误想打印代码位置的时候，使用 `@babel/code-frame` 包
babel 的整体功能通过 `@babel/core` 提供，基于上面的包完成 babel 整体的编译流程，并实现插件功能。
### @babel/parse
基于 acorn 实现的，扩展了很多语法，可以支持 es next（现在支持到 es2020）、jsx、flow、typescript 等语法的解析(指定语法插件)

提供了有两个 api：parse 和 parseExpression。两者都是把源码转成 AST，不过 parse 返回的 AST 根节点是 File（整个 AST），parseExpression 返回的 AST 根节点是是 Expression（表达式的 AST），粒度不同。
```javascript
function parse(input: string, options?: ParserOptions): File 
function parseExpression(input: string, options?: ParserOptions): Expression 
```
options 主要分为两类，一是 parse 的内容是什么，二是以什么方式去 parse
#### parse 的内容是什么：

- `plugins`： 指定jsx、typescript、flow 等插件来解析对应的语法
- `allowXxx`： 指定一些语法是否允许，比如函数外的 await、没声明的 export等
- `sourceType`： 指定是否支持解析模块语法，有 module、script、unambiguous 3个取值，module 是解析 es module 语法，script 则不解析 es module 语法，当作脚本执行，unambiguous 则是根据内容是否有 import 和 export 来确定是否解析 es module 语法。
#### 以什么方式 parse

- `strictMode` 是否是严格模式
- `startLine` 从源码哪一行开始 parse
- `errorRecovery` 出错时是否记录错误并继续往下 parse
- `tokens parse` 的时候是否保留 token 信息
- `ranges `是否在 ast 节点中添加 ranges 属性
```javascript
// parse tsx 模块
require("@babel/parser").parse("code", {
  sourceType: "module",
  plugins: [
    "jsx",
    "typescript"
  ]
});
```
### @babel/traverse
parse 出的 AST 由 `@babel/traverse` 来遍历和修改，babel traverse 包提供了 `traverse`方法：
```javascript
function traverse(parent, opts) 
```
常用的就前面两个参数，`parent` 指定要遍历的 AST 节点，`opts` 指定 `visitor` 函数。babel 会在遍历 `parent` 对应的 AST 时调用相应的 `visitor` 函数。

#### 遍历过程
visitor 对象的 value 是对象或者函数：

- 如果 value 为函数，那么就相当于是 enter 时调用的函数。
- 如果 value 为对象，则可以明确指定 enter 或者 exit 时的处理函数。
   - `enter` 时调用是在遍历当前节点的子节点前调用
   - `exit` 时调用是遍历完当前节点的子节点后调用。

函数会接收两个参数 `path` 和 `state`。
```javascript
visitor: {
    Identifier (path, state) {},
    StringLiteral: {
        enter (path, state) {},
        exit (path, state) {}
    }
}
```
多个节点类型通过` | `连接
```javascript
// 进入 FunctionDeclaration 节点时调用
traverse(ast, {
  FunctionDeclaration: {
      enter(path, state) {}
  }
})

// 默认是进入节点时调用，和上面等价
traverse(ast, {
  FunctionDeclaration(path, state) {}
})

// 进入 FunctionDeclaration 和 VariableDeclaration 节点时调用
traverse(ast, {
  'FunctionDeclaration|VariableDeclaration'(path, state) {}
})

// 通过别名指定离开各种 Declaration 节点时调用
traverse(ast, {
  Declaration: {
      exit(path, state) {}
  }
})
```
具体的别名有哪些在[babel-types 的类型定义](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fbabel%2Fbabel%2Fblob%2Fmain%2Fpackages%2Fbabel-types%2Fsrc%2Fast-types%2Fgenerated%2Findex.ts%23L2489-L2535)
#### path
path 是遍历过程中的路径，会保留上下文信息，有很多属性和方法，比如:

- `path.node` 指向当前 AST 节点
- `path.get`、`path.set` 获取和设置当前节点属性的 path
- `path.parent `指向父级 AST 节点
- `path.getSibling`、`path.getNextSibling`、`path.getPrevSibling` 获取兄弟节点
- `path.find` 从当前节点向上查找节点
- `path.scope`获取当前节点的作用域信息
- `path.isXxx` 判断当前节点是不是 xx 类型
- `path.assertXxx` 判断当前节点是不是 xx 类型，不是则抛出异常
- `path.insertBefore`、`path.insertAfter` 插入节点
- `path.replaceWith`、`path.replaceWithMultiple`、`replaceWithSourceString` 替换节点
- `path.remove` 删除节点
- `path.skip` 跳过当前节点的子节点的遍历
- `path.stop` 结束后续遍历
#### state
 state 则是遍历过程中在不同节点之间传递数据的机制，插件会通过 state 传递 options 和 file 信息，我们也可以通过 state 存储一些遍历过程中的共享数据
### @babel/types
遍历 AST 的过程中需要创建一些 AST 和判断 AST 的类型
举个🌰
创建`IfStatement`
```javascript
t.ifStatement(test, consequent, alternate);
```
判断节点是否是 `IfStatement`
isXxx 会返回 boolean 表示结果，而 assertXxx 则会在类型不一致时抛异常。
```javascript
t.isIfStatement(node, opts);
t.assertIfStatement(node, opts);
```
opts 可以指定一些属性是什么值，增加更多限制条件，做更精确的判断
```javascript
t.isIdentifier(node, { name: "paths" })
```
### @babel/template
`@babel/template`api 包括
```javascript
const ast = template(code, [opts])(args);
const ast = template.ast(code, [opts]);
const ast = template.program(code, [opts]);
```

- `template.ast`或者 `template.program` 方法，都是直接返回 ast 
- `template.program` 返回的 AST 的根节点是 `Program`
- 如果知道具体创建的 AST 的类型，可以使用 `template.expression`、`template.statement`、`template.statements` 等方法创建具体的 AST
> 默认 `template.ast` 创建的 `Expression`会被包裹一层 `ExpressionStatement` 节点（会被当成表达式语句来 `parse`），但当 `template.expression` 方法创建的 AST 就不会

如果模版中有占位符
```javascript
const fn = template(`console.log(NAME)`);

const ast = fn({
  NAME: t.stringLiteral("whh"),
});
```
或者
```javascript
const fn = template(`console.log(%%NAME%%)`);

const ast = fn({
  NAME: t.stringLiteral("whh"),
});
```
### @babel/generator
AST 转换完之后就要打印成目标代码字符串，通过 @babel/generator 包的 generate api
```javascript
function (ast: Object, opts: Object, code: string): {code, map}  
```

- 第一个参数是要打印的 AST
- 第二个参数是 `options`，指定打印的一些细节，比如通过 `comments` 指定是否包含注释，通过 `minified` 指定是否包含空白字符
- 第三个参数当[多个文件合并打印](https://link.juejin.cn/?target=undefined)的时候需要用到

`options` 中常用的是 sourceMaps，开启了这个选项才会生成 sourcemap
**const** { code, map } = generate(ast, { sourceMaps: true })
### @babel/code-frame
当有错误信息要打印的时候，需要打印错误位置的代码，可以使用`@babel/code-frame`。
```javascript
const result = codeFrameColumns(rawLines, location, {   /* options */ }); 
```
options 可以设置 highlighted （是否高亮）、message（展示啥错误信息）
🌰
```javascript
const { codeFrameColumns } = require("@babel/code-frame");

try {
 throw new Error("xxx 错误");
} catch (err) {
  console.error(codeFrameColumns(`const name = whh`, {
      start: { line: 1, column: 14 }
  }, {
    highlightCode: true,
    message: err.message
  }));
}
```
### @babel/core
`@babel/core` 包则是基于以上包完成整个编译流程，从源码到目标代码，生成 sourcemap。
```javascript
// 从源代码开始处理
transformSync(code, options); // => { code, map, ast }
// 从源代码文件 
transformFileSync(filename, options); // => { code, map, ast }
// 从源代码 AST
transformFromAstSync(
  parsedAst,
  sourceCode,
  options
); // => { code, map, ast }
```
`options`主要配置 `plugins` 和 `presets`，指定具体要做什么转换。
异步的版本，异步地进行编译，返回一个 `promise`
```javascript
transformAsync("code();", options).then(result => {})
transformFileAsync("filename.js", options).then(result => {})
transformFromAstAsync(parsedAst, sourceCode, options).then(result => {})
```
> transformXxx 的 api，已经被标记为过时了，直接用 transformXxxSync 和 transformXxxAsync。

