---
title: 项目发布-基础
date: 2018-12-13 09:42:50
tags:
- maven
- jcenter
- bintray
categories:
- 其他
---

# 前言

**发布项目的定义**。发布项目到远程JCenter仓库，准确的说是发布项目构件到JCenter仓库，用英语说是*Publishing artifacts to the JCenter*。本文所说的发布项目都是指发布项目构建后的生成物，即构件(Artifacts)。

**简介**。JCenter是JFrog公司旗下Bintray平台上一个公开的Java仓库。要发布项目到JCenter，首先需要发布项目到Bintray平台，然后才能发布到它的公开库——JCenter。

- Bintray官网： https://bintray.com/
- JCenter仓库地址：https://jcenter.bintray.com/

Bintray平台上可以托管多种类型的库，如比较流行的Maven和npm。

**本文写作目的**。本文目的是介绍如何上传Android Library项目构件到Bintray Maven库，并最终发布到JCenter这一整个流程。其实关于发布项目到JCenter这方面的介绍，网上已经有许多文章了，官网也有用户指南，那为什么我还有写这篇文章呢？

- 博客介绍不全。一般博客只是介绍如何使用Gradle插件如*gradle-bintray-plugin*等上传项目Artifacts，但参数为什么要那样配置？上传失败如何定位问题？等等都没有介绍。如果不使用插件，又该如何做？
- 官方用户指南较凌乱。

**源码地址**。本文涉及到的`nicelogger`项目Github地址：https://github.com/zhangliangnbu/nice-logger

------

# Maven相关

