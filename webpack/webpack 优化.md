
`chain-webpack` 提供了


## 打包瓶颈分析

1. speed-measure-webpack-plugin

可视化 webpack 构建期间各个阶段花费的时间

```bash
pnpm i speed-measure-webpack-plugin -D   
```

```js

const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
// ...
module.exports = smp.wrap(prodWebpackConfig)

// chain-webpack
const wrappedConfig = smp.wrap(config.toConfig());
config.toConfig = () => wrappedConfig;

```


1. `hard-source-webpack-plugin
```js
config
	.plugin('HardSourceWebpackPlugin')
	.use(new HardSourceWebpackPlugin());
```


