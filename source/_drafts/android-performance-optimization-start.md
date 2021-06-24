---
title: Android性能优化-开始
date: 2019-03-30 09:08:34
tags:
- Android
- 性能优化
categories:
- Android开发
---

# 前言

系统地学习下Android性能优化方面的知识。

学习主题：硬件性能和电池寿命、屏幕和UI性能、内存性能、CPU性能、网络性能。

参考资料：

- [《高性能Android应用开发》](https://book.douban.com/subject/26891270/)。(主)
- [Android开发者官方指南-Performance](<https://developer.android.com/topic/performance>)
- [《Android应用性能优化最佳实践》](https://book.douban.com/subject/27036747/)。
- [《Android移动性能实战》](https://book.douban.com/subject/27021800/)。
- [《移动App性能评测与优化》](https://book.douban.com/subject/26891415/)。
- 其他的网络资料。

学习方式：

- 进行主题学习，以《高性能Android应用开发》书籍为主，其他为辅。
- 学习理论和工具使用、分析并优化当前工作中的APP、小结。

---

# 硬件性能和电池寿命

耗电原因

大部分耗电问题与硬件本身无关，而在于APP滥用设备功能。

# 内存性能

纲要

- 内存泄漏-概念、常见场景

- OOM-概念、场景

- 内存泄漏和OOM关系

- APP运行时内存区分

- 内存分析工具

- Jvm垃圾回收触发时机、如何回收 

内存区分

- 共享内存和私有内存。
- 脏内存和干净内存。脏内存指仅存在与RAM中的内存数据，干净内存指即存在于RAM也存在于磁盘中的内存数据。
- [Android进程内存空间](https://www.jianshu.com/p/e425e02c1eb4)

垃圾回收-GC

- 查看GC。调用`Runtime.getRuntime().gc()`可以打印GC日志：*I/xxx: Explicit concurrent copying GC freed 2825(474KB) AllocSpace objects, 1(20KB) LOS objects, 49% free, 1546KB/3MB, paused 1.381ms total 11.205ms*

App较大且复杂时，尽可能少用内存

查看APP可使用的最大内存

```kotlin
val am = getSystemService(ActivityManager::class.java) as ActivityManager
println("memory -> ${am.memoryClass}")
```

查看APP的内存分配信息：` adb shell dumpsys meminfo <pid>`

## 内存泄漏

概念。内存空间使用完毕之后未回收，或者说当应用不再需要某个对象时，仍未释放该对象的所有引用。

Android中几种常见内存泄漏场景如下：

- **长周期对象或变量持有短周期对象引用**。如Application的成员变量、类的静态变量、单例等，持有Activity或Attached View（间接持有Activity）引用。解决：如要使用Context，应该尽可能使用ApplicationContext。
- **使用内部类静态实例**。内部类持有外部类引用，Activity里的内部类会持有Activity引用。
- **使用匿名Handler**。匿名类持有声明它的对象的引用，Handler的消息可能不及时处理，则长久只有外部引用造成泄漏。同理还有使用`Thread`或者`TimerTask`、`AsycTask`，都会出现这种情况。解决：使用弱引用或在Stop或Destroy时清除消息。
- **资源对象未关闭**。资源型对象如Cursor、File文件，使用完后没有关闭，导致缓存没有及时清理。解决：使用完后务必调用`close()`方法。
- **注册对象未注销**。如各种需要回调的SystemService或观察者，如果只注册而没有注销，则可能导致泄漏。（系统好像可以自己注销系统服务，但不及时）。解决：在Activity或Fragment停止或销毁时注销。

> 使用`Leak Canary`只检测到静态对象持有短周期对象引用的泄漏，其他的没有检测到



内存泄漏分析工具

- Heap Dump
- Allocation Tracker
- MAT - 内存泄漏分析
- Leak Canary。[中文使用说明](<https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/>)

Android Studio内存分析工具

- [The Memory Profiler in Android Studio](<https://developer.android.com/studio/profile/memory-profiler.html>)



参考

- [Eight Ways Your Android App Can Leak Memory](<http://blog.nimbledroid.com/2016/05/23/memory-leaks.html>)
- <https://zhuanlan.zhihu.com/p/26043999>