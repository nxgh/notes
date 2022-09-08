

## package.json

### dependencies

dependencies字段中声明的是项目的生产环境中所必须的依赖包


# ## peer-dependencies

不同于 dependencies 与 dev-dependencies, peer-dependencies  不会被自动安装

一个典型的插件场景:
A模块是B模块的插件。用户安装的B模块是1.0版本，但是A插件只能和2.0版本的B模块一起使用。这时，用户要是将1.0版本的B的实例传给A，就会出现问题。因此，需要一种机制，在模板安装的时候提醒用户，如果A和B一起安装，那么B必须是2.0模块

```json
"name": "chai-as-promised",  
"peerDependencies": {  
   "chai": "1.x"  
}
```
上面代码指定在安装chai-as-promised模块时，主程序chai必须一起安装，而且chai的版本必须是1.x。如果项目指定的依赖是chai的2.0版本，就会报错

[关于前端大管家 package.json，你知道多少？](https://mp.weixin.qq.com/s/Np-tDI84_VTJPHAIAl8aGQ)

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

