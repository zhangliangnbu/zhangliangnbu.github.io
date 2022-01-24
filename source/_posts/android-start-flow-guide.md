---
title: Android开机启动流程概览
date: 2021-10-24 11:23:41
tags:
- android
categories:
- Android开发
---

# 简介

Android 开机流程，即从用户按开机键到Launcher启动完成这一整个流程。由于Android 系统基于 Linux，因此整个启动流程会包括Linux的启动流程。

Android启动流程，大致分为六个阶段，如下所示：

![android-start-flow-1](/images/android-start-flow-1.png)

- Boot ROM。用于加载并执行Boot Loader。Android 设备上电后，开始执行固化在 CPU芯片里的 Boot ROM 中的预设代码，程序将 Boot Loader 代码加载到 ROM 中。这一步由芯片厂商负责设计和实现。
- Boot Loader。用于将系统代码加载到 RAM 中。具体功能有：检查 RAM、初始化系统的硬件参数等功能，找到 Linux kernel 的代码，设置启动参数，并最终加载到RAM中。
- Kernel。Linux 内核开始启动，初始化各种软硬件环境、加载驱动程序、挂载根文件系统、并执行 init 程序，由此开启 Android 的世界。
- Init。Init 进程负责启动Android系统关键的几个核心进程，其中最关键的是Zygote 进程，Zygote 进程中创建了 ART 或 Dalvik 虚拟机。
- Zygote。Zygote 是 Android 系统的启动进程，Zygote 会启动 System Servers。
- System Servers。是Android系统的核心，负责启动和管理整个Java FrameWork，包含ActivityManagerService， WorkManagerService，PagerManagerService，PowerManagerService等服务。

待服务启动完后，系统开始启动 APP。

<!--more-->

上图对应的系统结构图如下：

![android-start-flow-2](/images/android-start-flow-2.png)

# 参考

- [Android Boot Sequence](https://learnlinuxconcepts.blogspot.com/2014/02/android-boot-sequence.html)
- [Android Internals: The Android OS Bootup Process](https://dzone.com/articles/android-internals-the-android-os-bootup-process)
- [Android Boot Process](https://www.geeksforgeeks.org/android-boot-process/)
- [Android 操作系统架构开篇](http://gityuan.com/android/)
