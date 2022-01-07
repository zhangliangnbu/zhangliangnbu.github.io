---
title: Android构建01-前言
date: 2018-08-27 14:52:48
updated: 2018-10-17 14:52:48
tags:
- android
- build
- gradle
- groovy
categories:
- Android开发
---

# 说明
开发Android应用，就离不开基于Gradle的Android构建系统。刚开始做Android开发的时候，遇到编译问题，一般上网搜索解决之。但一般很难知道问题产生的深层次原因，也不知道以后如何避免，更不知道如何快速解决一般性的编译异常问题。想去学习，但一般的博客或书籍的内容与Android开发者官网介绍的内容相似，没有关于Groovy和Gradle的内容，让人很难理解，只知道是这样，而不知道为什么是这样。于是，一直想对Android构建系统做一个系统性的学习和总结，最近终于能够抽出一些空来做这件事。

本来只想写一篇文章，但写到Gradle之后，发现篇幅太长，故分开写，当做一个系列。

<!-- more -->

**核心问题**

当我执行`./gradlew assemble`命令时，我需要知道Gradle做了哪些事情？它是如何做的（即源码是如何写的）？

**学习目标**
对于Android Build System，我们应该掌握哪些知识或技能点呢？

1. Groovy。语言基础，尤其是闭包概念，API快速查阅等。
2. Gradle。项目构建流程、基础概念、基本命令行指令、插件、API和Reference的查阅等。
3. Android Plugin。基础概念、Reference的查阅等。
4. Android应用构建（编译和打包）流程。

文章也将主要按照这个学习流程来写。

**参考资料**
主要参考官方文档
1. [Groovy 官网](http://www.groovy-lang.org/learn.html)
2. [Gradle官网](https://gradle.org/)
3. [Android开发者官网-Configure your build](https://developer.android.com/studio/build/)

<br>

# Android构建系统与三大知识块
**Android构建系统**
Android构建系统，即Android Build System，它的作用是编译app资源和源代码，并将它们打包成APK。之后我们可以对APK进行测试、部署、签名和发布。

>The Android build system compiles app resources and source code, and packages them into APKs that you can test, deploy, sign, and distribute

<br>**三大知识块**
整个Android Build System基于三大知识块：

* [Apache Groovy](http://groovy-lang.org/)
一种功能强大的JVM语言，包含脚本编写、DSL（特定领域语言）创建、运行时和编译时元编程以及函数编程等功能。
* [Gradle](https://gradle.org/)
一种项目自动化构建工具。综合了Ant和Maven的一些特点，并使用Groovy的DSL来声明项目设置，而不是传统的XML。包含插件（Plugin）等功能。
>提示：目前Gradle可以使用Kotlin语言来编写脚本，但建议初学者以Groovy为主，毕竟遗留项目以及当前大多数项目的Gradle脚本都是使用Groovy来编写的。

* [Android Plugin](http://google.github.io/android-gradle-dsl/current/)
Gradle的插件，用于增加一些特定于构建Android应用的功能。

<br>

# 开发环境
1. 主流操作系统。Gradle能够运行在所有的主流操作系统上，我的系统是`masOS High Sierra Version 10.13.6` 。

2. [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7+。Gradle基于Groovy，Groovy需要Java开发环境，Gradle官网说Gradle运行的最低Java JDK版本是7。

   我的Java版本如下
```
$ java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

3. Gradle。Gradle环境中包含Groovy，因此安装Gradle之后就不需要单独安装Groovy了。各平台以及不同安装方式见官网 [Installing Gradle](https://docs.gradle.org/current/userguide/installation.html)。macOS上可以使用`Homebrew`快速安装`$ brew install gradle`，安装成功后，查看版本信息如下：
```bash
$ gradle -v
------------------------------------------------------------
Gradle 4.8.1
------------------------------------------------------------

Build time:   2018-06-21 07:53:06 UTC
Revision:     0abdea078047b12df42e7750ccba34d69b516a22

Groovy:       2.4.12
Ant:          Apache Ant(TM) version 1.9.11 compiled on March 23 2018
JVM:          1.8.0_91 (Oracle Corporation 25.91-b14)
OS:           Mac OS X 10.13.6 x86_64
```



<br>

# Hello World示例
创建一个简单的Gradle项目：包含一个`build.gradle`文件的目录。
```bash
$ mkdir hello-world
$ cd hello-world
$ vim build.gradle
```


`build.gradle`文件

```groovy
task helloworld {
  doLast {
      println'Hello World!'
  }
}
```
执行任务
```zsh
$ gradle helloworld
# 输出如下：
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :helloworld
Hello World!
BUILD SUCCESSFUL in 2s
1 actionable task: 1 executed
```

简单解释下 `build.gradle`文件：
1. `build.gradle`是Gradle的编译脚本，在构建过程中Gradle会据此生成一个`Project`的Java实例。
2. `task`是`Gradle DSL`的关键字，用于添加任务到当前那项目中。`helloworld`是任务的名称。后面连接的一个Groovy闭包，构建时，会返回一个名为‘helloworld’的任务。
3. 闭包中的`doLast`是`Task`的方法，根据闭包的语法，省略了方法的对象，写全的话是`owner.doLast`。表示向任务中添加一个打印'Hello World!'字符串的`Action`到任务的`Action`列表的最后。
4. 一个任务（`Task`）可以包含很多`Action`，执行任务的时候，顺序执行`Action`。



<br>

# 参考
1. [Groovy 官网](http://www.groovy-lang.org/learn.html)
2. [Gradle官网](https://gradle.org/)
3. [Android开发者官网-Configure your build](https://developer.android.com/studio/build/)
4. [Gradle轻松入门](https://blog.csdn.net/innost/article/details/48228651)