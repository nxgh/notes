#代码规范 #husky #eslint #prettier #editorconfig

## TL;DR

- EditorConfig: 解决了编辑器配置层面的编码风格一致性问题。
- Prettier: 统一代码风格(Code Formatter)
- ESLint: 专注于找到代码存在的问题避免错误
- Git hook: 在合适的时机自动执行指定任务
- Pre-commit: 多语言的框架，能作用于所有git hooks的所有阶段
- lint-staged: 将格式化处理范围限制在 Git 暂存区内 (staged) 的文件
- husky: 非常方便地添加 git hook
- 解决 ESLint 和 Prettier 职责重叠部分的冲突？


## Editorconfig

统一代码编辑器编码风格

项目中共享 `.editorconfig` 文件，该文件可被 EditorConfig 解析，由 EditorConfig 告知编辑器覆盖默认配置

一些编辑器内置支持 EditorConfig，比如 WebStorm、Github；而一些编辑器需要安装 EditorConfig 插件，比如 VSCode、Sublime

```txt
indent_style    设置缩进为 tab 或 space
tab_width       设置 tab 所占列数。默认是indent_size
indent_size     设置缩进所占列数，如果 indent_style 为 tab，则以 tab_width 值作为缩进宽度
end_of_line     设置换行符，值为lf、cr和crlf
charset         设置编码，值为latin1、utf-8、utf-8-bom、utf-16be和utf-16le，不建议使用utf-8-bom
trim_trailing_whitespace  设为 true 表示会去除行尾的空白字符
insert_final_newline      设为 true 表示使文件以一个空白行结尾
root        　　　表示是最顶层的配置文件，设为 true 时，停止向上查找
```

