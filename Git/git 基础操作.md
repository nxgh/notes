[github 镜像站 https://hub.fastgit.org/](https://hub.fastgit.org/)

## git  config

### proxy 

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global --unset http.proxy
```

### username & email 

```bash
# 全局配置
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"

# 查看配置信息
git config --global --list

# 生成 ssh-key
ssh-keygen -t rsa -C "youremail@example.com"
```
### 单个仓库配置

```bash
# 在项目主目录下执行
git config user.name "Your Name"
git config user.email "email@example.com"
# 配置
git config credential.helper store
# 重新输入一次用户名和密码
git pull # or ...
```

### 使用 ssh-key

```bash
ssh-keygen -t rsa -C 'xxxxx@company.com'
```

配置多个 SSH-Key

```bash
$ ssh-keygen -t rsa -C 'xxxxx@company.com' -f ~/.ssh/gitee_id_rsa
$ ssh-keygen -t rsa -C 'xxxxx@example.com' -f ~/.ssh/github_id_rsa
```

```bash
# git服务器的域名，IdentityFile指定私钥的路径
# gitee
Host gitee.com    
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```
测试
```bash
$ ssh -T git@gitee.com
$ ssh -T git@github.com
```

### 使用 gum 配置用户信息
[https://github.com/gauseen/gum](https://github.com/gauseen/gum)
## git commit 提交

```bash
git commit -a --allow-empty-message -m ""
```

## git log  查看历史记录
```bash
git log
# 一行显示
git log --pretty=oneline

# 记录每一次的命令
git reflog

# 
git log --pretty=oneline --abbrev-commit
```
## git reset 回退版本
```bash
# 回退到上一个版本
git reset --hard HEAD^ 

# 回退到上 100 个版本
git reset --hard HEAD~100 
```

- 撤销回退
```bash
git reflog
git reset --hard commit_id
```
## git restore 丢弃修改
```bash
# 丢弃工作区的修改
git restore filename

# 旧版命令
# git checkout -- filename

# 丢弃暂存区的修改
git restore --staged 

# 旧版命令 
# git reset HEAD filename
```
## git rm 删除文件
```bash
git rm filename

git commit -m "xx"
```
## git remote 添加远程库
```bash
git remote add origin url

# 推送, -u 把本地的master分支和远程的master分支关联起来
git push -u origin master
```
### 远程分支

查看是否和远端的仓库建立连接
```bash
git remote -v
```

没有则添加
```bash
git remote add origin 远程分支的git仓库地址
```


拉取远程分支到本地并切换
```bash
# 在本地创建和远程分支对应的分支, 名称最好一致
git checkout -b dev origin/dev

# 指定本地dev分支与远程origin/dev分支的链接
git branch --set-upstream-to=origin/dev dev
```
### 更换远程仓库地址

1.  直接修改
```javascript
git remote set-url origin [url]
```

2. 删除后添加
```javascript
git remote rm origin
git remote add origin 你的新远程仓库地址
```

3. 修改 `.git/config` 
## git branch/switch 创建分支
```bash
# 创建并切换到 dev 分支
git branch -b dev
git switch -c dev

# 相当于
# git branch dev
# git switch dev
```
**查看分支**
```bash
git branch

## 
git branch -r # 列出远程分支 
git branch -a # 列出所有分支
```
**切换分支**
```bash
git switch dev

# 旧版命令
# git checkout dev
```

**合并分支**
```bash
# 将 dev 分支与 master 分支合并
git merge dev

# --no-ff 禁用 Fase forward
# 本次合并要创建一个新的commit，
git merge --no-ff -m " " dev
```

**删除分支**
```bash
git branch -d dev

# 删除一个没有被合并过的分支
git branch -D never-merge-dev
```
**将本地分支推送到远程**
```bash
git push origin dev:dev


git branch --set-upstream-to=origin/dev dev
将本地分支与远程进行关联，origin/dev是你本地分支对应的远程分支，dev是你当前的本地分支。未关联会，git会有提示：

删除远程分支
git push origin --delete [branch_name]
```
## git stash 储藏

- 当代码未完成而又不得不切换分支时，可以使用 `git stash` 
- stash，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作
- `git stash` 可以将你当前未提交到本地（和服务器）的代码推入到Git的栈中，这时候你的工作区间和上一次提交的内容是完全一样的
```bash
# 保存当前所有未提交修改
git stash 

# 使用 save 记录版本
git stash save "test-cmd-stash"

# 缓存工作目录中的新文件
# git stash --include-untracked
git stash -u

# 缓存被忽略的文件
# git stash -all
git stash -a


# 查看栈中的记录
git stash list
```
**重新应用缓存的 stash**

```bash
# 将缓存堆栈中的第一个 stash 删除，并将对应修改应用到当前的工作目录下。
git stash pop

# 同 pop, 但不删除缓存 stash
git stash apply

# 指定使用 stash
git stash list
git stash apply "stash@{0}"
```
**移除 stash**
```bash
# 
git stash drop "stash@{0}"

# 删除所有缓存
git stash clear
```
**查看指定的 stash 的 diff**
```bash
git stash show 

# 查看全部
git stash show -p
```
**从 stash 创建分支**
```bash
git stash branch test

```
## git gc

- 清理不必要的文件并优化本地存储库

## git tag  
**创建一个标签**
```bash
git tag v1.0

# 对历史提交的 commit 创建标签
git tag v0.9 f32c8ff # commit_id 

# 创建带有说明的标签
# -a 指定标签名
# -m 指定说明文字
git tag -a v0.1 -m 'version 0.1 released' 
```
**查看标签**
```bash
git tag

# 查看标签信息
git show <tagname>
```
**删除标签**
```bash
# 创建的标签都只存储在本地，不会自动推送到远程
git tag -d v0.1

# 删除远程标签
git tag -d v0.1 # 删除本地
git push origin :refs/tags/v0.1
```
**推送标签**
```bash
# 推送某个标签到远程
git push origin <tagname>

# 一次性推送全部尚未推送到远程的本地标签
git push origin --tags
```
## 

## 忽略某个文件

```bash
# 暂时忽略你对文件做的修改
git update-index --assume-unchanged
# 恢复跟踪
git update-index --no-assume-unchanged

如果忽略的文件多了，可以使用以下命令查看忽略列表

git ls-files -v | grep '^h\ '
提取文件路径，方法如下

git ls-files -v | grep '^h\ ' | awk '{print $2}'
所有被忽略的文件，取消忽略的方法，如下

git ls-files -v | grep '^h' | awk '{print $2}' |xargs git update-index --no-assume-unchanged  
```
```bash
// config.ts
cp config.ts config.back.ts
git update-index --assume-unchanged config.ts
// 远程仓库更改,本地 pull 时会提示需要手动处理 config 文件
git update-index --no-assume-unchanged config.ts // 取消忽略
git stash
git pull
```
```bash
// config.ts
# 忽略 config.ts
git update-index --assume-unchanged config.ts
# 远程仓库更改,本地 pull 时会提示需要手动处理 config 文件
git update-index --no-assume-unchanged config.ts // 取消忽略
# 将 config.ts 加入暂存区 
git stash
git pull origin develop

# 将缓存堆栈中的第一个 stash 删除，并将对应修改应用到当前的工作目录下。
git stash pop
```
## 使用 git rebase 合并多个 commit

`rebase`的作用简要概括为：可以对某一段线性提交历史进行编辑、删除、复制、粘贴

> 不要通过rebase对任何已经提交到公共仓库中的commit进行修改


```bash
// 把如下分支B、C、D三个提交记录合并为一个完整的提交
A -> B -> C -D

A -> B
```
```bash
git rebase -i  [startpoint]  [endpoint]
```

- `-i`的意思是`--interactive`，即弹出交互式的界面让用户编辑完成合并操作
- `[startpoint] [endpoint]`则指定了一个编辑区间
- `(start-commit, end-commit] `前开后闭区间，默认 `end-commit` 为当前 `HEAD`
```bash
git log --pretty=format:"%h %s"
156d86d D
dfe9661 C
a53648d B
32d789e A
```
```bash
git rebase -i HEAD~3
```
```bash
edit a53648d B
pick dfe9661 C
pick 156d86d D

# Rebase 32d789e..156d86d onto 32d789e (3 commands)
#
# Commands:

# p, pick <commit> = use commit 
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#

# pick：保留该commit（缩写:p）
# reword：保留该commit，但我需要修改该commit的注释（缩写:r）
# edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
# squash：将该commit和前一个commit合并（缩写:s）
# fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
# exec：执行shell命令（缩写:x）
# drop：我要丢弃该commit（缩写:d）
```
```bash
# 把 C 、D提交都合并到 B 
pick a53648d B
s dfe9661 C
s 156d86d D
```
`:wq` 保存后进入 注释修改
```bash
# This is a combination of 3 commits.
# This is the 1st commit message:
B
# This is the commit message #2:
C
# This is the commit message #3:
D
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon May 24 10:09:14 2021 +0800
#
# interactive rebase in progress; onto 32d789e
# Last commands done (3 commands done):
#    squash dfe9661 C
#    squash 156d86d D
# No commands remaining.
# You are currently rebasing branch 'master' on '32d789e'.
#
# Changes to be committed:
#	modified:   test.md
#

```
```bash
# This is a combination of 3 commits.
fix: 合并提交

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon May 24 10:09:14 2021 +0800
#
# interactive rebase in progress; onto 32d789e
# Last commands done (3 commands done):
#    squash dfe9661 C
#    squash 156d86d D
# No commands remaining.
# You are currently rebasing branch 'master' on '32d789e'.
#
# Changes to be committed:
#	modified:   test.md
#

```
`:wq` 后完成保存，查看 log 
```bash
git log --pretty=format:"%h %s"
b7ba9fa fix: 合并提交
32d789e A
```
## Git Submodule
[git submodule](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
> `git submodule` 允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立

> 
> 举例：某个工作中的项目需要包含并使用另一个项目。 也许是第三方库，或者你独立开发的，用于多个父项目的库。你想要把它们当做两个独立的项目，同时又想在一个项目中使用另一个。


1. 首先将一个已存在的 Git 仓库添加为正在工作的仓库的子模块
- `git submodule add [URL]`
```bash
git submodule add https://github.com/xxx/xxx
```
默认情况下，子模块会将子项目放到一个与仓库同名的目录中, 如果你想要放到其他地方，那么可以在命令结尾添加一个不同的路径

- 运行 `git status`
```bash
$ git status

	new file: .gitmodules
  new file: xxx
```
.gitmodules 文件。 该配置文件保存了项目 URL 与已经拉取的本地目录之间的映射, 如果有多个子模块，该文件中就会有多条记录。 要重点注意的是，该文件也像 .gitignore 文件一样受到（通过）版本控制。 它会和该项目的其他部分一同被拉取推送。


克隆含有子模块的项目

 1. 克隆一个含有子模块的项目时，默认会包含该子模块目录，但其中还没有任何文件, 运行

- `git submodule init` 初始化本地配置文件
- `git submodule update` 抓取所有数据并检出父项目中列出的合适的提交
2. `git clone `命令传递 `--recurse-submodules`  会自动初始化并更新仓库中的每一个子模块， 包括可能存在的嵌套子模块。
## 常见问题

### git push 免密码

1. 通常情况

主目录下创建用户名和密码文件
```javascript
vim .git-credentials
https://username:password@github.com
```
添加git config内容

```javascript
git config --global credential.helper store
```
执行此命令后，用户主目录下的.gitconfig文件会多了一项
```javascript
[credential]
        helper = store
```


重新git push就不需要用户名密码了

1. 使用 ssh 协议

首先生成密钥对：
```javascript
ssh-keygen -t rsa -C "youremail"
```

将生成的位于`~/.ssh/`的`id_rsa.pub`的内容复制到你github setting里的ssh key中
复制之后，如果还没有 clone 仓库，直接使用ssh协议用法：
```javascript
git@github.com:yourusername/yourrepositoryname
```

如果已经使用https协议，那么按照如下方法更改协议：
```javascript
git remote set-url origin git@github.com:yourusername/yourrepositoryname.git
```

Done!
### git add 使用tab键自动补全的中文文件名乱码
**解决方法为：**
```javascript
git config --global core.quotepath false
```

### git迁移仓库到另外一个仓库

- 仅保留历史提交信息

```javascript
git clone --bare yourrepository
```

然后在你的其他服务，比如gogs新建一个仓库，然后进入你上步克隆出的仓库中，执行：
```javascript
git push --mirror yourNewRepository
```

然后你就可以删除原来的仓库，然后`git clone`新仓库就行了。


### error: RPC failed; curl 18 transfer closed with outstanding read data remaining

1. 资源文件太大
```git
git clone http://xxx.git --depth 1
git fetch --unshallow
```

2. 缓存区溢出
```git
Git config --global http.postBuffer 524288000
```

3. 网络下载速度缓慢
```git
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

### 修改 commit 的 name 和 email


**修改最近一次**
```
git commit --amend --author="userName <userEmail>"
# 注意不能缺少< >
```


**批量修改**
```bash
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="要更改的用户邮箱"
CORRECT_NAME="正确的用户名"
CORRECT_EMAIL="正确的邮箱地址"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```
如果执行失败的话，执行一下这段命令
```bash
如果执行失败的话，执行一下这段命令

git filter-branch -f --index-filter 'git rm --cached --ignore-unmatch Rakefile' HEAD
```
推送到远程
```bash
git push origin --force --all
```

## 
