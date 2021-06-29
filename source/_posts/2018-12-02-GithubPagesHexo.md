---
title: GithubPages&Hexo&Next部署个人博客
date: 2018-12-02 11:05:16
tags:
- gitHub
- hexo
categories:
- 其他
---

# GitHub Pages

**简介**。一种利用GitHub项目建立的静态主页，一般用来介绍GitHub开发者自己和某个项目，但也可以利用它来建立自己的免费个人博客。

**两种主页**。可以建立两种GitHub Pages，一种是个人或组织主页，一个账号只能有一个；一种是项目主页，每个项目又可以有一个。

**建立主页**。建立主页比较简单：具体可以查阅[GitHub Pages官网介绍](https://pages.github.com/)，更加详细的说明请见[GitHub Pages帮助文档](https://help.github.com/categories/github-pages-basics/)

**博客框架**。建立GitHub Pages后，默认使用[Jekyll](https://help.github.com/articles/using-jekyll-with-pages)框架，目前比较流行的是Hexo。

---

<br>

# Hexo

**简介**。快速、简洁且高效的博客框架。详情请见[官网-Hexo](https://hexo.io/zh-cn/)，

**主题**。Hexo有许多不同的主题，比较流行的有Next。

---

<br>

# NexT

一种Hexo主题，源码和安装请见[GitHub Next Theme](https://github.com/theme-next/hexo-theme-next)，相关配置请见[Hexo主题配置](http://theme-next.iissnan.com/theme-settings.html#categories-page)。

> 主题配置的那个网站是针对低版本Next的，除了安装部分要参考[GitHub Next Theme](https://github.com/theme-next/hexo-theme-next)，其余部分如增加标签和分类功能等，都是有用的。



---

<br>

# 实践-搭建个人博客

## GitHub上创建Github Pages项目

* 用户或组织主页。创建一个名称为`{用户名称}.github.io`的新项目。然后就可以用`https://{用户名称}.github.io`去访问了。
* 项目主页。创建任意名称的项目，在项目`Settings->GitHub Pages`中设置`Source`为`master`分支。然后就可以用`http://{用户名称}.github.io/{项目名称}`去访问了。
* 创建`blog-src`分支。由于有两份代码需要处理，一份是博客源代码，一份是部署后生成的静态网站的代码，所以需要开分支用于分别存储，这里我们用`master`分支存储部署后的代码，用`blog-src`分支存储源代码。



## 本地生成Hexo项目

* 安装Hexo。见[官网-安装Hexo](https://hexo.io/zh-cn/docs/)。
* 生成Hexo项目。进入博客所在的文件目录中，`hexo init {博客名}`，
* 测试。`hexo server`查看是否在本地生成静态站点。

详细命令如下：

```bash
npm install hexo-cli -g
hexo init {博客名}
cd blog
npm install
hexo server
```



## 管理和配置

**关联GitHub Pages和Hexo项目**。初始化Hexo项目为`git`项目，然后添加GitHub Pages项目作为远程链接。

```bash
cd {博客名}
git init
# 在blog-src分支上编辑
git checkout -b blog-src
# 个人或组织主页
git remote add origin git@github.com:{用户名}.git
# 项目主页
git remote add origin git@github.com:{用户名}/{项目名}.git
```

**初始化配置**。在站点`_config.yml`里修改`url`、`root`和`deploy`项。

```properties
# 如果是个人或组织主页
url: https://{用户名称}.github.io
root: /
# 如果项目主页
url: https://{用户名称}.github.io/{项目名称}
root: /{项目名称}/

# 下面的{项目名称}指的是管理GitHub Pages的项目名称，不论是个人主页还是项目主页，都有一个项目与之对应。个人主页的项目名称是：{用户名}.github.io。
deploy:
  type: git
  repo: git@github.com:{用户名称}/{项目名}.git
  branch: gh-pages
```

**部署**。利用`hexo g -d`命令生成并发布静态网页内容到GitHub Pages服务中，然后用相对应的配置的`url`去浏览器查看。



## 生成标签和分类页面

详情请见[主题配置文档](http://theme-next.iissnan.com/theme-settings.html#tags-page)



## 配置主题

默认主题是`landscape`,可以配置`NexT`主题，详细操作请见[主题配置文档](http://theme-next.iissnan.com/getting-started.html)



## 版权声明

请见[Hexo文章末尾添加版权信息](http://stevenshi.me/2017/05/26/hexo-add-copyright/)



---

<br>

# 参考

1. [官网-GitHub Pages](https://pages.github.com/)
2. [官网-Hexo](https://hexo.io/zh-cn/)
3. [NexT](https://github.com/theme-next/hexo-theme-next)
4. [NexT配置](http://theme-next.iissnan.com/theme-settings.html#categories-page)
5. [Hexo文章末尾添加版权信息](http://stevenshi.me/2017/05/26/hexo-add-copyright/)