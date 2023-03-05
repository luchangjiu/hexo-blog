---
title: git 学习笔记
date: 2018-03-02 20:03:02
tags: Git
---

# git 学习笔记

## 下载并安装

### git 下载地址

windows:   https://gitforwindows.org/ 
安装时需要勾选 git base 、git gui
mac or linux:  https://git-scm.com/downloads

查看是否安装成功

```bash
git --version
```

### GUI

熟悉命令操作后可以使用图形化界面来操作，可以在： https://git-scm.com/downloads/guis  下载相关的图形化软件

## 初始配置

配置文件名为： .gitconfig
windows 系统在 c:\\user\\你的计算机名\\.gitconfig  
linux 系统在 ~/.gitconfig  

首先需要配置 你的邮箱和名称

```bash
# 需要注意的是由于是外国的软件所以name 尽量配置英文
git config --global user.email 'xxxxx@xx.com'
git config --global user.name = 'xxxx'
```

## 常用命令

```bash
# 初始化新仓库
git init

# 克隆代码 
git clone https://gitee.com/xxxxx/xxxxx.git

# 克隆指定分支
git clone -b 分支名 git@gitee.com:xxxx/xxxx/xxxx.git

# 查看状态
git status 

# 提交单个文件
git add 文件名.后缀名

# 提交所有文件
git add -A 

# 试用通配符提交
git add *.txt

# 提交到仓库中
git commit -m '提交信息'

# 提交已经跟踪过的文件，不需要执行add 
git commit -a -m '提交信息'

# 删除版本库和项目目录中的文件 
git rm a.txt

# 只删除版本库中的文件
git rm --cached a.txt

# 修改最后一次提交
git commit --amend

```

## 初始流程

```bash
# 1. 克隆项目
git clone https://gitee.com/xxx/xxx.git

# 2. 开始开发添加a.txt或修改文件，这时新的文件并没有被版本管理，可以通过以下命令查看没有被管理的文件
git clean -n

# 3. 将所有操作提交到暂存区
git add .

# 这时再通过clean 命令查看会发现结果为空，即文件已经被版本库管理了
git clean -n

# 4. 不小心将文件a.txt删除了，现在可以将暂存区中的a.txt 恢复回来
git checkout a.txt

# 5. 完成工作后创建一个新的提交，并使用 -m 选项说明完成的工作
git commit -m '说明完成的事情'

# 6. 将代码提交到远程服务器，与同事或其他开发者分享代码
git push 

```

## 工作区

git clean 命令用来从工作目录中删除所有没有跟踪（tracked）过的文件
1. ` git clean -n ` 是一次clean的演习，告诉你那些文件会被删除
2. ` git clean -f ` 删除当前目录下没有tracked 过的文件，不会删除.gitignore 指定的文件
3. ` git clean -df ` 删除当前目录下没有被tracked 过的文件和文件夹
4. ` git checkout . ` 将没有放到暂存区的所有文件恢复
5. ` git checkout a.txt ` 恢复没有放到暂存区的指定文件
6. ` git checkout -- a.txt ` 将文件从暂存区恢复（如果没有提交到暂存区，将恢复到最近版本）

## 暂存区

1. ` git add  . ` 提交所有修改和新增的文件
2. ` git add -u ` 只提交修改文件不提交新增文件
3. ` git ls-files -s ` 查看暂存区文件列表
4. ` git cat-file -p xxxxx ` 查看暂存区文件内容
5. ` git reset ` 撤销上次提交到暂存区动作

## 日志查看

1. ` git log ` 查看日志
2. ` git log -p 2 ` 查看最近2 次提交日志显示文件差异
3. ` git log --name-only ` 显示已修改的文件清单
4. ` git log --name-status ` 显示新增、修改、删除的文件清单 
5. ` git log --oneline ` 一行显示并只显示SHA-1 的前几个字符
6. 下面是自定义的精简日志信息(可以设置在别名上方便使用)

```bash

git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

```

## 分支管理

分支用于为项目增加新功能或修复bug时使用

### 分支流程

一般情况下主分支 用来存放稳定版本，在开发功能时应该由拉去新分支去完成，然后再主分支合并。
所以一把修复bug 和开发新功能都应创建分支

