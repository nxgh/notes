- husky 用途主要是统一管理项目中的 Git Hooks 脚本
- husky 可以方便地添加 git hooks

## git hooks

通常为了保证项目的代码质量、以及更好地进行团队之间的协作，我们都会在提交代码时做一些额外的工作，包括：检查 commit message 的规范性、统一代码风格、进行单元测试等等。

大部分版本控制系统都会提供一个叫做钩子（Hooks）的东西, Hooks 可以让我们在特定的重要动作发生时触发自定义脚本，通常分为客户端和服务端，而我们接触的大部分 Hooks 都是客户端的

### 使用 git hooks

在 Git 中使用 Hooks，我们只需要在项目的 `.git/hooks` 目录中创建一个**与某个 hook 同名的可执行脚本**即可

```bash
# .git/hooks/commit-msg  
  
#!/usr/bin/env bash  

# 阻止一切提交，并将 commit message 打印到终端
INPUT_FILE=$1  
  
START_LINE=$(head -n1 $INPUT_FILE)  
  
echo "当前提交信息为：$START_LINE"  
  
echo "阻止此次提交！！！"  
  
exit 1
```

git hooks 的缺陷

> 在 git 文档中对客户端 hooks 有一段话: 
> 需要注意的是，克隆某个版本库时，它的客户端钩子并不随同复制。 如果需要靠这些脚本来强制维持某种策略，建议你在服务器端实现这一功能。

简单来说就是，我们上面添加的这个 `commit-msg` Hook，只能在我们自己的机器上，不能被加入到版本控制中推送到远端，也就意味着我们无法同步这些 Hooks 脚本

而 husky 解决了这个问题，同时提供了更加简便的方式来使用 git hooks

## husky(>v4) 安装

- 安装
```bash
npm i husky -D
```
- 初始化  
```bash
npm set-script prepare "husky install"  
npm run prepare
```
利用 npm 的 prepare 钩子，其可以执行 npm publish 和 不带参数的 npm install 时执行
- 配置 hooks
```bash
npx husky add .husky/pre-commit "npm test"  
git add .husky/pre-commit
```

 husky 一共支持以下命令：
1.  husky install：安装，主要是配置 Git 的 core.hooksPath
2.  husky uninstall：卸载，主要是恢复对 Git 的 core.hooksPath 的修改
3.  husky set：新增 hook
4.  husky add：给已有的 hook 追加命令


绕过 hooks 
```bash
git commit -m "test" --no-verify
```

[每周轮子之 husky：统一规范团队 Git Hooks](https://4ark.me/post/weekly-npm-packages-02.html)