<!--
key: 
 - 前端工程化
-->
# 前端工程化

> 工程化的最终目的: 「**让业务开发可以 100% 聚焦在业务逻辑上**」

> 目录设计者

目的：

- 提高开发效率
- 保持系统质量
- 降低成本: 开发、维护、运维

考虑

- 兼容性
- 难以发现的 bug

大致方向：

- 减少人工操作，能够自动化的尽量自动化

## 开始工作

## 技术选型

- 标准： 可控性 稳定性 适用性 易用性

- react、vue、angular、svelte、solidjs
- css:
  - 预处理：less、sass、stylus、postcss
  - 原子化: tailwindcss、windicss、unocss
  - css-in-js: styled-components、emotion、linaria、jss
- typescript

多人协作问题

## 项目维护

- monorepo
  - lerna
  - yarn/pnpm workspaces
  - nx、Rush...
- 微服务/微前端
- Serverless
- 文档规范

## 代码质量

统一规范

- typescript
- stylelint
- eslint
- prettier
- editorconfig

## code review

- 结对编程
- 相互审查

## Git

- 分支管理
  - merge rebase 规范
- commit 规范
  - husky
  - git hooks
  - commitlint

## 命名规范

- 目录命名
  - 业务相关特性: feature/
  - 全局通用组件: component/

UI 规范

- 统一命名
- 统一样式

## 开发

- 指引思想： 高内聚、低耦合。
- 代码的可读性、可维护性、可扩展性、可复用性

### 模块化、组件化

业务复用时遇到的问题：

- 不能满足需求
- 为了满足多个业务的复用需求，不得不把组件修改到很别扭的程度
- 参数失控
- 版本无法管理

组件化该如何做？

组件可以分为：有状态组件和无状态组件

- 有状态组件一般位于顶层，无状态组件则依赖于外部传入的状态与控制
- 有状态组件内部也可以分为两层：一层处理状态;一层处理渲染,这一层也是无状态组件
- 对于纯交互类组件，状态外置通常是更好的选择

上下文管理依赖项

- 避免直接引入全局的依赖，而是通过上下文管理依赖项

组合优于继承

- 业务状态与 UI 状态分离
- UI 状态与交互呈现隔离





### 状态管理

- 组件状态: 只有组件自身以及他的子孙组件需要这部分状态
- 应用状态
  - 状态尽可能靠近使用他的组件
- 服务端状态
  - 缓存失效、序列化数据
  - 尽量使用第三方工具，例如：swr/react-query/apollo client/urql + RESTful API/GraphQL
- 表单状态
  - 受控与非受控组件、表单验证
  - 推荐使用第三方工具，例如：react-hook-form/Formik
- URL 状态
  - url params/query params
  - 通常由路由库来管理

## 测试

- mock
- unit test
  - jest mocha vitest
- e2e test
  - cypress puppeteer selenium
- 测试覆盖率

## 构建

- webpack
- rollup
- vite
- snowpack
- esbuild
- swc
- Turbopack

## 部署 & 发布

- CI/CD
  - 需求单分配：分配并自动拉取相应 Git 分支
  - 代码提交：代码规范检查 + 自动化测试 + 部署测试环境 + 根据需求单配置通知相应的产品和测试
  - 产品验证/功能测试：BUG 单自动关联需求单和分支 + 验证完成后，根据需求单通知开发侧
  - BUG 修复：代码规范检查 + 自动化测试 + 部署测试环境 + 根据分支 BUG 单和需求单通知测试 + 验证完成后，根据需求单通知开发侧
  - 代码合入主干：向团队成员发起代码 Review + Review 通过后代码合入主干
  - 日常发布：定时器发起发布流程 + 预发布环境部署 + 进行自动化测试 + 测试通过后进入灰度过程
  - 灰度发布：根据配置比例进行灰度 + 灰度过程中自动化进行监控 + 可选择性进入快速回滚流程
  - 全量发布：自动扭转需求单状态，并将版本进行归档（Git Tag）
- Jenkins
- GitHub Actions

## 异常监控

- 埋点
- Sentry
- js error
  - addEventListener
  - window.onerror
  - promise: unhandledrejection
- http error
- 行为数据采集: PV、UV、页面停留时长、页面访问深度、点击、跳转、路由变更
- sourcemap

## 性能优化

- 检测网页性能的指标: FPS、FCP、LCP、CLS
- 主要分为：加载时优化、运行时优化
- 减少 http 请求、使用 http2
- SSR
- CDN
- iconfont 代替图片图标
- 缓存
- 压缩
- gzip
- 图片懒加载、降低体积，使用 webp、使用 css 代替
- 按需加载、webpack externals

开发优化

- 大量分支判断使用 switch case 或查找表
- 减少循环嵌套
- 使用 requestAnimationFrame
- 减少 DOM 操作、减少重绘重排
- css 选择器优化

## 缓存

## 重构

前提：

- 有完善的测试用例
- 当代码重复超过三次时，就应该考虑重构（事不过三）
- 可读性变差时
- 代码不够简洁时
- 违背单一职责原则时

- 提取重复代码，封装成函数
- 拆分功能太多的函数
- 变量/函数改名
- 替换算法
- 以函数调用取代内联代码
- 移动语句
- 折分嵌套条件表达式
- 将查询函数和修改函数分离

[带你入门前端工程](https://woai3c.gitee.io/introduction-to-front-end-engineering)

## 通用简洁架构

- 领域 domain
- 用例 use case
- 应用层 application layers