以一个后台管理项目为例

1. 新建一个登录模块的分支

```bash
git branch login
```

2. 切换到登录模块的分支

```bash
git checkout login 
```

3. 开发并完成提交

```bash
# 模拟创建登录模块文件
touch login.html 

git add .

git commit -m '提交登录模块代码'
```

4. 合并分支到master

```bash
# 切换到主分支master
git checkout master
# 合并login 模块
git merge login
```

5. 提交代码到master 远程分支

```bash

git push

```

### 分支常用命令

1. ` git branch dev ` 创建分支dev
2. ` git branch ` 查看分支
3. ` git checkout dev ` 切换到dev 分支
4. ` git checkout -b dev2 ` 创建并切换分支 dev2
5. ` git checkout branch -m dev dev3 ` 将分支dev 更新为dev3 名称
6. ` git merge dev `  合并分支 dev 分支到当前分支
7. ` git branch -d dev ` 删除dev 分支
8. ` git branch -D dev ` 删除没有合并的分支(相当于放弃该分支的所有操作,谨慎操作)
9. ` git push origin :dev ` 删除远程分支
10. ` git branch --no-merged ` 查看未合并的分支（需要切换到创建分支的分支比如master）
11. ` git branch --merged ` 查看已经合并的分支（需切换到创建分支的分支比如master)

### 分支优化技巧



### 历史版本

下面是使用历史版本创建分支

` git log ` 查看历史版本日志
` git checkout 'commit-id' ` 切换到提交的commit-id （SHA-1 签名6 个字符）历史版本
` git checkout 'commit-id' -b 新分支名称 ` 以历史版本创建分支

## reset

使用reset恢复到历史提交点，重置暂存区与工作目录的内容。

### reset 可选参数

reset 有个三个选项可以使用

1. --hard 重置位置的同时，直接将working tree 工作目录、index 暂存区及repository 都重置成目标reset 节点的内容
2. --soft 重置位置的同时，保留working tree 工作目录和index暂存区的内容，只让repository 中的内容和reset 目标节点保持一致
3. --mixed(默认) 重置位置的同时，只保留working tree 工作目录的内容，但是会将index 暂存区和repository 中的内容更改和reset目标节点一致

### 使用示例

1. ` git reset ` 将add 到暂存区的内容回退
2. ` git reset --hard '版本id' ` 恢复到指定提交版本（先通过` git log  ` 查看版本号),重置stage 区和工作目录里的内容
3. ` git reset --hard HEAD^^^ ` 恢复前3 个版本
4. ` git reset --soft ` 保留工作区内容，只回退commit的动作。保留working tree 工作目录的内容，index 暂存区与working tree 工作目录内容一致，只是仓库repository中的内容的改变。
5. ` git reset HEAD -- . ` 撤销暂存区的文件
6. ` git reset --hard ` 清空工作区和暂存区的改动
7. ` git reset HEAD xx.txt ` 放弃已经add 暂存区的文件xx.txt

## 其他知识

通过创建命令别名可以减少命令输入量，有几种方式进行设置

### 配置文件定义

修改配置文件 ~/.gitconfig 并添加以下命令别名 配置段

```bash

[alias]
    a = add .
    c = commit 
    s = status 
    l = log 
    b = branch
```

现在可以使用 ` git a ` 实现 git add . 一样的效果了

### 系统配置定义

window 用户可以使用 ~/.bashrc 或 ~/.bash_profile 文件.
mac/linux 修改~/.zshrc 文件中定义常用的别名指令，需要首先安装zsh命令行进行扩展

```bash
alias gs="git status"
alias gc="git commit -m "
alias gl="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
alias gb="git branch"
alias ga="git add -A"
alias go="git checkout"
alias gp="git push;git push github"

```

命令行直接使用 ` gs ` 就可以实现 ` git status ` 一样的效果了。

**window 系统需要使用git for window 中的git bash 软件才能达到效果**

### .gitignore 

.gitignore 用于定义忽略提交的文件

- 所有空行或者以注释符 ` # ` 开头的行都会被git忽略
- 匹配模式最后跟反斜杠（` / `）说明要忽略的是目录。
- 可以使用标准的glob模型匹配

