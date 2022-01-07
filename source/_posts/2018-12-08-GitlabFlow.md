---
title: 版本管理之GitlabFlow简介
date: 2018-12-08 20:36:59
tags:
- git
categories:
- 其他
---

> **原文链接** [Introduction to GitLab Flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html)

<!-- more -->



![](https://upload-images.jianshu.io/upload_images/2658578-4fbe838c70462bed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在分支管理和合并方面，用Git做版本管理比用以前的版本管理系统如SVN更容易。Git版本管理允许各种各样的分支策略和工作流程。这些策略和流程是对以前的版本管理方法上的一种改进和提升。但是许多企业的工作流程或者不清晰，或者过于复杂，或者未集成问题跟踪系统（issue tracking systems）。鉴于此，我们提出Gitlab工作流(GitLab flow)，作为定义清晰的最佳实践模式。它结合了[功能驱动开发模式](https://en.wikipedia.org/wiki/Feature-driven_development)和[功能分支模式](https://martinfowler.com/bliki/FeatureBranch.html)，并集成了问题跟踪功能。

从其他版本管理系统迁移到Git的团队一般很难用高效的流程进行开发。本文介绍Gitlab工作流，它结合了Git工作流和问题追踪系统，是一种简单、透明和有效的使用Git进行版本管理的工作方式。

![](https://upload-images.jianshu.io/upload_images/2658578-5050f3e3155da858.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将工作副本提交到远程服务，大多数版本管理系统只需要一步，而Git需要三步：从工作区添加文件到暂存区（staging area）、提交暂存区内容到本地库、推动本地库信息到远程库。熟悉这三步后，开发人员又面临另一个挑战——Git的分支模型。

![](https://upload-images.jianshu.io/upload_images/2658578-e88141fe2d2ee890.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于许多刚接触Git的团队并没有约定如何使用它，就会导致之后的开发变得一团糟。 他们遇到的最大问题是许多长期运行的分支都包含部分变更，这就很难确定应该在哪个分支上开发，在哪个分支上部署生产版本。 对此问题的解决方法通常是采用标准化模式，例如Git flow和GitHub流。 我们认为仍有改进的余地，并将详细介绍我们称之为最佳实践的GitLab流。

# Git Flow和它的问题

![Git Flow](https://upload-images.jianshu.io/upload_images/2658578-215fdb1fb9e423c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Git Flow是最早提出的分支工作模型之一，引起了广泛的关注。它包含两个长期分支，master主分支和开发分支，以及其他的暂时分支，功能分支、发布分支以及程序修复分支。工作流程是，在开发分支上开发，之后转移动发布分支，最后合并到master分支。Git Flow的分支模型定义清晰、明确，但过于复杂，存在两个问题。
1. 开发使用开发分支而不是master分支。master分支的作用仅仅是存储生产版本的代码，没有发挥更大的作用。一般的开发惯例是在默认分支上开发，在此分支上创建分支以及合并。由于大多数工具会自动将主分支设置为默认分支，并且默认显示该分支，因此每次必须切换到另一个分支上开发是很烦人的事情。
2. 过于复杂。引入了程序修复分支和发布分支，导致Git Flow过于复杂。这些分支对某些项目来说可能是必要的，但对绝大多数项目来说则是没有必要的。如今大多数项目都是持续发布的，这意味着你的默认分支会被发布。这样就没有必要引入修复分支和发布分支，以及相关的复杂操作。这些操作的一个例子是发布分支的合并，通常开发人员会犯错误，例如，更改只合并到master中，而忘记合并到develop分支中，但执行发布并不意味着也会执行修补程序。这些错误的根本原因是Git Flow的使用相对较复杂。

<br>
# GitHub Flow，一个简单的替代者

![image.png](https://upload-images.jianshu.io/upload_images/2658578-341bb1faacc1e34f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

针对Git Flow的简单替代方案是[GitHub Flow](https://guides.github.com/introduction/flow/index.html)。这个流程只包含一个主分支和一些功能分支，简单、干净。许多企业已经都在采用这个工作流，并取得了巨大的成功。Atlassian推荐了一个类似的流程，只是修改了功能分支( rebase feature branches)。 将所有内容合并到主分支并部署，这会让你的代码库更简洁，符合持续交付的最佳实践。 但是这个流程仍然存在很多关于部署、环境、发布和议题集成(integrations with issues)的问题。 而GitLab Flow会解决这些问题。
<br>

# GitLab Flow的Production分支策略

![GitLab Flow的Production分支](https://upload-images.jianshu.io/upload_images/2658578-1df9470b1c257e6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

GitHub Flow是假设你每次合并功能分支后都能部署到生产环境。这可以用于像SaaS之类的应用程序，但在许多情况下并不能这样做。一种情况是你自己无法控制确切的发布时间，例如IOS应用程序，需要通过App Store审核。另一种情况是你有部署的窗口期（如工作日的上午10点到下午4点，运营团队才帮助你部署），但是你也需要在其他时间合并其他的代码（不能傻等）。在这些情况下，您可以创建一个反映需要部署的代码的生产分支。您可以通过将master分支合并到生产分支来部署新版本。如果你需要知道生产中的代码，您可以查看生产分支。在版本控制系统中，合并提交都有记录，很容易看到部署的大致时间。如果你自动部署生产分支，则这个时间会非常准确。如果你需要更加准确的时间，则可以利用脚本文件在每个部署版本上创建标记。此流程无需Git Flow中常见的发布、标记和合并操作。

>这句话怎么理解？原文:This flow prevents the overhead of releasing, tagging and merging that is common to git flow

<br>

# GitLab Flow的Environment分支策略

![GitLab Flow的Environment分支](https://upload-images.jianshu.io/upload_images/2658578-69c8a8b6177f39ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以在master分支上自动更新环境(开发、测试、生产等)，在这种情况下环境名和分支名可能不一致。假设你有一个开发环境(staging environment)，一个预生产环境(pre-production environment)和一个生产环境(production environment)。在这种情况下，master分支部署在开发环境中，当要部署到预生产时，创建从master分支到预生产分支的合并请求，之后再将预生产分支合并到生产分支中用于发布生产版本。这种只向下游做提交的工作流程(master->pre-production->production)可确保代码在所有环境中都经过测试。如果您需`cherry-pick`一个热修复提交，通常是在功能分支上开发它，之后将其合并到master分支中，但不要删除该功能分支。如果master分支运行良好（假设你正在开发持续部署类项目），然后你需要将修复补丁分支合并到其他分支中。

> 疑问：这和Git Flow中打紧急修复补丁的做法有什么区别吗？都是需要在开发分支和生产分支上合并修复的代码啊？
> 这段感觉他说的不是很清楚，或者我翻译的太糟。我看阮一峰的[Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)，也有些迷惑，如下:
>
> >开发分支是预发分支的"上游"，预发分支又是生产分支的"上游"。代码的变化，必须由"上游"向"下游"发展。比如，生产环境出现了bug，这时就要新建一个功能分支，先把它合并到`master`，确认没有问题，再`cherry-pick`到`pre-production`，这一步也没有问题，才进入`production`。

>这里“新建一个功能分支”是在哪里创建？master的最新提交还是对应的创建生产分支的那个提交？。`herry-pick`哪个分支上的提交？功能分支还是master分支？
>正文中说“不要删除该功能分支”，加上实践，只能是在master对应的创建生产分支的那个提交上新建分功能分支，然后`cherry-pick`功能分支上提交到预生产和生产分支上。

如果由于需要许多手动测试而无法实现，则可以发送合并请求，将功能分支合并到下游分支。
>这句话不甚理解。原文: If this is not possible because more manual testing is required you can send merge requests from the feature branch to the downstream branches

<br>

# GitLab Flow的发布分支策略

![GitLab Flow的Release分支](https://upload-images.jianshu.io/upload_images/2658578-7d01159cf93fa25a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只有在你向外界发布软件时，才需要使用发布分支。在这种情况下，每个分支包含较小的版本（2-3-stable, 2-4-stable等）。stable分支从master分支检出，并尽可能晚地创建。越晚创建stable分支，错误越少，最大限度地避免将错误修复补丁应用于多个分支。在创建发布分支后，发布分支中仅允许修复严重的错误。如果需要修复，那么这些错误修复要首先合并到master分支中，然后`cherry-pick`到发布分支中，这样能避免在后续版本中遇到相同的bug。这就是所谓的“上游优先”策略，也是Google和Red Hat实施的策略。每次在发布分支中增加错误修复补丁时，都会通过设置新标记来发布补丁版本（以符合语义版本控制）。有些项目还有一个稳定的分支，指向与最新发布的分支相同的提交。在此流程中，拥有生产分支（或git flow master分支）并不常见。

>这是向外发布软件的工作流，需要维护历史版本的时候，可以利用这个。软件的每个版本都有一个对应的分支来维护。

>修复历史版本的Bug的工作流程是怎样的？我的理解如下：当发现历史版本如v1.3中有错误需要修复的时候，首先在master分支中对应v1.3的提交上创建hotfix分支，测试无误后提交，首先合并到master分支（以后在maser分支上创建的发布分支就不会出现这个Bug），然后`cherry-pick`hotfix分支上的提交到v1.3及以后的所有的发布分支上。
>是不是如此呢？我想不出其他的。


>上游优先策略。master是所有发布分支的上游，要改动发布分支上的代码，必须先改动`master`，然后`cherry-pick`到发布分支上。
>问题：`cherry-pick`的提交一定是在`master`分支上的吗？不是的，只是说先改动`master`，并不是说一定`cherry-pick` `master`分支上的提交。

<br>

# 待续

翻译得心累(⊙o⊙)，待以后完全理解了GitlabFlow再把接下来的几节翻译完。

* GitLab Flow的Merge/pull requests
* GitLab Flow的议题追踪
* 在merge requests中关联或关闭议题
* 使用```rebase```合并```commits```