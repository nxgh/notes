## pnpm 的三层寻址

每个包的寻找都要经过三层结构：`node_modules/package-a` > 软链接 `node_modules/.pnpm/package-a@1.0.0/node_modules/package-a` > 硬链接 `~/.pnpm-store/v3/files/00/xxxxxx`

- 所有 npm 包都安装在全局目录 `~/.pnpm-store/v3/files` 下，同一版本的包仅存储一份内容，甚至不同版本的包也仅存储 diff 内容。
- 每个项目的 `node_modules` 下有 `.pnpm` 目录以打平结构管理每个版本包的源码内容，以硬链接方式指向 pnpm-store 中的文件地址。
- 每个项目 `node_modules` 下安装的包结构为树状，符合 node 就近查找规则，以软链接方式将内容指向 `node_modules/.pnpm` 中的包。

### 目的

1. 第一层寻找依赖是 `nodejs` 或 `webpack` 等运行环境/打包工具进行的，他们的在 `node_modules` 文件夹寻找依赖，并遵循就近原则,
2. 第二层的 `node_modules/package-a` > 软链接 `node_modules/.pnpm/package-a@1.0.0/node_modules/package-a` 寻址利用软链接解决了代码重复引用的问题
3. 第三层映射 `node_modules/.pnpm/package-a@1.0.0/node_modules/package-a` > 硬链接 `~/.pnpm-store/v3/files/00/xxxxxx` 已经脱离当前项目路径，指向一个全局统一管理路径了

## 幻影依赖

幻影依赖是指，项目代码引用的某个包没有直接定义在 `package.json` 中，而是作为子依赖被某个包顺带安装了。
代码里依赖幻影依赖的最大隐患是，对包的语义化控制不能穿透到其子包，也就是包 `a@patch` 的改动可能意味着其子依赖包 `b@major` 级别的 Break Change

三层寻址的设计，使得第一层可以仅包含 `package.json` 定义的包，使 node_modules 不可能寻址到未定义在 `package.json` 中的包

还有一种更难以解决的幻影依赖问题，即用户在 Monorepo 项目根目录安装了某个包，这个包可能被某个子 Package 内的代码寻址到，要彻底解决这个问题，需要配合使用 Rush，在工程上通过依赖问题检测来彻底解决


## peer-dependences 安装规则
[[package.json#peer-dependencies]]

`pnpm` 对 `peer-dependences` 有一套严格的安装规则

对于定义了 `peer-dependences` 的包来说，意味着为 `peer-dependences` 内容是敏感的，潜台词是说，对于不同的 `peer-dependences`，这个包可能拥有不同的表现，因此 `pnpm` 针对不同的 `peer-dependences` 环境，可能对同一个包创建多份拷贝。
比如包 `bar` `peer-dependences` 依赖了 `baz^1.0.0` 与 `foo^1.0.0`，那我们在 Monorepo 环境两个 Packages 下分别安装不同版本的包会如何呢？
```bash
- foo-parent-1  
  - bar@1.0.0  
  - baz@1.0.0  
  - foo@1.0.0  
- foo-parent-2  
  - bar@1.0.0  
  - baz@1.1.0  
  - foo@1.0.0
```
结果是这样（引用官网文档例子）：
```
node_modules  
└── .pnpm  
    ├── foo@1.0.0_bar@1.0.0+baz@1.0.0  
    │   └── node_modules  
    │       ├── foo  
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar  
    │       ├── baz   -> ../../baz@1.0.0/node_modules/baz  
    │       ├── qux   -> ../../qux@1.0.0/node_modules/qux  
    │       └── plugh -> ../../plugh@1.0.0/node_modules/plugh  
    ├── foo@1.0.0_bar@1.0.0+baz@1.1.0  
    │   └── node_modules  
    │       ├── foo  
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar  
    │       ├── baz   -> ../../baz@1.1.0/node_modules/baz  
    │       ├── qux   -> ../../qux@1.0.0/node_modules/qux  
    │       └── plugh -> ../../plugh@1.0.0/node_modules/plugh  
    ├── bar@1.0.0  
    ├── baz@1.0.0  
    ├── baz@1.1.0  
    ├── qux@1.0.0  
    ├── plugh@1.0.0
```
安装了两个相同版本的 `foo`，虽然内容完全一样，但却分别拥有不同的名称：`foo@1.0.0_bar@1.0.0+baz@1.0.0`、`foo@1.0.0_bar@1.0.0+baz@1.1.0`。

## 软链接与硬链接
[[软连接与硬链接]]

## 全局  pnpm-store

`pnpm` 在第三层寻址时采用了硬链接方式，这个硬链接目标文件并不是普通的 NPM 包源码，而是一个哈希文件，这种文件组织方式叫做 *content-addressable*（基于内容的寻址)

pnpm-store 的组织方式
```
~/.pnpm-store  
- v3  
  - files  
    - 00  
      - e4e13870602ad2922bfc7..  
      - e99f6ffa679b846dfcbb1..  
      ..  
    - 01  
      ..  
    - ..  
      ..  
    - ff  
      ..
```

NPM 包一经发布内容就不会再改变，因此适合内容寻址这种内容固定的场景，同时内容寻址也忽略了包的结构关系，当一个新包下载下来解压后，遇到相同文件 Hash 值时就可以抛弃，仅存储 Hash 值不存在的文件


## 切换到 pnpm 的一些问题

- 使用 `pnpm install --shamefully-hoist`
如果依赖一直有问题，可以使用 `pnpm install --shamefully-hoist` 创建一个扁平 node_modules 目录结构, 类似于 npm 或 yarn
- 解决幽灵依赖时，安装默认的包导致报错
先使用 npm 安装，生成 package-lock.json, 安装缺少的包时，使用 lock 里面的版本
- 即使删除了 node_modules 和 lock 文件，安装时，特定的包还是报错
比如我们在升级时，一个包把最新的版本删除了。导致安装时一直失败。可以尝试使用 pnpm store prune 来删除

[精读《pnpm》](https://mp.weixin.qq.com/s/xxNrtwGgjXLihkUmZ1sGzw)
