---
title: Android构建05-Gradle最佳实践
date: 2018-10-22 09:09:26
update: 2018-10-22 09:09:26
tags:
- android
- build
- gradle
categories:
- Android开发
---



# 文件分布简述

详情请见官方文档[Directory Layout](https://docs.gradle.org/current/userguide/directory_layout.html)。

之前官方文档并没有这方面的说明，现在却单独用一个章节来做说明，可能考虑到许多开发者都遇到过这方面的疑惑吧：项目的依赖包在哪里？我去掉了项目里`gradle.properties`和IDE（如Android Studio）里的代理设置之后，会什么代理还会生效？等等。

Gradle相关文件包括Wrapper文件、Gradle各版本包、依赖包等。它们并不都存放在项目目录里，如依赖包，你在项目根目录下是找不到的。

Gradle相关文件分布在两个地方：

1. 项目根目录。这个目录下的gradle相关文件只对当前项目生效。包括Wrapper配置和可执行命令文件、项目的`gradle.properties`、`build.gradle`等构建和配置文件。
2. 用户目录。`$USER_HOME/.gradle`，macOS中执行`$ cd ~/.gradle`即可进入，该目录下的配置对所有的项目生效。包括配置文件`gradle.properties`、Gradle各版本`wrapper/dists/`、项目依赖包`~/.gradle/caches/modules-2/files-2.1/`等。

-- -- --

<br>



# 问题排查



**获取帮助**

1. [Gradle Forums](https://discuss.gradle.org/c/help-discuss)。更新缓慢，回复率很低。
2. [StackOverflow #gradle](https://stackoverflow.com/questions/tagged/gradle) 。在StackOverFlow上一般能找到所需要的解答。
3. [help.gradle.org](https://help.gradle.org/)。没用过，估计和Gradle论坛一样吧。



**安装问题**

详情请见官网[Troubleshooting Gradle installation](https://docs.gradle.org/current/userguide/troubleshooting.html#sec:troubleshooting_installation)，按照官网的教程安装，一般没有什么问题。



**定位一般构建问题**

1. 执行`$ ./gradlew [task name] -s`。打印栈信息，一般可以定位问题出现在哪里，否则执行下一步操作。
2. 执行`$ ./gradlew assembleDebug -d > gradle.log `。打印所有的构建信息到`gradle.log `文件，通过搜索`WARNING`或`ERROR`定位错误。



**依赖解析问题**

参考官网[Troubleshooting Dependency Resolution](https://docs.gradle.org/current/userguide/troubleshooting_dependency_resolution.html)，补充如下：

1. 版本冲突。升级低版本或在某个依赖里排除掉(`exclude`)低版本。
2. 依赖下载失败。一般会报错`Could not resolve xxx....xxxx timed out`，有以下两种解决方法：
   * 设置HTTP代理或国内镜像仓库。HTTP代理可在`$USER_HOME/.gradle/gradle.properties`或项目根目录`gradle.properties`里设置。镜像仓库可使用[阿里云的仓库](http://maven.aliyun.com/mvn/view)，包括google、Maven Central和jcenter等的镜像仓库。
   * 下载依赖并从缓存执行构建。从官网下载依赖并放在缓存(`$USER_HOME/.gradle/caches/jars-3或modules-2`)里，然后使用命令行构建时添加`--offline`可选项。

-- -- --

<br>



# 代理设置

Gradle的代理一般用于解决Gradle版本和依赖下载失败的问题，代理参数可以设置在两个地方：

* 项目根目录下的`gradle.properties`。仅对当前项目生效。
* 用户目录下的`$USER_HOME/.gradle/gradle.properties`。对所有的Gradle项目生效。

我的代理策略是使用[Privoxy](https://www.privoxy.org/)做HTTP代理，再让Privoxy监听Shadowsocks端口，真正实现代理的是本地Shadowsocks与远程代理服务中Shadowsocks的通信。

> 因为Shadowsocks走的是Socks协议，而有些资源一定需要HTTP代理才生效，故需要使用Privoxy做一下转换。

我一般的做法是在`$USER_HOME/.gradle/gradle.properties`下设置代理，代理参数及说明如下：

```properties
# https代理
# 代理主机地址。本地，因为是先要走本地Privoxy。
systemProp.https.proxyHost=127.0.0.1
# 代理端口号。Privoxy的端口号
systemProp.https.proxyPort=8118
# 不走代理的地址。公司内部Maven库、内网、本地、阿里云OSS文件服务、个推、eclipse资源等，不用走代理，否则连接缓慢或者失败
systemProp.https.nonProxyHosts=maven.xxxx.com|192.168.*|localhost|oss.sonatype.org|mvn.gt.igexin.com|repo.eclipse.org

# http代理。与https代理参数一直。
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=8118
systemProp.http.nonProxyHosts=maven.xxxx.com|192.168.*|localhost|oss.sonatype.org|mvn.gt.igexin.com|repo.eclipse.org
```



> IDE以Andorid Studio 为例，其代理设置（在`Preferences->System Settings->HTTP Proxy`中）仅仅对IDE更新、下载SDK等生效，不会影响到Gradle构建时下载依赖。
>
> 需要注意的是，Andorid Studio中设置了代理后，一般会弹出对话框，问是否要把Studio的代理设置到用户目录下的`$USER_HOME/.gradle/gradle.properties`中，如果你点击了确定，则Studio会把自己的代理配置更新到用户目录的`gradle.properties`文件中，这样代理就对Gradle生效了。一般不建议点确定，因为Studio和Gradle的代理的资源不一样，配置也会不一致，有时候一个可能需要代理，而另一个却不需要代理。
>
> IDE和Gradle的代理应该分别配置。



-- -- --

<br>



# 待续

撰写可维护的构建脚本

[Authoring Maintainable Build Scripts](https://docs.gradle.org/current/userguide/authoring_maintainable_build_scripts.html)



搭建Gradle项目

[Organizing Gradle Projects](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html)



提高构建性能

[Improving the Performance of Gradle Builds](https://guides.gradle.org/performance/)



使用缓存

[https://guides.gradle.org/using-build-cache/](https://guides.gradle.org/using-build-cache/)



-- -- --

<br>

# 参考



1. [Troubleshooting](https://docs.gradle.org/current/userguide/troubleshooting.html)
2. [Troubleshooting Dependency Resolution](https://docs.gradle.org/current/userguide/troubleshooting_dependency_resolution.html)





