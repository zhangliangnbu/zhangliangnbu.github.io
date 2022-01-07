---
title: 版本管理之Git分支合并方式和策略
date: 2019-03-27 09:47:58
tags:
  - git
  - merge
  - rebase
categories:
  - 其他
---

# Merge和Rebase

区别。参考[闲谈 git merge 与 git rebase 的区别](https://www.jianshu.com/p/c17472d704a0)

- merge 合并两个分支的Head；提交历史忠实地记录了实际发生过什么，关注点在真实的提交历史上面。
- rebase 提取当前分支的修改，将其复制在了目标分支的最新提交后面；反映了项目过程中发生了什么，关注点在开发过程上面
- 区别。`merge`会保留当前分支的历史记录，而`rebase`会改动当前分支的历史记录。

<!-- more -->

# Merge合并方式

FAST-FORWARD 和非 FAST-FORWARD

- FAST-FORWARD：当前分支HEAD是欲合并入分支的一个祖先节点，无需创建额外的合并提交(create a merge commit)节点，直接更新HEAD指针。
- 非FAST-FORWARD：合并后会创建额外的合并提交节点，这是TRUE MERGE，有几种合并策略。

使用

- 执行`git merge others`。如果系统认为当前分支HEAD是`others`分支的一个祖先节点，会使用`FAST-FORWARD`方式，否则使用`非FAST-FORWARD`方式。
- 执行`git merge others --ff` 。同执行`git merge others`。
- 执行`git merge others --no-ff`。都会创建一个额外合并提交节点。
- 执行`git merge others --ff-only`。如果系统认为当前分支HEAD是`others`分支的一个祖先节点，会使用`FAST-FORWARD`方式，否则不执行合并。

---

# Merge合并策略

非FAST-FORWARD的的几种合并策略：

- resolve。采用三路合并，当有多个共同祖先时，系统采用自认为合适的祖先。
- recursive。采用递归三路合并。
- octopus。合并超过两个分支时。

>**三路合并（3-way merge）算法** 参考[维基百科-合并 (版本控制)](https://zh.wikipedia.org/zh-hans/%E5%90%88%E5%B9%B6_(%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6))
>
>- 简单三路合并。两个分支的文件和共同祖先文件做差异分析。
>- 递归三路合并。有多个共同祖先，先创建虚拟共同祖先。



---

# 参考

1. [闲谈 git merge 与 git rebase 的区别](https://www.jianshu.com/p/c17472d704a0)
2. [A Formal Investigation of Diff3](http://www.cis.upenn.edu/~bcpierce/papers/diff3-short.pdf)
3. [维基百科-合并 (版本控制)](https://zh.wikipedia.org/zh-hans/%E5%90%88%E5%B9%B6_(%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6))
4. [git 合并策略 之 recursive](http://blog.fedeoo.cn/2017/02/22/git-%E5%90%88%E5%B9%B6%E7%AD%96%E7%95%A5-%E4%B9%8B-recursive/)