示例
```
# https://editorconfig.org

# 已经是顶层配置文件，不必继续向上搜索
root = true

[*] 
indent_style = space            # 缩进风格是空格
indent_size = 4       		      # 一个缩进占用两个空格，因没有设置tab_with，一个Tab占用4列
end_of_line = lf         			  # 换行符 lf
charset = utf-8         		    # 编码字符集
trim_trailing_whitespace = true # 去除行首的任意空白字符
insert_final_newline = true     # 文件以一个空白行结尾

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

## Prettier

Prettier 通过语法分析将代码解析为 AST 树，在 AST 树上应用代码风格规范重新生成代码
[prettier playground](https://prettier.io/playground/)

Prettier 提供如下配置项

```ts
interface Config {
    printWidth: number, // 打印宽度，默认是 80 列
    tabWidth: number, // 缩进所占列数，默认是 2 列
    useTabs: boolean, // 缩进风格是否是Tab，默认是 false ，使用空格缩进
    semi: boolean, // 在语句末尾添加分号，默认是 true
    singleQuote: false, // 使用单引号，默认是 false
    // 默认 "as-needed" 只对需要的属性加引号，
    // "consistent" 同一对象中属性引号保持统一，有福同享，有难同当
    // "preserve" 强制使用引号。
    quoteProps: "as-needed" | "consistent" | "preserve"  // 对象中的属性使用引号，
    jsxSingleQuotes: false, // JSX中使用单引号，默认是 false
    //  默认 "es5" ES5中允许逗号的容器中添加逗号，比如 objects/arrays
    // "all" 尽可能添加逗号  "none" 不允许添加逗
    trailingComma: "es5" | "all" | "none", // 多行时是否结尾添加逗号
    bracketSpacing: boolean, // 是否保留对象内侧两端的空格，比如 { foo: bar } 和 {foo:bar} 的区别
    jsxBracketSameLine: boolean, // 多行 JSX 的元素是否能和属性同行，默认是 false
    arrowParens: "always | avoid", // 箭头函数参数使用圆括号包裹 比如 (x) => x 和 x => x 的区别
    rangeStart: number, // 只格式化文件中的一部分，范围开始于第几行
    rangeEnd: Infinity, // 只格式化文件中的一部分，范围结束于第几行
    // 指定解析器，Prettier会根据文件路径推断解析器
    // 比如 .js 文件使用 babel 解析，.scss 文件使用 post-scss 解析
    parser: "none"
    filepath: "none" // 指定用于推断使用那个解析器的文件名
    // 适用于在大型未格式化项目中，先指定少量文件格式化
    requirePragma: false // 限制只格式化在文件顶部做了需格式化标识的文件 
    insertPragma: false
    proseWrap: "preserve"
    htmlWhitespaceSensitivity: "css" | "strict" | "ignore" // HTML 文件的空格敏感度
    // "css" 和 css 的 display 属性保持一致
    // "strict" 空格敏感
    // "ignore" 空格不敏感
    vueIndentScriptAndStyle: false // 是否对 Vue 文件中 <script> 和 <style> 标签内的代码应用缩进
    endOfLine: "lf" // 换行符
    // "auto" Prettier 自动识别并格式化
    // "off" 关闭自动格式化
    embeddedLanguageFormatting: "auto" | "off" // 是否格式化嵌入引用代码，比如 markdown 文件中嵌入的代码块
}
```

Prettier 使用 cosmiconfig 支持配置文件，cosmiconfig 是一种常用的配置文件读取工具，按照下述顺序沿文件树寻找配置文件，找到则停止：

- package.json 中的 prettier 字段
- `.prettierrc` 文件
- `.prettierrc.json` 文件
- `.prettierrc.js` 文件
- `.prettierrc.toml` 文件

选择上述任一方式进行自定义配置 Prettier，如不存在配置文件，Prettier 将依照默认值处理。

## Eslint 

### Eslint 生态

### 规则

- `babel-eslint` 

该依赖包允许你使用一些实验特性的时候，依然能够用上Eslint语法检查

- `@typescript-eslint/pareser`

typescript语法的解析器，类似于`babel-eslint`解析器一样

- `eslint-config-airbnb`
该包提供了所有的Airbnb的ESLint配置, 该包提供了所有的Airbnb的ESLint配置
    - `eslint` 
    - `eslint-plugin-import`
    - `eslint-plugin-react`
    - `eslint-plugin-react-hooks`
    - `eslint-plugin-jsx-a11y`
- `eslint-config-airbnb-base`

该包不包含 react 的规则，一般用于服务端检查
- `eslint-config-jest-enzyme`

jest和enzyme专用的校验规则，保证一些断言语法可以让Eslint识别而不会发出警告。
- `eslint-config-prettier` 

将会禁用掉所有那些非必须或者和 `prettier` 冲突的规则, 该配置只是将规则**off**掉,所以它只有在和别的配置一起使用的时候才有意义
### Plugins
- `eslint-plugin-babel` 

与 `babel-eslint`配合使用，`babel-eslint` 不能更改内置规则来支持实验性特性
- `eslint-plugin-import`

支持对ES2015+ (ES6+) import/export语法的校验, 并防止一些文件路径拼错或者是导入名称错误的情况
- `eslint-plugin-jsx-ally `

检查JSX元素的可访问性
- `eslint-import-resolver-webpack`

借助webpack的配置来辅助eslint解析,比如 alias
类似的还有 [eslint-import-resolver-typescript](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falexgorbatchev%2Feslint-import-resolver-typescript "https://github.com/alexgorbatchev/eslint-import-resolver-typescript")

- `eslint-plugin-react`

React专用的校验规则插件
- `eslint-plugin-jest`

Jest 专用的 Eslint 规则校验插件
- `eslint-plugin-prettier`

配合 *Eslint* 可以平滑地与 *Prettier* 一起协作，并将 *Prettier* 的解析作为 *Eslint* 的一部分，在最后的输出可以给出修改意见

如果你禁用掉了所有和代码格式化相关的Eslint规则的话，该插件可以更好得工作。

可以使用 `eslint-config-prettier`禁用掉所有的格式化相关的规则
- `@typescript-eslint/eslint-plugin`

Typescript辅助Eslint的插件。

- `eslint-plugin-promise`

promise规范写法检查插件，附带了一些校验规则。


#### Prettier

- prettier
- prettier-eslint
- prettier-eslint-cli

1. 最基础配置 *prettier*，
2. 用 *eslint-config-prettier* 去禁用掉所有和 *prettier* 冲突的规则，这样才可以使用 *eslint-plugin-prettier*  去以符合eslint规则的方式格式化代码并提示对应的修改建议。
3. 为了让 *prettier* 和 *eslint* 结合起来，所以就诞生了 *prettier-eslint* 这个工具，但是它只支持输入代码字符串，不支持读取文件，因此又有了 *prettier-eslint-cli*


## Eslint 配置文件

```js
module.exports =  {
  parser:  'babel-eslint',  // Specifies the ESLint parser
  parserOptions: {
    ecmaVersion: 2015,
    sourceType:  'module',  // Allows for the use of imports
    ecmaFeatures: {
      jsx: true, // enable JSX
      impliedStrict: true // enable global strict mode
    }
  },
  extends:  [
    'airbnb',
    'plugin:promise/recommended',
  ],
  settings: {
    'import/resolver': {
      webpack: {
        config: './webpack/webpack-common-config.js'
      }
    },
  },
  env: {
    browser: true // enable all browser global variables
  },
  'plugins': ['react-hooks', 'promise'], // ['prettier', 'react-hooks']
  rules:  {
    "react-hooks/rules-of-hooks": "error",
    "semi": ["error", "never"],
    "react/jsx-one-expression-per-line": 0,
    /**
     * @description rules of eslint-plugin-prettier
     */
    // 'prettier/prettier': [
    //   'error', {
    //     'singleQuote': true,
    //     'semi': false
    //   }
    // ]
  },
};

