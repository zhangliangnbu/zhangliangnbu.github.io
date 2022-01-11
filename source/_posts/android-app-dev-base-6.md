---
title: Android应用开发基础六：应用性能优化概览
date: 2018-08-21
updated: 2022-01-11
tags:
- android
categories:
- Android开发
---

Android应用的性能优化是一个很大的课题，本文简单介绍性能优化各项指标、引起性能问题的场景以及解决方案。

# 简介

性能优化的内容主要体现在如下几个方面：

- 流畅：启动速度、页面显示速度、响应速度
- 稳定：ANR、崩溃
- 节省：安装包大小、能耗、内存、网络

<!-- more -->

# 启动速度优化

提高应用启动速度，即减少应用启动时间。以下先分析应用启动过程的内部机制，然后介绍如何监测和分析启动过程总耗时和每个阶段的耗时，最后给出优化措施。

应用启动过程分为三类，重点是冷启动，其他两种可理解为是它的局部：

- 冷启动。完全的启动过程，应用从头开始启动，包括系统进程的工作和应用进程的工作。
- 热启动。仅包含冷启动的最后一个工作，系统将后台运行的应用带入前台。
- 温启动。仅包含冷启动的部分工作，比热启动工作多。

## 冷启动耗时分析

中，一般耗时主要发生在应用创建和Activity创建两个阶段，重点一般为`Application.onCreate()`方法和`Activity.onCreate()`方法：

- 应用创建。创建Application对象，调用其`Application.onCreate()`方法。开发者一般会在该方法中进行全局变量或对象的初始化任务，任务过重时会显著增加耗时。
- Activity 创建。应用进程创建Activity后，Activity 会回调周期方法，其中`onCreate()`方法对加载时间的影响最大，因为它执行工作的开销最高：加载和填充视图，以及初始化运行 Activity 所需的对象。

## 耗时监测和分析

**使用 logcat 输出应用启动时间**

输出应用初步显示时间：初步显示的时间，即页面初次显示的时间，不包括延迟加载而导致页面重绘的耗时。在 Android 4.4（API 级别 19）及更高版本中，logcat 包含一个输出行，其中包含名为 `Displayed` 的值。此值代表从启动应用进程到完成 Activity 绘制所用的时间。经过的时间包括以下事件序列：

**使用adb命令输出应用初步显示时间**

```bash
> adb shell am start -S -W com.liang.androidskilldemo/.MainActivity

Stopping: com.liang.androidskilldemo
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.liang.androidskilldemo/.MainActivity }
Status: ok
LaunchState: COLD
Activity: com.liang.androidskilldemo/.MainActivity
TotalTime: 1002  # 表示启动过程中，所有的 Activity 的启动时间，关注 TotalTime 就可以
WaitTime: 1012 # 表示应用进程的创建时间 + TotalTime
Complete
```

## 常见问题及优化措施

常见问题：常见导致启动速度慢的问题为在应用初始化和Activity初始化时做了较多的耗时工作。

优化方向：减少`Application.onCreate()`和`Activity.onCreate()` 耗时。

具体措施：

- 延迟加载不立即使用的对象。比如以下全局单例对象，应该在使用它们时初始化。
- 非主线处理磁盘IO、网络IO、加载和解码位图等耗时工作。线程过多时使用线程池。
- 对于Activity初始化，还需要精简页面布局（降低控件层级、减少控件个数）。

> 💡 可使用CPU Profiler分析方法耗时

# 页面显示优化

界面显示是指从应用生成帧并将其显示在屏幕上的动作。要确界面显示流畅，应用应达到每秒 60 帧的呈现速度，即呈现每帧的时间不超过 16ms。界面呈现慢即界面的某帧在16ms内没有绘制完并显示处理而导致跳帧，让用户感觉界面显示不流畅，这种情况称为“卡顿”。

界面绘制原理简介：在界面绘制的过程中， CPU 主要的任务是计算出屏幕上所有 View 对应的图形和向量等信息。 GPU 的主要任务就是把 CPU 计算出的图形栅格化并转化为位图，可以简单理解为屏幕像素点对应的值。如果操作过程中卡顿了，一般就是 CPU 和 GPU 其中的一个或者多个无法短时间完成对应的任务。

一般而言，CPU 除了需要计算 View 对应的图形和向量等信息，还要做逻辑运算和文件读写等任务，所以 CPU 造成卡顿更常 见。一般也是通过减少 CPU 的计算任务来优化卡顿。

