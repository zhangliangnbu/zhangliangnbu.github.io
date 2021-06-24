---
title: Git分支管理策略的选择
date: 2018-12-08 20:46:51
tags:
- Git
- 分支管理策略
categories:
- 版本管理
---

# 前言

目前流行的Git项目工作流程有[Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)、[GitHub Flow](https://guides.github.com/introduction/flow/index.html)和[GitLab Flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html)，对于不同的项目应该选择不同的工作流程以提交开发效率。

----

<br>

# 选择

1. 持续构建的项目，优先选择GitHub Flow。例如Web项目，每次更新后即可随时发布。
2. 有明显的版本迭代或有发布窗口期的项目，优先选择GitLab Flow，如iOS项目。
3. 任何项目，如果不想选择以上二者，都可以选择Git Flow。

-----

<br>

# 待续

不够详细，待续

----

<br>

# 参考

1. [阮一峰-Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)
2. [阮一峰-Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
3. [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
4. [GitHub Flow](https://guides.github.com/introduction/flow/index.html)
5. [GitLab Flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html)
6. [Four git workflow](https://medium.com/@patrickporto/4-branching-workflows-for-git-30d0aaee7bf)