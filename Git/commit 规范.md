## git commit message 规范

```bash
<type>(<scope>): <subject>
```

1.  tye: 用于说明git commit的类别类型
2.  scope: commit 影响的范围, 比如: route, component, utils, build...
3.  subject: commit 的概述
4.  body: commit 具体修改内容, 可以分为多行.
5.  footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.

命令行可搭配 `Commitizen`使用, vscode 可使用 `git-commit-plugin`


## Type 类型

- feat: 新功能、新特性
- fix: 修改 bug, 适合于一次提交直接修复问题
- To：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix
- perf: 更改代码，以提高性能
- refactor: 代码重构（重构，在不影响代码内部行为、功能下的代码修改）
- docs: 文档修改
- style: 代码格式修改, 不影响代码运行的变动
- test: 测试用例新增、修改
- build: 影响项目构建或依赖项修改
- revert: 恢复上一次提交
- ci: 持续集成相关文件修改
- chore: 其他修改（不在上述类型中的修改）
- release: 发布新版本
- workflow: 工作流相关文件修改

## Git Emoji

💄 `:lipstick:` 更新 UI 与样式文件.
🎉 `:tada:` 首次提交.
🚧 `:construction:` 正在开发中.
♻️ `:recycle:` 重构代码.
💩 `:hankey:` 提交了一些"坏"代码，后续需要改进.
👽 `:alien:` 由于使用的外部 API 变化而需要更新代码.
💥 `:boom:` 本次 API 或者代码拥有某些重大的变化，会导致无法向下兼容.
✨ `:sparkles:` 新的功能与特性.
🌐 `:globe_with_meridians:` 针对多语言本地化的一些更改.
📱 `:iphone:` 响应式设计相关变更.
🐛 `:bug:` 修复了某些 Bug.
🚑 `:ambulance:` 重要的问题修复.
✏️ `:pencil2:` 修正拼写错误.
🔒 `:lock:` 修复安全相关的问题.
🐧 `:penguin:` 修复了在 Linux 系统上的某些问题.
🏁 `:checkered_flag:` 修复了在 Windows 系统上的某些问题.
🤖 `:robot:` 修复了在 Android 系统上的某些问题.
🍏 `:green_apple:` 修复了在 IOS 系统上的某些问题.
⚡️ `:zap:` 性能提升.
🎨 `:art:` 改进代码的结构与格式.
💬 `:speech_balloon:` 更改了某些注释文本和字符串数据.
🚚 `:truck:` 移动/重命名文件.
🏗 `:building_construction:` 更改架构.
📝 `:memo:` 增加项目文档.
💡 `:bulb:` 编写源码相关的文档.
🔇 `:mute:` 移除日志.
🚨 `:rotating_light:` 移除 Linter 的警告.
✅ `:white_check_mark:` 添加测试用例.
⬇️ `:arrow_down:` 依赖项版本降级.
⬆️ `:arrow_up:` 升级依赖项版本.
📌 `:pushpin:` 固定依赖项版本.
➖ `:heavy_minus_sign:` 移除一个依赖项.
➕ `:heavy_plus_sign:` 添加一个依赖项.
🔧 `:wrench:` 配置文件的相关更改.
📦 `:package:` 更新已编译的文件或包.
🍱 `:bento:` 添加或者更新某些资源文件.
🗃 `:card_file_box:` 执行了数据库相关的一些更改，例如 EF 迁移.
⏪ `:rewind:` 回滚更改.
🚀 `:rocket:` 项目部署相关的提交.
🔖 `:bookmark:` 给代码增加版本化的 Tag(标签).
🔀 `:twisted_rightwards_arrows:` 合并分支.
💚 `:green_heart:` 修复 CI 构建相关的问题.
👷 `:construction_worker:` 添加 CI 构建系统.
📈 `:chart_with_upwards_trend:` 添加分析/跟踪代码.
🐳 `:whale:` 与 Docker 相关的某些更改.
🍻 `:beers:` 醉醺醺地编写代码，或许这些代码会有某些问题.
🤡 `:clown_face:` Mock 测试相关 🥚 增加了彩蛋.
👌 `:ok_hand:` 为了代码审查而进行的某些更改.
♿️ `:wheelchair:` 提升无障碍体验.
📸 `:camera_flash:` 添加或者更新了快照文件.
🙈 `:see_no_evil:` 增加或者更新了 .gitignore 文件.
📄 `:page_facing_up:` 添加或者更新许可.
👥 `:busts_in_silhouette:` 新增贡献者.
🚸 `:children_crossing:` 提升用户体验.