```

## git hooks

[git hook](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)

通常为了保证项目的代码质量、以及更好地进行团队之间的协作，我们都会在提交代码时做一些额外的工作，包括：检查 commit message 的规范性、统一代码风格、进行单元测试等等。

大部分版本控制系统都会提供一个叫做钩子（Hooks）的东西, Hooks 可以让我们在特定的重要动作发生时触发自定义脚本，通常分为客户端和服务端，而我们接触的大部分 Hooks 都是客户端的

- 客户端钩子：pre-commit, prepare-commit-msg, commit-msg, post-commit等，主要在服务端接收提交对象时、推送到服务器之前调用。
- 服务器钩子：pre-receive, post-receive, update等，主要在服务端接收提交对象时、推送到服务器之前调用。

在这我们只会用到“提交工作流”钩子，提交工作流包含 4 个钩子：

1. `pre-commit` 在提交信息 编辑前 运行，在这个阶段塞入 代码检查 流程，检查未通过返回非零值即可停止提交流程；
2. `prepare-commit-msg` 在默认信息被创建之后运行，此时正是 启动编辑器前 ，可在这个阶段加载 commitizen 之类的辅助填写工具；
3. `commit-msg` 在 完成编辑后 运行，可在这个阶段借助 commitlint 进行提交信息规范性检查；
4. `post-commit` 在 提交完成后 运行，在这个阶段一般做一些通知操作。

在 Git 中使用 Hooks，我们只需要在项目的 `.git/hooks` 目录中创建一个**与某个 hook 同名的可执行脚本**即可。
最直观的方式是操作 .git/hooks 下的示例文件，将对应钩子文件的 .sample 后缀名移除即可启用。然而这种操作方式存在弊端：

1.  需要操作项目范围外的 .git 目录
2.  无法同步 .git/hooks 到远程仓库

> 在 git 文档中对客户端 hooks 有一段话:  
 需要注意的是，克隆某个版本库时，它的客户端钩子并不随同复制。 如果需要靠这些脚本来强制维持某种策略，建议你在服务器端实现这一功能。

## pre-commit

pre-commit 是一个多语言的框架，能作用于所有git hooks的所有阶段


## lint-staged

lint-staged 将格式化处理范围限制在 Git 暂存区内 (staged) 的文件

```javascript
options
  -V, --version                      输出版本号
  --allow-empty                      当任务撤消所有分阶段的更改时允许空提交（默认值：false）
  -c, --config [path]                配置文件的路径
  -d, --debug                        打印其他调试信息（默认值：false）
  -p, --concurrent <parallel tasks>  要同时运行的任务数，或者为false则要连续运行任务（默认值：true）
  -q, --quiet                        自己的控制台输出（默认值：false）
  -r, --relative                     将相对文件路径传递给任务（默认值：false）
  -x, --shell                        跳过任务解析以更好地支持shell（默认值：false）
  -h, --help                         输出用法信息
