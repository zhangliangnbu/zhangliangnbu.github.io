---
title: Android应用开发基础七：其他
date: 2018-08-22 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

记录Android应用开发中涉及到的琐碎知识。

<!-- more -->

# JVM&Dalvik&Art

Java虚拟机。JVM(Java Virtual Machine)是一种软件实现，执行像物理程序的机器。JVM并是不专为Java所实现运行的，只要其他编程语言的编译器能生成Java字节码，那这个语言也能实现在JVM上运行。因此，JVM通过执行Java bytecode可以使java代码在不改变的情况下在各种硬件之上。

Dalvik虚拟机：与Java虚拟机不一样的虚拟机，适应低内存，低处理器速度的移动设备环境。Dalvik虚拟机依赖于Linux内核，实现进程隔离与线程调试管理，安全和异常管理，垃圾回收等重要功能。

Art虚拟机：ART代表AndroidRuntime，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码，运行时编译后的应用代码都需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。
 ART则完全改变了这种做法，在应用安装的时候就预编译字节码到机器语言，这一机制叫Ahead-Of-Time(AOT)预编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。兼容Dalvik。

## Dalvik与Java虚拟机的比较

1. Java虚拟机基于栈，而Dalvik虚拟机基于寄存器。基于栈的机器必须使用指令来载入和操作栈上数据。
2. Java虚拟机运行的是.class文件，而Dalvik运行的是自己专属的.dex字节码格式。
3. Java虚拟机在运行的时候为每一个类装载字节码，Dalvik程序只包含一个.dex文件，这个文件包含了程序中所有的类。
4. 一个应用对应一个Diavik虚拟机实例，独立运行。

## Dalvik与Art的比较

1. Art只会首次启动编译，而Dalvik每次都要编译再运行；
2. Art占用空间比Dalvik大，是用“空间换时间”；
3. Art减少编译，减少了CPU使用频率，明显改善电池续航；
4. Art应用启动更快、运行更快、体验更流畅、触感反馈更及时。

# 主线程 Looper.loop()卡死问题

1. 需要使用死循环阻止线程结束。
2. 没消息时会阻塞，但阻塞是程序处于等待状态，不会无限消耗CPU资源。
3. 有消息时，程序被唤醒去处理消息。

# 参考

- https://www.jianshu.com/p/dcf2ef4a1860
  

