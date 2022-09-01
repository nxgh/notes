
## 版本管理器
fnm 
nvs
nvm

## 包管理器
[JavaScript 包管理器简史（npm/yarn/pnpm）](https://zhuanlan.zhihu.com/p/451025256)
[深入浅出 tnpm rapid 模式 - 如何比 pnpm 快 10 秒](https://zhuanlan.zhihu.com/p/455809528)

成熟方案
- npm 
- yarn
 - yarn pnp
新方案
- pnpm 
探索
- tnpm rapid - npminstall

扁平化方案带来的新问题
- **幽灵依赖问题**（[phantom dependencies](https://link.zhihu.com/?target=https%3A//rushjs.io/pages/advanced/phantom_deps)）。
- **多重身问题**，无法彻底解决重复依赖
- **依赖结构的不确定性**。（通过 依赖关系图 可以解决）
- 扁平化算法的复杂度和性能损耗。

pnpm 的方案: 软链接 + 硬链接


## npm 依赖升级

