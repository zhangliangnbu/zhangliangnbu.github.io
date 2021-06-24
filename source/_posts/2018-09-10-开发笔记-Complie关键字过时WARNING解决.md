---
title: 开发笔记-Complie关键字过时警告问题
date: 2018-09-10 16:29:15
updated: 2018-10-17 14:52:48
tags:
- Android
- Gradle
- Build
- 开发笔记
categories:
- Android开发
---





# 问题描述

之前构建Android项目时，经常出现以下WARNING：

```bash
WARNING: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'.
It will be removed at the end of 2018. For more information see: http://d.android.com/r/tools/update-dependency-configurations.html
```

但我检查了APP项目以及所有的Module，都没有发现`compile`关键字，用全局搜索（command + shift + f）也没有查到，所以应该是某个远程依赖或插件中使用了`compile`关键字，但如何定位是哪一个呢？

通过IDE`Sync`后的提示如下：

{% asset_image warning1.png  Sync后的Warning提示 %}



<br>

# 问题分析和解决



IDE给的提示太简略，只能通过Gradle命令和[Logging options](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_debugging)来打印详细的构建信息来定位问题。Logging级别从简略到详细依次为`-q`、`-w` 、`-i`、`-d`。IDE默认应该是使用`-q`的，可以逐级打印信息，也可以直接使用`-d`级别日志打印信息。

在项目根目录下，执行命令

```
./gradlew assembleDebug -d > gradle.log
```

其中`-d`是为了打印构建的Debug级别的信息，非常详细。由于信息非常多，命令行界面不能一屏完全显示，故放入名为`gradle.log`的文件中。

最后生成的文件有13M多，5万多行。通过搜索关键字"WARNING"，定位信息如下：

```bash
...
16:03:26.459 [DEBUG] [org.gradle.internal.progress.DefaultBuildOperationExecutor] Build operation 'Apply plugin com.sensorsdata.analytics.android to project ':app'' started
16:03:26.459 [WARN] [org.gradle.api.Project] WARNING: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'.
It will be removed at the end of 2018. For more information see: http://d.android.com/r/tools/update-dependency-configurations.html
...
```

警告信息出现在"Build operation Apply plugin com.sensorsdata.analytics.android to project…"语句下，那很可能就是Gradle在应用这个插件的时候报的错。

插件是神策统计的，我的版本是`1.0.2`，查看官网给的[Mavne库](https://dl.bintray.com/zouyuhan/maven)，查看最新版本，`android-gradle-plugin2/`目录下，最高版本是`2.0.2`。

更新插件版本，重新构建，警告消失了，问题解决了。



> 文章首发[我的个人博客](https://zhangliangnbu.github.io/)和[简书](https://www.jianshu.com/)