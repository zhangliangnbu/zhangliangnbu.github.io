---
title: Android内存泄漏情形及解决
date: 2021-07-01 11:34:22
tags:
- android
- 内存泄漏
- 性能优化
categories:
- Android开发
---

## **前言**

内存泄漏：对于Java来说，就是new出来的对象无法被GC回收。内存泄漏发生时的主要表现为内存抖动，可用内存慢慢变少。

![https://pic3.zhimg.com/80/v2-4483280b8b2bab984b40251648f92222_720w.png](C:\Users\zhang\Projects\blog\source\_posts\Android内存泄漏情形及解决.assets\v2-4483280b8b2bab984b40251648f92222_720w.png)

在Android中，造成内存泄漏的情形有：

1. 使用全局静态变量持有`Activity`的强引用，而没有及时清空。
2. 活在`Activity`生命周期之外的对象或线程，持有`Activity`的强引用，而没有及时清空。
3. 忘记释放资源，比如忘记关闭`Cursor`，或忘记反注册传感器等。

<!-- more -->

## **具体场景和解决方案**

- 静态Activity。在类中定义了静态Activity变量，把当前Activity实例赋值给这个静态变量。解决方案，销毁时置空。
- 静态View。如果一个View初始化耗费大量资源，而且在一个Activity生命周期内保持不变，那可以把它变成static，加载到视图树上(View Hierachy)，像这样，当Activity被销毁时，应当释放资源。
- 内部类。Activity中有个内部类，如果一个静态变量持有该内部类实例就容易造成内存泄漏。解决方案，销毁时置空。
- 匿名内部类。匿名内部类和内部类一样，都会持有外部类的强引用，其内存泄漏原理同内部类的错误使用一样。解决方案，销毁时置空。
- Handler。两种方式造成内存泄漏，一种是作为内部类或匿名内部类而被静态变量持有，一种是消息没有及时消费导致Activity实例不能被销毁，于是发生内存泄漏。解决方案，一，定义Handler为静态内部类或外部对象，二，对Activity的强引用改为弱引用。
- Thread。作为匿名内部类或内部类持有Activity的引用，线程没有结束时Activity都不能销毁。解决方案，销毁时停止并销毁线程，或改为持有Activity的弱引用。
- TimerTask。只要要是匿名类的实例，不管是不是在工作线程，都会持有Activity的引用，导致内存泄漏。
- 传感器服务类。注册系统服务，销毁时需要反注册，否则导致内存泄漏。

## **使用Leakcanary检测**

**Leakcanary**基本原理：

- 在一个Activity执行完onDestroy()之后，将它放入WeakReference中，
- 然后将这个WeakReference类型的Activity对象与ReferenceQueue关联。
- 从ReferenceQueue中查看是否有没有该对象，如果还存在，执行gc，再次查看，还是有的话则判断发生内存泄露了。
- 最后用HAHA这个开源库去分析dump之后的heap内存。

## 通常的解决方案

不管内存泄漏的代码表现形式如何，其核心问题在于：在Activity生命周期之外仍持有其引用。

解决方法：

- static。Activity销毁之前置空即可。
- 内部类或匿名内部类、Handler。将子类声明为静态内部类或外部类，对Activity的强引用改为弱引用。
- Thread、TimerTask。如果坚持使用匿名类，只要在生命周期结束时中断线程就可以。
- Sensor Manager。在Activity结束时注销监听器。

## **参考**

- https://www.jianshu.com/p/ac00e370f83d
- https://blog.nimbledroid.com/2016/09/06/stop-memory-leaks.html