## 卡顿本质

所谓卡顿，即在 16.6ms 内 CPU 无法完成相应的计算，从而造成重复显示同一帧的现象。影响屏幕绘制的因素有：

1. 绘不完。绘制任务繁重，需要处理大量的数据，时间大于 16.6ms；
2. 来不及绘。主线程过于繁重，导致在 Vysnc 信号来时没有准备好数据，导致丢帧；

## 优化措施

- **避免在主线程做计算、IO等耗时操作**。避免频繁使用`SharePerfrences`；减少从`xml`文件读取数据的操作等。
- **避免频繁创建对象而导致频繁GC**。避免循环里面使用 "+" 拼接字符串；避免ArrayList 等有容积限制的容器类初始化的容量不合理，导致后续新增数据频繁扩容，等等。
- **减少View.onDraw()耗时**。避免创建大量对象，避免循环操作
- **避免过度绘制**。1. 降低布局层级。可以利用 ViewStub 、merge 和 include 等标签来尝试减少布局层次，或者使用高效的布局容器，比如 ConstraintLayout，可以不用嵌套布局容器来实现复杂效果。2. 移除控件中不必要的背景。3. 部分效果可以考虑用自定义 View 实现。

## 调试工具

- Hierarchy Viewer。Android Studio 提供的UI性能检测工具，检测层级。
- Profile GPU Rendering。工具在设置-开发者选项-Profile GPU rendering选项，打开后选择on screen as bars。
- Systrace。

# 避免ANR

ANR，即Application Not Response，应用无响应。如果 Android 应用的UI线程处于阻塞状态时间过长，就会触发“应用无响应”(ANR) 错误，当应用处于前台时系统会弹出一个ANR提示对话框。

## 产生ANR的原因

系统根据以下响应时间内没有响应或处理完事件，就会触发ANR：

- 按键或触摸事件在5秒内无响应。这是发生最多的情况。
- BroadcastReceiver在10秒内无法处理完成
- Service在20秒内无法处理完成。

一般具体情况表现为，UI处理耗时任务，导致触摸事件或绘制操作无法在规定时间内完成，从而触发ANR。

主线程阻塞的具体情况有：

1. 应用在主线程上非常缓慢地执行涉及 I/O 的操作。
2. 应用在主线程上进行长时间的计算。
3. 主线程在对另一个进程进行同步 binder 调用，而后者需要很长时间才能返回。
4. 主线程处于阻塞状态，为发生在另一个线程上的长操作等待同步的块。
5. 主线程在进程中或通过 binder 调用与另一个线程之间发生死锁。主线程不只是在等待长操作执行完毕，而且处于死锁状态。

## 避免ANR的做法

主要针对UI线程阻塞的情况

1. UI线程尽量只做跟UI相关的工作
2. 耗时的操作(I/O、网络连接、计算等)，放在单独的线程处理。
3. 用Handler、AsyncTask等处理UI线程和其他线程之间的交互。

## 检测ANR

- BlockCanary。ANR检测框架。原理为，在Looper循环处理每一个消息的前后打点记时，看是否超过5秒，超过即触发警告。
- 启用后台 ANR 对话框。系统会显示后台应用的ANR弹框。
- StrictMode。作为开发工具的Java类，用于检测主线程的IO操作和网络连接。

- 查看跟踪信息文件。Android将ANR信息存储到跟踪信息文件里。低版本为 `/data/anr/traces.txt`，高版本为`/data/anr/anr_*` 文件。前提是设备可以root：
- 使用 Systrace 和 Traceview 等性能工具。分析TraceView文件，查看耗时方法。

# 避免崩溃

常见的Android 应用崩溃（App Crash）是由未处理的异常导致的。异常可分为有两类，一类是Java Exception异常，一类是Native Signal异常。当应用崩溃时，Android 会终止应用的进程并显示一个对话框，告知用户应用已停止。

避免崩溃的手段是尽可能早地检测异常即尽早捕获异常，分析异常并作出相应的处理。其中重点在于捕获异常。

> 💡 注意：本文主要讨论线上版本APP崩溃的情况，开发阶段的崩溃通过Logcat即可获取异常堆栈信息并作出相应处理。

## Java异常机制及捕获

Java异常分为两类：Checked Exception和UnChecked Exception，前者一般使用try…catch显式捕获处理即可，引起崩溃的往往是后者。

**Checked Exception**