```git
.idea
/vendor
.env
/node_modules
/public/storage
*.txt
```

### 冲突解决

不同分支修改同一个文件或不同开发者修改同一个分支文件都可能造成冲突，造成无法提交或合并代码。

1. 使用编辑器修改冲突文件
2. 添加暂存` git add . ` 表示已经解决冲突
3. ` git commit -m 'xxxx' `提交完成

### Stashing

当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点

"暂存" 可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

1. 储藏工作 ` git stash `
2. 查看储藏列表 ` git stash list `
3. 应用最近的储藏 ` git stash apply `
4. 应用更早的储藏 ` git stash apply stash@{2} `
5. 删除储藏` git stash drop stash@{0} `
6. 应用并删除储藏 ` git stash pop `

### Tag
Git 也可以对某一时间点上的版本打上标签 ，用于发布软件版本如 v1.0

1. 添加标签 ` git tag v1.0 `
2. 列出标签 ` git tag `
3. 推送标签 ` git push --tags `
4. 删除标签 ` git tag -d v1.0.1 `
5. 删除远程标签 ` git push origin :v1.0.1 `

## 打包发布

```bash
git archive master --prefix='xxx/' --format=zip > xxx.zip
```

## 远程仓库

### 创建仓库

可以使用 github 或者 自己搭建的gitlib（linux） 和 Bonobo Git Server （windows） 作为远程仓库

#### github 步骤

1. 在远程仓库上创建仓库
2. 为方便不需要每次都输入密码github 可以使用本机ssh 生成密钥

3. ssh 生成密钥的步骤

3.1. 运行命令

```bash
ssh-keygen -t rsa
```

3.2. 如果不需要特别填写信息可以一路回车知道结束，此时在系统的`~/.ssh` 目录中会有`id_rsa`密钥和`id_rsa.pub`公钥.

4. 用记事本或其他编辑器打开`id_rsa.pub` 复制上面的公钥

5. 打开github 上的 Settings 下的 SSH and GPG keys 将复制到的公钥信息粘贴到上面即可

#### Boonobo Git Server

1. 创建仓库
2. 通过 `git clone https://xxxxx.com/xxxx/xxx.git` 拉去代码
3. `git push`时会有录入密码的提示
4. 如果不想重复录入密码，可以执行`git config credential.helper store` 命令，在本仓库中存储密码，如果需要全局配置 则使用`git config --global credential.helper store`

### 关联远程

1. 创建本地库并完成初始提交

```bash
git init
touch xxxx.txt
echo 'gaga' > xxxx.txt
git add -A
git commit -m 'initial'
```

2. 添加远程仓库

```bash
git remote add origin git@github.com:xxxxx/xxx.git
```

3. 查看远程库

```bash
git remote -v
```

4. 推送数据到远程仓库

```bash
git push -u origin master
```

5. 删除远程仓库关联

```bash
git remote rm origin
```

### pull

拉取远程主机某个分支更新，在与本地指定分支合并

1. 拉取origin主机的 dev分支与本地的master分支合并

```bash
git pull origin dev:master
```

2. 拉取origin主机的dev分支与当前分支合并

```bash
git pull origin dev
```

3. 如果远程分支与当前分支同名直接执行

```bash
git pull
```

### push

`git push` 命令用于将本地分支的更新，推送到远程主机。

1. 将当前分支推送到 origin主机的对应分支（如果当前分支只有一个追踪分支，可省略主机名origin）

```bash
git push [origin]
```

2. 使用`-u` 选项指定一个默认主机，这样以后久可以不加任何参数直接使用 `git push`

```bash
git push -u origin master
```
3. 删除远程 dev 分支

```bash
git push origni --delete dev
```

4. 本地dev 分支关联远程分支并推送

```bash
git push --set-upstream origin dev
```

### 多库提交

在push 时可以将代码提交到多个远程仓库上。

```bash
# 增加一个远程库
git remote add github git@github.com:xxxx/xxxx.git
# git remote add custom_repo http://xxxx.com:80/xxxx/xxx.git

# 提交到远程库
git push github & git push custom_repo
```

可以在~/.bashrc 中定义别名方便同时push 到多个远程库

```bash
alias gp='git push & git push github'
```
