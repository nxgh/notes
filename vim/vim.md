- [LearnVim](https://github.com/iggredible/Learn-Vim)
- [Vim 命令速查表](https://www.cnblogs.com/chloneda/p/vim-cheatsheet.html)
- https://juejin.cn/post/6940261524216905736

## 0. 学习 Vim 的目的

1. 操作的一致性： 无论是 IDEA、VSCode、Obsidian、Chrome 都提供了 Vim 映射，允许你使用同一套操作
2. 远程编辑器：即使有了类似 VSCode Remote SSH 这样的插件, 部分场景还是需要 Vim


> 使用 neovim, neovim 是 vim 的兼容版本，提供了更现代化的插件系统


## 1. Vim: 如何上手

0. 只需掌握 20% 即可熟悉 Vim

- 最常用的功能是 模糊搜索

1. Vim 中只有一种语法 `Verb + noun`

Nouns 名词 表示动作 (Motions) `hjkl{}web$` 等

- Motions 接受 number 作为参数 `3h 3j y2h`
  Verbs 动词 表示运算符 (Operator)

> 使用 `:h operator` 查看 , Vim 中有 16 个 Operator, 常用的这三个便够了

```sh
y    # Yank text (copy)
d    # Delete text and save to register
c    # Delete text, save to register, and start insert mode
```

- 动词
  - `i` insert
  - `a` append
  - `c` change
  - `r` replace
  - `d` delete
  - `y` yank
  - `p` paste
  - `f` find
  - `v` visual
- 介词
  - `i` in
  - `a`
  - `t` to
  - `f` forward
- 名词
  - `w` word
  - `p` paragraph
  - `t` tag
  - `s` sentence
  

```
cib  # change in bracket
daw  # delete a word
```

常用的对行操作都是连续键入两次操作符，例如

- `dd` 删除整行
- `cc` 更改整行
- `yy` 复制整行

建议： 0. 不要纠结精细的移动光标

1. 不要去背快捷键

   - 要在使用过程中检测不够高效的地方，在哪里浪费太多时间
   - 去找更高效的办法, 然后习惯它

2. 关于插件：

   目前使用 VSCode 搭配 Vim 插件的形式，Vim 的插件只有 `surround` 这一个，当配合编辑器使用时几乎可以不用到插件

## 快速上手

### 1. 在光标定位上超过鼠标

- 减少 `h j k l` 的使用频率, 只在有限行或单词内使用

  1.2 大段的移动光标

-「行」为单位的移动 `w e b W E B $ 0 ^ | `

- 符号间的移动 `%`
- 段落间的移动 `{ } ( ) gg G nG ngg`
- 相对于屏幕的移动 - `Ctrl e y u d b f`、 `H M L` 、 `zz zt zb`
- 使用搜索来进行移动 - `/ ? n N `、 `f F t T ; .`

vim 的使用不要拘泥于这些，例如 `^` 在实际使用中并不方便，可以使用 `0 w` 替代

> 可以使用 `:set relativenumber` 来显示相对行号，跳转更加方便，`:set number` 显示绝对行号

## 2. 高效编辑

### Text Object

vim 中，文本是结构化的存在，有一部分语法是专门用于处理文本对象的

文本对象的语法结构一般为 `operator + i/a + text object`

- `i` inner 内部
- `a` a 一个

文本对象主要有

```
() b
{} B
[]
<>
tag
''
""
``
w    # word
s    # sentence
p    # paragraph
```

🌰

- `viw` 快速选中光标所在的单词，无论光标单词的位置
- `vi(` 光标在 `()` 内时， 选中 `()` 内的内容
- `va{` 选中内容包括 `()`

### Vim Surround

了解了 Text Object 后，可以搭配 vim - surround 快速处理成对符号

vim surround 提供了 `s` Operator

```sh
ys   # 添加环绕字符
yss  # 为整行添加环绕字符
cs   # 替换环绕字符
ds   # 删除环绕字符

yS   # 同 ys, 拆分新行
ySS  # 同 yss, 拆分新行
```

| 源文本               | 操作             | 结果                   |
| -------------------- | ---------------- | ---------------------- |
| `Hello`              | `ysw"`           | `"Hello"`              |
| `"Hello"`            | `cs"'`           | `'Hello'`              |
| `Hello`              | `ysiw[`          | `[ Hello ]`            |
| `Hello`              | `ysiw]`          | `[Hello]`              |
| `Hello`              | `cs]}` 或 `cs]B` | `{Hello}`              |
| `Hello World`        | `ysiw<em>`       | `<em>Hello</em> World` |
| `Hello World`        | `ysiw<em>`       | `<em>Hello</em> World` |
| `<p>Hello</p> World` | `dst`            | `Hello World`          |
| `[Hello]`            | `ds]`            | `Hello`                |

以下是 Virtual 模式下的操作, 光标选中单词

| 源文本 | 操作 | 结果 |
| ---- | ---- | ---- |
| `Hello` | `S"` | `"Hello"` |


```
hello world
-> VS}
{
hello world
}
```

### 文本编辑

- 编辑操作 `r s x R S X` 、 `i a o I A O`、`gi`
- 复制粘贴 `p P y`

搭配 TEXT OBJECT `ci/di/vi/yi` `'"({[<t`

命令模式

```sh
:3,5t.              # 把第 3 行到第 5 行的内容复制到当前行下方
:t5                 # 把当前行复制到第 5 行下方
:t.                 # 复制当前行到当前行下方（等价于普通模式下的 yyp）
:t$                 # 把当前行复制到文本结尾
:'<,'>t0            # 把高亮选中的行复制到文件开头
```

#### 批量修改



## 正则表达式

## 缓冲区

## 寄存器

## 宏

## 进行复杂文本处理

## 配置 vscode 与 vim

0. 使用 `vimrc` 配置 vscode vim

```json
"vim.vimrc.enable": true,
"vim.vimrc.path": "~/.vimrc"
```

1. 将右 `shift` 映射为 `ESC`

```json
"vim.insertModeKeyBindings": {
  {
    "before": {"<R-Shift>"},
    "after": {"<Esc>"},
    "vimMode": "Insert"
  }
}
```

## 插件: surround

surround 可以处理

```
() {} [] <> `` "" ''
```

1. 开启

- `:set surround`

在 NORMAL 模式下在选中两端添加符号

```
Hello  ysw"  "Hello"
```

```
const foo => {
	bar: 'bar'
}


const foo => ({
	bar: 'bar'
})
```

### 插入模式下执行的命令

- 缩进
  - `ctrl + d` 减少
  - `ctrl + t` 增加
- 删除
  - `ctrl + h`删除光标前的字符
  - `ctrl + u` 当前行删除到行首所有字符
  - `ctrl + w`删除光标前的一个单词
- 临时退出
  - `ctrl + o`临时退出插入模式，执行单条命令又返回插入模式
  - `ctrl + \ ctrl + 0`临时退出插入模式（光标保持），执行单条命令又返回插入模式
- 寄存器
  - `Ctrl+R 0` 插入寄存器（内部 0 号剪贴板）内容，Ctrl+R 后可跟寄存器名
  - `Ctrl+R "` 插入匿名寄存器内容，相当于插入模式下 p 粘贴
  - `Ctrl+R =` 插入表达式计算结果，等号后面跟表达式
  - `Ctrl+R :` 插入上一次命令行命令
  - `Ctrl+R /` 插入上一次搜索的关键字
- `Ctrl + x/a` 对数字执行加/减操作, 可以携带次数前缀 `18<Ctrl+x>`

## 使用技巧

1. 快速定位 `f` 搭配 `;` `,` - `t`
2. 在 Virtual 模式下选中多行，键入 `Shift + i` 进入批量编辑