也叫编译时异常，编译器会强制处理的异常。Java认为这类异常都是可以被处理（修复）的，需要用try…catch显式捕获并处理。在Java API文档的方法说明中，都会添加是否throw某个exception，这个exception就是Checked异常。如果没有try…catch这个异常，则编译出错，错误提示类似于“Unhandled exception type xxxxx”。

**UnChecked Exception**

也叫运行时异常，所有RuntimeException类及其子类的实例都属于此类，最常见的例子如NullPointerException。这类异常一般没有用try…catch处理，称为Uncaught异常，当Uncaught异常发生时就会导致应用崩溃。Java 提供了一个接口 UncaughtExceptionHandler 可用于处理这类异常，避免应用崩溃。

Uncaught异常发生时会终止线程，此时，系统会通知`UncaughtExceptionHandler`，告诉它被终止的线程以及对应的异常，然后调用`uncaughtException`函数。如果该handler没有被显式设置，则会调用对应线程组的默认handler。

在任何线程中，都可以设置该handler，但在Android应用程序中，全局的Application和Activity、Service都同属于UI主线程，线程名称默认为“main”。所以，在Application中应该为UI主线程添加UncaughtExceptionHandler，这样整个程序中的Activity、Service中出现的UncaughtException事件都可以被处理。

```java
// 在主线程设置 UncaughtExceptionHandler
public void onCreate(){
    super.onCreate();
    Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
}
```

如果多次调用setDefaultUncaughtExceptionHandler设置handler，以最后一次为准。因此如果有多个抓崩溃的SDK或代码同时使用，需要通过`Thread.getDefaultUncaughtExceptionHandler()` 保留上个handler并调用它。

捕获Exception之后，通过异常对象的printStackTrace方法打印异常的堆栈信息，找到异常的源头。

```java
public static String getStackTraceInfo(final Throwable throwable) {
    String trace = ""; 
    try {
        Writer writer = new StringWriter();
        PrintWriter pw = new PrintWriter(writer);
        throwable.printStackTrace(pw);
        trace = writer.toString();
        pw.close();
    } catch (Exception e) { 
        return "";
    } 
    return trace;
}
```

## Native异常机制及捕获

Android平台可以通过C/C++编写动态链接库（so），Java可以利用JNI来调用so库。

C++可以通过try…catch去处理一些异常，但如果出现了Uncaught异常，so库就会引起崩溃并产生信号异常，通过捕获信号异常可以捕获Android Native崩溃信息。要对某个信号进行特殊处理，需要注册相应的信号处理函数。具体请看参考链接里的资料。

## 常用三方SDK

