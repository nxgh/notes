
## Eslint 生态

### 规则

#### babel-eslint 
该依赖包允许你使用一些实验特性的时候，依然能够用上Eslint语法检查

#### @typescript-eslint/pareser
typescript语法的解析器，类似于`babel-eslint`解析器一样

#### eslint-config-airbnb
该包提供了所有的Airbnb的ESLint配置, 该包提供了所有的Airbnb的ESLint配置

- `eslint` 
- `eslint-plugin-import`
- `eslint-plugin-react`
- `eslint-plugin-react-hooks`
- `eslint-plugin-jsx-a11y`

#### eslint-config-airbnb-base
该包不包含 react 的规则，一般用于服务端检查

#### eslint-config-jest-enzyme
jest和enzyme专用的校验规则，保证一些断言语法可以让Eslint识别而不会发出警告。

#### eslint-config-prettier 
将会禁用掉所有那些非必须或者和 `prettier` 冲突的规则, 该配置只是将规则**off**掉,所以它只有在和别的配置一起使用的时候才有意义

### Plugins

#### eslint-plugin-babel 
与 `babel-eslint`配合使用，`babel-eslint` 不能更改内置规则来支持实验性特性

#### eslint-plugin-import

支持对ES2015+ (ES6+) import/export语法的校验, 并防止一些文件路径拼错或者是导入名称错误的情况

#### eslint-plugin-jsx-ally 
检查JSX元素的可访问性

#### eslint-import-resolver-webpack
借助webpack的配置来辅助eslint解析,比如 alias
类似的还有 [eslint-import-resolver-typescript](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falexgorbatchev%2Feslint-import-resolver-typescript "https://github.com/alexgorbatchev/eslint-import-resolver-typescript")

#### eslint-plugin-react

React专用的**校验规则插件**

#### eslint-plugin-jest

*Jest* 专用的 *Eslint* 规则校验插件
 
#### eslint-plugin-prettier

配合 *Eslint* 可以平滑地与 *Prettier* 一起协作，并将 *Prettier* 的解析作为 *Eslint* 的一部分，在最后的输出可以给出修改意见

如果你禁用掉了所有和代码格式化相关的Eslint规则的话，该插件可以更好得工作。

可以使用 `eslint-config-prettier`禁用掉所有的格式化相关的规则
#### @typescript-eslint/eslint-plugin
Typescript辅助Eslint的插件。

#### eslint-plugin-promise

promise规范写法检查插件，附带了一些校验规则。


## Prettier

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