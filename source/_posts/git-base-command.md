---
title: 版本管理之Git基础与命令
date: 2018-10-23 10:18:06
update: 2018-10-23 10:18:06
tags:
- git
- vcs
categories:
- 其他
---

本文主要介绍Git的机制及使用、常用命令等。

<!-- more -->

# 前言

使用Git也有几年了，涉及场景较多的是：

**学习开源库**。从GitHub上克隆代码

```
$ git clone ...
```
**公司开发**

```
// 分支相关
$ git branch
$ git checkout -b dev
$ git checkout dev

// 提交代码
$ git status
$ git add .
$ git commit -m "..."
$ git pull
$ git push origin dev
```
其他情况使用命令行如`git merge`等较少，多用IDE（如 Android Studio）自带的版本控制GUI工具。一般情况下，记得上面几个命令就足以应付日常工作了，如果需要使用其他的命令，网上搜一下即可。

最近开始管理小团队，需要管理分支的创建、代码审核、分支合并、删除、冲突解决等等，开发的人数开始增多，以前不常用的命令，现在也要常用了，因此有必要全面的总结下Git的使用。

可能有人会问，为什么要学习命令行，熟悉GUI不就可以了吗？答案是不可以的。
> 只有在命令行模式下你才能执行 Git 的 所有 命令，而大多数的 GUI 软件只实现了 Git 所有功能的一个子集以降低操作难度。 如果你学会了在命令行下如何操作，那么你在操作 GUI 软件时应该也不会遇到什么困难，但是，反之则不成立。 此外，由于每个人的想法与侧重点不同，不同的人常常会安装不同的GUI 软件，但所有人一定会有命令行工具。