- [腾讯 Bugly](https://bugly.qq.com/v2/)
- [听云](https://www.tingyun.com/tingyun_apm.html)

# APK瘦身

APK 的大小会影响应用加载速度、使用的内存量以及消耗的电量。APK瘦身即是对APK大小进行压缩，减小APK安装包大小，更小的安装包更有助于吸引用户安装。

## APK构成

APK 文件由一个 Zip 压缩文件组成，其中包含构成应用的所有文件。这些文件包括 Java 类文件、资源文件和包含已编译资源的文件。在Android Studio里双击apk文件或用zip工具打开apk文件，可大概查看apk文件的构成。

APK 包含以下目录和文件：

- `META-INF/`：包含 `CERT.SF` 和 `CERT.RSA` 签名文件，以及 `MANIFEST.MF` 清单文件。
- `assets/`：包含应用的资源；应用可以使用 `AssetManager` 对象检索这些资源。
- `res/`：包含未编译到 `resources.arsc` 中的资源。
- `lib/`：包含特定于处理器软件层的已编译代码。此目录包含每种平台类型的子目录，如 `armeabi`、`armeabi-v7a`、`arm64-v8a`、`x86`、`x86_64` 和 `mips`。
- `resources.arsc`：包含已编译的资源。此文件包含 `res/values/` 文件夹的所有配置中的 XML 内容。打包工具会提取此 XML 内容，将其编译为二进制文件形式，并压缩内容。此内容包括语言字符串和样式，以及未直接包含在 `resources.arsc` 文件中的内容（例如布局文件和图片）的路径。
- `classes.dex`：包含以 Dalvik/ART 虚拟机可理解的 DEX 文件格式编译的类。
- `AndroidManifest.xml`：包含核心 Android 清单文件。此文件列出了应用的名称、版本、访问权限和引用的库文件。该文件使用 Android 的二进制 XML 格式。

对各文件目录以及文件大小进行优化，即可优化最终的apk大小。

## 资源的优化

主要方向为缩减资源的数量和大小。

- 移除未使用的资源。可使用 Android Studio 中附带的静态代码分析器Lint检测到 res/ 文件夹中未被代码引用的资源。
- 移除重复使用资源。可以为图片的变体添加单独的资源文件或设置属性，而非新增一个单独的图片。例如同一图片经过色调调整、阴影设置或旋转的版本。
- 用代码渲染。简单图片可以用代码进行绘制、渲染，从而避免存储图片而减少apk尺寸。
- 图片优化。在Android下支持很多格式的图片，例如：[PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics)、[JPG](https://en.wikipedia.org/wiki/JPEG) 、[WebP](https://en.wikipedia.org/wiki/WebP)，那我们该怎么选择不同类型的图片格式呢？ 准则优先级从上往下依次为：
  1. 能用矢量图形表示且图片较小时，使用VectorDrawable。
  2. 如果支持WebP则优先用WebP
  3. PNG主要用在展示透明或者简单的图片
  4. 其它场景可以使用JPG格式
- 开启资源压缩。Android的编译工具链中提供了一款资源压缩的工具，可以通过该工具来压缩资源，如果要启用资源压缩，可以在`build.gradle`文件中将`shrinkResources`设置为`true`。
- 语言资源优化。根据App自身支持的语言版本选用合适的语言资源，例如使用了AppCompat，如果不做任何配置的话，最终APK包中会包含AppCompat中所有已翻译语言字符串，无论应用的其余部分是否翻译为同一语言，可以通过`build.gradle`里的`resConfig`来配置使用哪些语言，从而让构建工具移除指定语言之外的所有资源。

## classes.dex的优化

如何优化classes.dex的大小呢？大体有如下方向：

- 时刻保持良好的编程习惯和对包体积敏锐的嗅觉，去除重复或者不用的代码，慎用第三方库，选用体积小的第三方SDK等等。
- 开启ProGuard来进行代码压缩，通过使用ProGuard来对代码进行混淆、优化、压缩等工作。

## 其他优化

- 减少 so 库资源大小。只编译指定平台的 so。一般我们都是给 arm 平台的机器开发，如果没有特殊情况，我们一般只需要考虑 arm 平台的。

# 内存泄漏及解决

内存泄漏（memory leak）：对于Java来说，就是new出来的对象无法被GC回收。内存泄漏发生时的主要表现为内存抖动，可用内存慢慢变少。

![android-memory-leak](/images/android-memory-leak.png)

在Android中，造成内存泄漏的情形有：

1. 使用全局静态变量持有`Activity`的强引用，而没有及时清空。
2. 活在`Activity`生命周期之外的对象或线程，持有`Activity`的强引用，而没有及时清空。
3. 忘记释放资源，比如忘记关闭`Cursor`，或忘记反注册传感器等。

## 场景和方案

**通常的解决方案**

不管内存泄漏的代码表现形式如何，其核心问题在于，在Activity生命周期之外仍持有其引用。

- static。Activity销毁之前置空即可。
- 内部类或匿名内部类、Handler。将子类声明为静态内部类或外部类，对Activity的强引用改为弱引用。
- Thread、TimerTask。如果坚持使用匿名类，只要在生命周期结束时中断线程就可以。
- 服务监听。在Activity结束时注销监听器。

**具体情形及解决方案如下：**

- 静态Activity。在类中定义了静态Activity变量，把当前Activity实例赋值给这个静态变量。解决方案，销毁时置空。
- 静态View。如果一个View初始化耗费大量资源，而且在一个Activity生命周期内保持不变，那可以把它变成static，加载到视图树上(View Hierachy)，像这样，当Activity被销毁时，应当释放资源。
- 内部类。Activity中有个内部类，如果一个静态变量持有该内部类实例就容易造成内存泄漏。解决方案，销毁时置空。
- 匿名内部类。匿名内部类和内部类一样，都会持有外部类的强引用，其内存泄漏原理同内部类的错误使用一样。解决方案，销毁时置空。
- Handler。两种方式造成内存泄漏，一种是作为内部类或匿名内部类而被静态变量持有，一种是消息没有及时消费导致Activity实例不能被销毁，于是发生内存泄漏。解决方案，一，定义Handler为静态内部类或外部对象，二，对Activity的强引用改为弱引用。
- Thread。作为匿名内部类或内部类持有Activity的引用，线程没有结束时Activity都不能销毁。解决方案，销毁时停止并销毁线程，或改为持有Activity的弱引用。
- TimerTask。只要要是匿名类的实例，不管是不是在工作线程，都会持有Activity的引用，导致内存泄漏。
- 传感器服务类。注册系统服务，销毁时需要反注册，否则导致内存泄漏。

## 使用Leakcanary检测

Leakcanary基本原理：

1. 在一个Activity执行完onDestroy()之后，将它放入WeakReference中，
2. 然后将这个WeakReference类型的Activity对象与ReferenceQueue关联。
3. 从ReferenceQueue中查看是否存在该对象，如果还存在，执行gc，再次查看，已久存在则判断发生内存泄露了。
4. 最后用HAHA这个开源库去分析dump之后的heap内存。

# 内存溢出及解决

内存溢出，OOM，Out Of Memory，一般指内存占有量超过了VM所分配的最大值的现象。Android系统的app的每个进程或者每个虚拟机有个最大内存限制。

**产生OOM的原因和场景**

1. 加载对象过大
2. 相应资源过多，来不及释放，比如过多的内存泄漏。注意，内存泄漏并一定导致内存溢出，但频繁的内存泄漏最终对导致内存溢出。

**分析和解决**

1. 在内存引用上做些处理，常用的有软引用、强化引用、弱引用
2. 在内存中加载图片时直接在内存中作处理，如边界压缩
3. 动态回收内存
4. 优化Dalvik虚拟机的堆内存分配
5. 自定义堆内存大小

# 参考

- [官方-应用启动时间](https://developer.android.google.cn/topic/performance/vitals/launch-time?hl=zh-cn#internals)
- 三方博客-[加快启动速度](https://blog.csdn.net/carson_ho/article/details/79708444)
- [三方博客-优化启动速度](https://juejin.cn/post/6950608825942868004#heading-10)
- [深入探索Android启动速度优化](https://jsonchao.github.io/2019/11/10/深入探索Android启动速度优化/)
- [官网-渲染速度缓慢](https://developer.android.google.cn/topic/performance/vitals/render?hl=zh-cn)
- [官网-优化布局层次结构](https://developer.android.google.cn/training/improving-layouts/optimizing-layout?hl=zh-cn)
- [冻结的帧](https://developer.android.google.cn/topic/performance/vitals/frozen)
- [深入理解 Android 屏幕绘制原理](https://leegyplus.github.io/2020/06/16/深入理解-Android-屏幕绘制原理/)
- [操作流畅度优化](https://juejin.cn/post/6950608825942868004#heading-24)
- [深入探索Android卡顿优化（上）](https://juejin.cn/post/6844904062610046990#heading-9)
- [Android性能优化典范之Profile GPU Rendering](https://www.jianshu.com/p/061bb80025c7)
- **[性能工具Systrace](http://gityuan.com/2016/01/17/systrace/)**
- [Android 开发指南-ANR](https://developer.android.google.cn/topic/performance/vitals/anr?hl=zh-cn)
- [BlockCanary — 轻松找出Android App界面卡顿元凶](http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/)
- [官方-崩溃](https://developer.android.google.cn/topic/performance/vitals/crash)
- [Android 平台的崩溃捕获机制及实现](https://toutiao.io/posts/euc9um/preview)
- [如何治理 Crash](https://juejin.cn/post/6950608825942868004#heading-14)
- [Android 平台 Native 代码的崩溃捕获机制及实现](https://www.cnblogs.com/mingfeng002/p/9118253.html)
- [google-breakpad - 使用篇](https://juejin.cn/post/6899070041074073614)
- [优雅的处理Android崩溃（一）](https://www.jianshu.com/p/8461d977e148)
- [Android崩溃优化（崩溃分类、原理分析以及解决）](https://juejin.cn/post/6844903839292719117)
- [官方-缩减应用大小](https://developer.android.google.cn/topic/performance/reduce-apk-size?hl=zh-cn)
- [美团APP包瘦身优化实践](https://tech.meituan.com/2017/04/07/android-shrink-overall-solution.html)
- [APK瘦身](https://juejin.cn/post/6950608825942868004#heading-1)
- https://www.jianshu.com/p/ac00e370f83d
- https://blog.nimbledroid.com/2016/09/06/stop-memory-leaks.html
