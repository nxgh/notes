

## dependencies
dependencies字段中声明的是项目的生产环境中所必须的依赖包

## peer-dependencies

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