本文主要资料参考[Git官网-Pro Git教程](https://git-scm.com/book/zh/v2)和[廖雪峰的Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)。廖写的东西通俗易懂，非常值得一读。我的总结尽可能精简，初学者可以看廖的教程，了解更详细的可以看[Git官网-Pro Git教程](https://git-scm.com/book/zh/v2)，说的非常详细，里面有中文文档。

-- -- --

<br>



# Git基础
**三个工作区域**
一个Git项目包括：工作目录、暂存区（stage或index）和仓库目录。基本的 Git 工作流程如下：

1. 在工作目录中修改文件。如通过`$ git clone`命令将项目克隆到本地，然后做一些修改。
2. 暂存文件，将文件的快照放入暂存区域。如通过`$ git add .`命令进行操作。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。如通过`$ git commit`命令进行操作。

![来源Git官网](http://upload-images.jianshu.io/upload_images/2658578-0a3c6479055e37b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**三种状态**
工作目录下的文件分为两种状态：已跟踪或未跟踪。
1. 未跟踪。没有纳入版本控制的文件。
2. 已跟踪。已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。

已跟踪的文件分为三种：
1. 已提交（committed）。表示数据已经安全的保存在本地仓库目录中。 
2. 已修改（modified）。表示修改了文件，但还没保存到暂存区中。 
3. 已暂存（staged）。表示对一个已修改文件的当前版本做了标记，即已经放到了暂存区，使之包含在下次提交的快照中。

![Git项目文件生命周期](https://upload-images.jianshu.io/upload_images/2658578-dd8bea81c4b8c82f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**分支结构变更示意**
`master`指向自己分支的最新提交，`HEAD`指向当前分支

1. 只有一个主分支时
    ![只有一个主分支master](https://upload-images.jianshu.io/upload_images/2658578-3d9b499d978d52fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 创建并切换到分支dev。即创建一个dev指针指向当前提交，同时HEAD也指向dev分支
    ![创建并切换分支dev](https://upload-images.jianshu.io/upload_images/2658578-0e4b492bab51186d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 修改并提交分支dev
    ![修改并提交分支dev](https://upload-images.jianshu.io/upload_images/2658578-7ddb24059a53d93e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 合并分支dev到master(默认的快速模式)
    ![合并分支-ff](https://upload-images.jianshu.io/upload_images/2658578-6028d4eab1c70d1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    ![合并分支-no-ff](https://upload-images.jianshu.io/upload_images/2658578-12b3c045442d59b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. 删除分支dev
    ![删除分支dev](https://upload-images.jianshu.io/upload_images/2658578-c9396e94563a2cfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-- -- --

<br>



# 常用命令
## 使用较多的6个命令

![日常使用的命令](https://upload-images.jianshu.io/upload_images/2658578-5bf3855193087be8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 创建

```bash
# 创建版本库。在当前文件件创建版本库。
$ git init
# 添加文件。修改文件后，需要把修改后内容添加到暂存区以备提交。“.” 代表所有文件，也可以添加指定的文件。
$ git add .
# 提交到本地版本库。
$ git commit -m "本次提交的信息"

# 关联远程服务器
$ git remote add origin "远程仓库链接"
# 推送当前分支信息到远程服务器。“origin”代表默认的远程服务，“master”代表分支。
$ git push -u origin master
```

## 查看
```bash
# 查看状态。当前版本目录的状态
$ git status

# 查看提交历史
$ git log
$ git log --pretty=oneline
$ git log --graph

# 查看命令历史：checkout、commit、reset等
$ git reflog

```
## 回退、撤销与删除
```bash
# 版本回退。回退到历史版本
# 其中“HEAD^”表示上一个版本，“HEAD^^”表示上上个版本
# 回退到当前版本，并清除所有修改
$ git reset --hard HEAD
# 回退到上个版本，并清除所有修改
$ git reset --hard HEAD^
# 回退到上个版本，本地保留修改
$ git reset --soft HEAD^
# 版本切换。回退到历史版本或跳到最新版本。
# “1094a”表示指定commit id的版本
# 可以通过“git log”命令查看历史版本commit id
# 或通过通过“git reflog”查看提交记录
$ git reset --hard 1094a


# 撤销已修改的文件。恢复到最近一次git commit或git add时的状态
$ git checkout -- <file>

# 撤销已暂存的文件。恢复到已修改状态。
# 然后通过“git checkout -- <file>”，恢复到最近一次git commit或git add时的状态
$ git reset HEAD <file>

```
## 分支管理

**命令-创建和切换**

```bash
# 创建并切换分支
$ git checkout -b dev
# 查看本地分支
$ git branch
# 查看远程分支
$ git branch -a
# 切换分支
$ git checkout dev
$ git checkout tag-v1.0 -b hotfix-v10-xxx
```

**命令-合并**
```bash
# 在其他分支上合并dev分支
# 默认是Fast-forward模式，直接把master指向dev的当前提交，这种模式下，删除分支后，会丢掉分支信息
$ git merge dev
# no-ff模式，合并后会保存分支信息
$ git merge dev --no-ff
```

**命令-保存和恢复工作现场**

```bash
# 保存当前工作现场。不提交情况下，可保存当前工作现场，再并切换到其他分支。
$ git stash

# 查看保存的当前工作现场
$ git stash list

# 恢复并删除当前工作现场。如果stash有很多，则默认操作最新的一个。
$ git stash pop
# 恢复当前工作现场。仅恢复，不删除
$ git stash apply
# 删除工作现场
$ git stash drop

```

**命令-删除分支**

```bash
# 删除本地分支`dev`。
$ git branch -d dev
# 删除本地分支。强行删除没有合并的分支
$ git branch -D dev
# 删除远程分支`dev`。
$ git push origin --delete dev
```

## 标签管理
```bash
# 增加标签。默认为HEAD
$ git tag v1.0
# 增加标签。针对历史版本
$ git tag v1.0 <commit id>
# 增加标签。附带说明
$ git tag -a v1.0 -m "version 1.0 release" <commit id>

# 查看所有标签
$ git tag
# 查看某个标签
$ git show v1.0

# 推送标签。增加标签存储在本地，如有必要可以推送到远程。
$ git push origin v1.0
# 推送所有标签。
$ git push origin --tags


# 删除本地标签
$ git tag -d v1.0
# 删除远程标签。先从本地删除，再推送到远程。
$ git push origin :refs/tags/v1.0
```
## 远程仓库
```bash
# 克隆远程库。本地没有建立项目时，使用这个
$ git clone <远程仓库地址>

# 关联本地项目到远程服务。本地有Git项目时，使用这个。
$ git remote add origin <远程仓库地址>

# 从远程库更新本地库。多人协作，远程有更新时
# 会自动合并
$ git pull
# 不会自动合并
$ git fetch

# 推动本地内容到远程
$ git push -u origin master

```


## 配置信息

```bash
# 配置别名
$ git config --global alias.st status
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# 配置颜色
$ git config --global color.ui true

# 配置用户名和邮件
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"

# 查看配置
$ git config --list
```

Git配置信息存储在配置文件中，可以在配置文件中直接修改或删除配置信息。
1. 当期仓库的配置信息在`.git/config`中
2. `--global`的配置信息在用户主目录下的`.gitconfig`中
3. `--system`的配置信息在`/etc/gitconfig`中

其中，每一个级别覆盖上一级别的配置

## 其他实用命令
```bash
# 查看使用手册。<verb>表示要查看的命令名
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```
-- -- --

<br>



# 常见场景
**创建、提交、推送**

```bash
# 本地没有项目，远程有Git项目
$ git clone <远程仓库地址>
$ git add .
$ git commit -m "初始化"
$ git push origin master

# 本地有项目，远程新创建空Git项目
$ git init
# 修改.gitignore文件
$ git add .
$ git commit -m ""
$ git remote add origin <远程仓库地址>
$ git push -u origin master
```

**多人协作**

```bash
# 合并他人分支、解决冲突
# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track dev origin/dev
$ git checkout dev
$ git pull
$ git checkout master
$ git merge dev --no-ff

# 如果没有冲突，直接就合并好了，如果有冲突，先解决冲突，然后提交
# 如果冲突没有解决好，希望撤销这次合并
$ git merge --abort

# 其他待续
```
-- -- --

<br>



# 参考

1. 说明。图中图片皆来自参考链接
2. [廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
3. [Git官网-Pro Git教程](https://git-scm.com/book/zh/v2)
4. [阮一峰-常用Git命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)