```

- `--allow-empty`: 默认情况下，当LITER任务撤消所有阶段性的更改时，LITET阶段将退出一个错误，并中止提交。使用此标志允许创建空git提交。
- `--config [path]` : 手动指定配置文件或npm包名称的路径。注意：使用时，lint-staged不会执行配置文件搜索，如果找不到指定的文件，则会打印错误。
- `--debug` :在调试模式下运行。设置后，它将执行以下操作
在内部使用debug记录有关暂存文件、正在执行的命令、二进制文件的位置等的其他信息。通过传递标志自动启用的调试日志也可以通过将环境变量$DEBUG设置为lint-staged*启用。
使用verbose渲染程序的listr; 这将导致串行无色输出到终端，而不是默认（美化，动态）输出。
- `--concurrent [number | (true/false)]`: 控制由lint-staged运行的任务的并发性。注意：这不会影响子任务的并发性（它们将始终按顺序运行）。可能的值为：
   - false:依次运行所有任务
   - true（默认）：无限并发。并行运行尽可能多的任务
   - {number}：并行运行指定数量的任务，其中1等效于false。
- `--quiet`:禁止所有CLI输出，但任务中除外。
- `--relative`: 将与process.cwd()（lint-staged运行）相关的文件路径传递给任务。默认值为false。
- `--shell`:默认情况下，将分析linter命令以提高速度和安全性。这具有常规shell脚本可能无法按预期工作的副作用。您可以使用此选项跳过命令解析


从v3.1开始,可以使用不同的方式进行配置
- package.json
```json
{
    "lint-staged": {
        "*": "cmd"
    }
}
```
- `.lintstagedrc`
```javascript
{
    "*": "cmd"
}
```

- `lint-staged.config.js`  
- 使用`--config`或`-c`标志传递配置文件

## husky >v4

非常方便地添加 git hook

- 安装

```bash
pnpm i husky -D
```

- 初始化

```bash
npm set-script prepare "husky install"
npm run prepare
```
利用 npm 的 prepare 钩子，其可以执行 npm publish 和 不带参数的 npm install 时执行

- 配置 hooks 
```bash
npx husky add .husky/pre-commit "npx lint-staged"
git add .husky/pre-commit
```

- 安装后自动启用hooks， 修改 `package.json`
```json
{
  "scripts":{
        "prepare":"husky install"
  }
}
```

- 添加hooks

```bash
npx husky add .husky/pre-commit "npm run pre-commit"
```

- 在package.json文件中添加pre-commit

```json
{
    "script":{
        "pre-commit":"npm run test && eslint"
    }
}
```

2、自定义.husky目录

```bash
npx husky install .config/husky
```

3、绕过钩子

```bash
git commit -m "test" --no-verify
```

husky 一共支持以下命令：

1.  husky install：安装，主要是配置 Git 的 core.hooksPath
2.  husky uninstall：卸载，主要是恢复对 Git 的 core.hooksPath 的修改
3.  husky set：新增 hook
4.  husky add：给已有的 hook 追加命令

工作原理

git 2.9开始引入的一个新功能 `core.hooksPath`。`core.hooksPath`可以让你指定`git hooks`所在的目录而不是使用默认的 `.git/hooks/``

husky可以使用husky install将git hooks的目录指定为.husky/，然后使用husky add命令向.husky/中添加hook


## eslint 

ESLint 的底层要素是 AST 和 规则（Rules）。ESLint 的内部工作步骤可以概括为：

1.  ESLint 通过解析器（parser）将源代码解析成 AST
2.  遍历 AST，遍历到节点和路径时触发特定的钩子
3.  Rule 在钩子上挂载检测逻辑；执行检测逻辑时发现当前语法不符合规范，直接向 ESLint 上报错误信息。


## 如何解决 ESLint 和 Prettier 职责重叠部分的冲突？

[eslint-config-prettier](https://zhuanlan.zhihu.com/p/366141969/%5Bprettier/eslint-config-prettier:%20Turns%20off%20all%20rules%20that%20are%20unnecessary%20or%20might%20conflict%20with%20Prettier.%20(github.com)%5D(https://github.com/prettier/eslint-config-prettier)) 关闭 ESLint 中和 Prettier 可能冲突的所有 Rules，eslint 负责代码质量检查，prettier 做 formatter；  
[eslint-plugin-prettier](https://link.zhihu.com/?target=https%3A//github.com/prettier/eslint-plugin-prettier) 该插件增加了 prettier/prettier 规则， 该规则执行 prettier 并将错误信息上报 eslint。简而言之，将 prettier 融合到 eslint 中，担起代码风格检查的功能，同时需要搭配 eslint-config-prettier 关闭掉 ESLint 中代码风格检查相关的规则。



[每周轮子之 husky：统一规范团队 Git Hooks](https://4ark.me/post/weekly-npm-packages-02.html)