详细介绍请见[Apach Maven](https://maven.apache.org/index.html)。简单讲，Maven就是项目构建和管理的工具，Maven仓库就是利用Maven来管理项目的仓库。

**Maven仓库文件规范**

上传文件到Maven仓库有一定的规范。

对于Android项目，必须上传的文件包括：

- `.aar`文件。[Android ARchive](https://developer.android.com/studio/projects/android-library)，Android构件特有文件，类似于`.jar`文件[Java ARchive](https://en.wikipedia.org/wiki/JAR_(file_format))，但包含一些资源文件。
- `.pom`文件。Maven仓库中必须有的文件，XML格式，包含项目的所有信息，详情请见[POM Reference](https://maven.apache.org/pom.html)。

有些Maven仓库审核比较严格，需要上传另外的两个文件：

- `-sources.jar`文件。Java源码文件。
- `-javadoc.jar`文件。Java文档文件。

**POM文件**

有四个必填的参数，以著名的[OkHttp项目](http://square.github.io/okhttp/)作为示例进行说明如下：

1. `modelVersion`。`POM`版本，一般都写`4.0.0`，支持Maven 2和3。
2. `groupId`。项目所在的项目组标识，如OkHttp所在的项目组标识是`com.squareup.okhttp3`。
3. `artifactId`。项目的标识，如`okhttp`。
4. `version`。项目当前版本号，如OkHttp项目的`3.12.0`。

另外还有一个参数`packaging`，对于Android项目来一般不能缺少，它表示打包方式，默认是`jar`，Android项目一般为`aar`。

> 我们用Gradle解析依赖的时候，会这样写：`implementation 'com.squareup.okhttp3:okhttp:3.12.0'`，其中`com.squareup.okhttp3`就是当前依赖的`groupId`，`okhttp`是依赖的`artifactId`,`3.12.0`是当前要解析的版本`version`。
>
> 注意：`groupId`、`artifactId`和`version`都是非常重要的参数，在上传文件之前必须先定义好，在之后的过程中会用到。

Android项目基本`POM`文件如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.12.0</version>
  <packaging>aar</packaging>
</project>
```



**文件名统一**

发布到JCenter里的四个文件名称必须统一如下：

- `{artifactId}-{version}.aar`
- `{artifactId}-{version}-sources.jar`
- `{artifactId}-{version}-javadoc.jar`
- `{artifactId}-{version}.pom`

---

# 用户手册

见[JFrog用户指南](https://www.jfrog.com/confluence/display/BT/Introduction)。里面详细介绍各种概念、使用、操作等。浏览一遍即可，有许多是用不上的。主要章节如下：

- 了解核心概念：仓库、包和版本。见章节：[Key Concepts](https://www.jfrog.com/confluence/display/BT/Key+Concepts)。
- 了解发布的三个步骤：创建版本、上传、发布。见章节：[Uploading](https://www.jfrog.com/confluence/display/BT/Uploading)、[Uploads](https://www.jfrog.com/confluence/display/BT/Uploads)
- 了解上传的方法：UI手动、cURL、Maven、Gradle。见章节：[Maven Repositories](https://www.jfrog.com/confluence/display/BT/Maven+Repositories)

> 任何个人博客和文章都不能代替官方用户手册，这是必须要读的，它应当是一切非官方文档的参考。

----

# 发布流程

一般发布流程如下：

1. 准备工作。定义参数；准备本地待发布项目；配置Bintray平台账号、仓库、Package、版本。
2. 生成构件文件。本地生成待发布的构件文件(包括POM文件)。
3. 发布到Bintray。上传和发布本地构件文件到Bintray平台。
4. 发布到JCenter。发布Bintray平台上的项目到JCenter。

<br>

---

# 发布方式

发布项目到JCenter有许多方式：

1. UI手动上传。通过Bintray网站UI，手动上传项目构件文件。
2. cURL上传。通过cURL调用REST API接口上传。
3. Maven上传。使用Maven客户端上传项目构件文件到Bintray平台的maven仓库。
4. Gradle插件上传。通[gradle-bintray-plugin](https://github.com/bintray/gradle-bintray-plugin#readme)和[bintray-release](https://github.com/novoda/bintray-release)等Gradle插件上传。

不同的方式会导致发布流程略微有所不同，有些操作可以合并在一起。比如Gradle插件上传方式中，创建pacakge、版本、创建构件文件和上传操作可以一起执行。

<br>

----

# 准备工作说明

主要描述定义参数、准备本地项目和配置Bintray平台这三步操作。

**定义参数**

- Bintray平台仓库名称。你要发布到哪个仓库，当然要知道它的名称了。我这里是`android`。
- Bintray平台Package名称。没有规定，我这里取为`nicelogger`。
- POM文件`groupId`。没有规定，一般为`xxx.xxx.xxx`，我这里取`com.liang.android`。
- POM文件`artifactId`。没有规定，我这里取`nicelogger`。
- POM文件`version`。版本号，我这里取`0.0.1`，作为第一个版本。

> 注意：Bintray平台Package名称和POM文件`artifactId`不必取相同的名称，可以不一致。为了方便说明和记忆，我取为一致。



**准备项目**

可以创建一个然后发布到GitHub或者从GitHub fork一个。为了描述方便工程名称为`NiceLoggerDemo`, Library Module名称为`nicelogger`。

项目的GitHub地址在之后的配置中，需要用到。



**配置Bintray平台**

* 创建账户。如果已经有账户了，就不用创建。在[Bintray官网](https://bintray.com)注册账号，参考[官网指南-Creating an Account](https://www.jfrog.com/confluence/display/BT/Getting+Started)。
* 创建Maven库。如果已经创建过Maven仓库，可以不用创建。进入个人中心主页，点击“Add New Repository”，进入填写仓库信息页。填写信息：“Name”，看其他博客许多人写了“maven”，我写了“android”，表示是一个用于存储Android项目的Maven仓库，这个名字需要记住，以后使用Gradle插件上传项目的时候用得着。“Type”选择“Maven”。其他选填。点击“Create”就可以创建一个Maven仓库了。
* 创建Package。用Gradle插件方式时可以不用手动创建。进入Maven仓库，点击“Add New Package”，进入仓库信息填写页。信息填写：“Name”填之前规定的`nicelogger`，“Version control”填你上传到Github上的项目地址，其他选填。点击“Create Package”即可。
* 创建版本。用Gradle插件方式时可以不用手动创建。从Package页面进入“Create New Version”页面，“Name”填写定义好的`0.0.1`，其他选填，点击“Create Version”即可。

准备工作做好后，接下来就可以通过不同的方式进行后续操作了。

<br>

---

# 参考

1. [JCenter是什么](https://www.geekhub.cn/a/1295.html)
2. [Bintray官网](https://bintray.com)
3. [JCenter仓库](https://jcenter.bintray.com/)
4. [JFrog用户指南](https://www.jfrog.com/confluence/display/BT/Introduction)
5. [Apach Maven](https://maven.apache.org/index.html)
6. [bintray-release](https://github.com/novoda/bintray-release)
7. [使用Gradle插件上传Artifacts到Bintrary](https://github.com/bintray/gradle-bintray-plugin#readme)
8. [使用Gradle插件上传示例](https://github.com/bintray/bintray-examples/tree/master/gradle-bintray-plugin-examples)
9. [博客-上传Gradle项目到Maven仓库](https://blog.csdn.net/xiangzhihong8/article/details/53869957)
10. [从Travis到Bintray](https://www.jianshu.com/p/905411bf4f8f)

<br>

----