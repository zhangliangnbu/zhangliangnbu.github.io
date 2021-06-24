---
title: Flutter入门
date: 2018-11-01 09:04:21
tags:
- Flutter
- 入门
categories:
- 其他
---

# 前言

照着官方[get-started](https://flutter.io/get-started/install/)教程做，略作补充。

-----

# 入门

**1 下载SDK安装包并解压**

**2 使用`$ flutter doctor`做检查并安装相关工具**

安装IOS相关工具时，如果报错

```bash
configure: error: Package requirements (libusbmuxd >= 1.1.0) were not met:
```

可以使用如下方法解决[IOS toolchain安装异常解决](https://stackoverflow.com/questions/52602425/libusbmuxd-version-error-during-flutter-install/52604913#52604913)

```bash
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
```

**3 配置IDE，安装插件**

**4 尝试运行项目**

创建、在Android和IOS模拟器上运行、修改字符串并热加载。

**5  写第一个APP** 

分为两部分，入门文档主要介绍第一部分。这两部分的内容都可以在[Google Codelas](https://codelabs.developers.google.com/?cat=Flutter)中找到。

-----

# 入门之坑

* 使用命令行方式`flutter upgrade`升级需要谨慎，有一次升级后，flutter直接挂了。

-- -- --





# 移动端开发演进

详细请看文章[为什么说Flutter是革命性的？](<http://www.infoq.com/cn/articles/why-is-flutter-revolutionary>)

**传统(Android&IOS)**。APP直接调用平台控件和服务。

![传统构架](/images/mobile_dev_oem.png)

**WebView**。APP直接调用平台的WebView以及通过桥接的方式调用服务。

![WebView](/images/mobile_dev_webview.png)

**ReactNative**。JS代码通过桥接的方式调用平台控件和服务。桥接的转化效率较低。

![WebView](/images/mobile_dev_rn.png)

**Flutter**。APP端通过接口直接调用平台的Canvas、Events和服务，转化效率较桥接的方式快几个数量。

![WebView](/images/mobile_dev_flutter.png)

> 上面的图皆来自文章[为什么说Flutter是革命性的？](<http://www.infoq.com/cn/articles/why-is-flutter-revolutionary>)

-----

<br>



# 其他

一切皆控件。所有的组成部分都是控件，包括位置、补白、主题、导航等。

Flutter运行机制。请看文章

- [Flutter原理解析](https://mp.weixin.qq.com/s/CQQXD0TrlbaNWjoClIcDtw)
- [Flutter原理与美团的实践](https://www.jianshu.com/p/e6cd8584fdbb)



# Flutter视图

- [Flutter，什么是 Widgets、RenderObjects 和 Elements](https://juejin.im/post/5b4c6054e51d4519475f1d5d#heading-1)
- [Flutter Dart Framework原理简解](https://www.stephenw.cc/2018/05/28/flutter-dart-framework/)

# Flutter异步编程

大纲：

- 概念澄清：线程进程、并行并发、同步异步、阻塞非阻塞、单线程模型
- 单线程异步事件模型
- API使用及场景实践：本地编写、本地资源读写、网络请求等。



**单线程模型**

仅仅指代码在一个线程里执行，但可以通过工作线程调度某段代码在其他时刻异步触发。



**事件机制**

![](https://segmentfault.com/img/bVxLvF)



参考

- [异步async、await和Future的使用技巧](https://segmentfault.com/a/1190000014396421)
- [JavaScript：彻底理解同步、异步和事件循环(Event Loop)](https://segmentfault.com/a/1190000004322358)
- [JavaScript单线程异步的背后——事件循环机制](https://zhuanlan.zhihu.com/p/27035708)
- [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
- [Flutter/Dart中的异步](https://juejin.im/post/5c4875f86fb9a049ff4e78cf)
- [Dart asynchronous programming: Isolates and event loops](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a)

# 混合开发实践

- [官方-Add Flutter to existing apps](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps#experiment-turn-the-flutter-project-into-a-module)
- [把flutter项目作为aar添加到已有的Android工程上](https://www.kikt.top/posts/flutter/exists/android-as-aar-to-maven/)
- [Android 工程中集成 Flutter Module](https://allenwu.itscoder.com/add-flutter-in-android)

这里考虑将flutter项目作为AAR添加到已有的Android工程里，可以将AAR文件作为本地文件依赖，或者发布AAR作为远程仓库依赖。

### 将flutter项目作为本地AAR依赖

- 创建flutter module项目
- 执行`flutter build apk`，生成aar文件`./.android/Flutter/build/outputs/flutter-release.aar`
- 将上述aar文件放入已有Android工程`app/libs`中，并使用`api fileTree(dir: 'libs', include: ['*.jar', '*.aar'])`依赖
- 在已有Android工程添加flutter初始化代码以及集成FlutterActivity的容器。

关于有三方控件不能使用的情况，我使用的是flutter版本为1.7.8+hotfix.4，直接打包成AAR作为依赖，就可以直接使用，可能flutter当前的版本已经修复了这个问题：在打包aar的时候，会将其依赖的第三方代码也一起打进aar文件里。

另外，[官方-Add Flutter to existing apps](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps#experiment-turn-the-flutter-project-into-a-module) 这篇文章里说将来稳定版flutter发布本地aar仓库非常简单。





# Flutter应用框架BLoC

纲要：

- 概念
- stream bloc
- bloc框架

参考

- [Reactive Programming Streams BLoC](https://www.didierboelens.com/2018/08/reactive-programming---streams---bloc/)
- [Reactive Programming Streams BLoC中文版：Flutter-BLoC-第一讲](https://juejin.im/post/5ca396b5518825440563e0f5)
- [Reactive Programming Streams BLoC中文版：Flutter-BLoC-第二讲]([https://silencezhou.github.io/2019/03/14/Flutter-BLoC-%E7%AC%AC%E4%BA%8C%E8%AE%B2/](https://silencezhou.github.io/2019/03/14/Flutter-BLoC-第二讲/))
- [Reactive Programming Streams BLoC中文版：Flutter-BLoC-第三讲]([https://silencezhou.github.io/2019/04/03/Flutter-BLoC-%E7%AC%AC%E4%B8%89%E8%AE%B2/](https://silencezhou.github.io/2019/04/03/Flutter-BLoC-第三讲/))
- [Bloc框架官方文档](https://felangel.github.io/bloc/#/gettingstarted)
- [谷歌官方视频介绍](https://www.youtube.com/watch?v=PLHln7wHgPE)

----

<br>

#  参考

1. [Flutter官网](https://flutter.io/get-started/install/)
2. [Flutter Github](https://github.com/flutter/flutter)
3. [Flutter中文社区](https://flutter-io.cn/#section-keynotes)
4. [Flutter官网中文翻译](https://flutterchina.club/get-started/install/)
5. [书籍-Flutter实战](https://book.flutterchina.club/)
6. [Flutter从配置安装到填坑指南详解](https://github.com/AweiLoveAndroid/Flutter-learning/blob/master/readme/Flutter%E4%BB%8E%E9%85%8D%E7%BD%AE%E5%AE%89%E8%A3%85%E5%88%B0%E5%A1%AB%E5%9D%91%E6%8C%87%E5%8D%97%E8%AF%A6%E8%A7%A3.md)
7. [IOS toolchain安装异常解决](https://stackoverflow.com/questions/52602425/libusbmuxd-version-error-during-flutter-install/52604913#52604913)
8. [为什么说Flutter是革命性的？](<http://www.infoq.com/cn/articles/why-is-flutter-revolutionary>)
9. [Flutter原理解析](https://mp.weixin.qq.com/s/CQQXD0TrlbaNWjoClIcDtw)