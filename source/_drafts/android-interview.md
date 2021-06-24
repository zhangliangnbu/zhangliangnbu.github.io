---
title: Android题目
tags:
- android
- 面试
categories:
- 其他
---

# 前言

基础知识都要熟悉，广度上有所了解，部分知识点深度上要掌握作为自己擅长的技能。

---

# Android

## Android事件机制

示意图

![示意图](https://img-blog.csdn.net/20160918183450879?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

伪代码

```kotlin
public fun dispatchTouchEvent(ev: MotionEvent): Boolean {
    var consume = false
    if (onInterceptTouchEvent(ev)){
        if(!View.OnTouchListener.onTouch(ev)) {
            consume = onTouchEvent(ev)
        }
    }else{
        consume = child.dispatchTouchEvent(ev)
    }
    return consume
}
```



## View绘制

Measure -> Layout -> Draw。View绘制与Activity生命周期不同步。

Measure结束后可获取mMeasuredWidth和mMeasuredHeight，一般与最终尺寸相等。

Layout结束后，方可获取最终的View尺寸。

MeasureSpec。SpecMode（UNSPECIFIED、EXACTLY、AT_MOST） + SpecSize

- DecorView的MeasureSpec由窗口尺寸和自身LayoutParams共同决定。
- 普通View的MeasureSpec由父容器的MeasureSpec和自身LayoutParams共同决定。
- 一般规则：固定宽高->EXACTLY、LayoutParams.MATCH_PARENT->父容器SpecMode、LayoutParams.WRAP_CONTENT->AT_MOST



## IPC通信

**进程间通信**特点

1. 存在中继区域。间接通信必须存在进程都能访问的中继区域，如进程中可共享的*内核空间*、Binder驱动。
2. 数据序列化。不能直接调用数据，需要复制，数据传递的对象必须可序列化。

进程间通信可能出现的问题如下：

1. 静态成员和单例模式失效不共用。不是同一个对象。
2. 同步锁失效。不同进程锁的不是同一个对象。
3. SharedPreferences可靠性下降。SharedPreferences不支持多并发写操作，无法保持同步。
4. Application对象会创建多次。启动一个进程，分配一个虚拟机，然后启动一个应用。

**序列化-Serializable vs Parcelable**

1. Serializable。使用简单，开销大。serialVersionUID保证同一个对象。对象序列化，大量I/O操作，临时变量多，频繁GC。
2. Parcelable。使用略复杂，开销小。官方建议最好不要持久化到disk或网络传输，因为parceble数据一旦更改，反序列化就会失败，不能保持同一性。将一个完整的对象进行分解。
3. 总结。本地IPC使用Parcelable，持久化到disk和网络传输使用Serializable。

**Binder**

1. 不同角度：一种Android中IPC的方式（机制）、IPC中用于连接各进程的Binder驱动（抽象）、实现`IBinder`的`Binder`类（代码实现）
2. Binder机制。使用Binder驱动，连接C/S/SM进程；使用内存映射，实现复制一次数据后就可以进行共享。

**IPC方式**。文件、Socket、（Bundle、Messager、AIDL）（Binder）、ContentProvider

参考

1. [Android跨进程通信：图文详解 Binder机制 原理](https://blog.csdn.net/carson_ho/article/details/73560642)
2. [彻底理解Android Binder架构](http://gityuan.com/2016/09/04/binder-start-service/)



## 多线程通信

**几种方式**

1. 共享内存（变量）
2. 文件，数据库
3. Handler
4. Java里的wait()，notify()，notifyAll()
5. 管道

**Handler Looper MessageQueue**

1. 准备工作。A线程创建`Looper`、`Handler`、执行`Looper.loop()`（其中会调用`MessageQueue.next()`阻塞线程）。
2. 通信。B线程调用Handler发送消息，消息入队，`loop()`中获取消息并分发。
3. 阻塞和ANR。`Looper.loop()`会阻塞线程，但不会卡死，也不会ANR。ANR是事件分发后没有及时处理而导致的。
4. 主线程`Looper.loop()`。阻塞主线程，保持主线程处于运行状态；Activity周期方法是通过发送消息在主线程`Handler.handleMessage()`中调用的。
5. 延时消息。有延时消息时在`MessageQueue.next()`中执行`nativePollOnce()`休眠线程，有即时消息过来时则在`MessageQueue.enqueueMessage()`中执行`nativeWake()`唤醒线程。

```java
// 准备工作
Looper.prepare();
mHandler = new Handler() {
	public void handleMessage(Message msg) {}
};
Looper.loop();
```



## 题目

点击一个图标到这个应用启动的全过程

- AMS 解析和检查、创建并绑定进程、启动Activity。
- https://www.jianshu.com/p/a5532ecc8377

Retrofit原理

- 通过动态代理将接口直接转换成代理对象。动态代理和静态代理的区别，动态代理直接在虚拟机层面构建字节码对象

View自定义的流程，实现哪些方法 

怎么设计app。准备

- gradle依赖、签名文件、版本管理、代码规范、框架（组件化、MVP）、划分模块及实现

有1，3，7三个面值的金钱，现在要取n元。怎么取个数最少   n/7 + (n%7)/3 + (n%7)%3 

手写简单的算法

activity launchmode中single task和single instance的区别,以及各自的使用场景。  

- https://github.com/qmsggg/qmsggg_BlogCollect/issues/160

aidl的实现原理，通过aidl生成的.java文件中都有哪些成员，各自的作用。  

- https://blog.csdn.net/ljd2038/article/details/50705935

Looper，Handler，MessageQueue的原理，Looper如何保证每个线程只有一个。 

用数组实现栈，考虑如何在O（1）时间内获得栈的最小值。  

- 新设数组存储对应索引n的前n个数的最小值。

5.Object中定义了哪些方法。 equals一般如何实现，实现equals（）时候还需要实现哪些方法，为什么。

-  引用、判断getClass和值

synchronized和lock的区别。  

如何判断两个链表是否有交点。  

activity与fragment生命周期的关系。  

kotlin中var和val的区别，lateinit和lazy的区别。  

从activity a启动activity b，二者生命周期如何变化，为何是这种顺序。 

-  a pause -> b create、start、resume -> a stop
- https://blog.csdn.net/speverriver/article/details/78402912

手写二叉树层序遍历。  

手写在数组指定位置插入元素，后续元素向后顺延，考虑数组长度和数组容量间的关系，长度等于容量时需要重新分配更大的内存。 

如何实现通讯录搜索的功能（通过输入汉字查找姓名，通过号码查找姓名）。 

java内部类可以访问外部类的field，如何做到的。  

- 在调用内部类的构造器初始化内部类对象的时候， 编译器默认也传入外部类的引用
- https://blog.csdn.net/zhangjg_blog/article/details/20000769

设计一个应用的登录系统。  

微信通讯录增删改设计（本地和服务端）。  

java中的线程和操作系统的线程区别。  

Android 内存泄漏

- https://juejin.im/entry/589542ed2f301e0069054007

OOM

- http://www.runoob.com/w3cnote/android-oom.html

ANR

TCP三次握手和四次分手

- https://github.com/jawil/blog/issues/14

---

# Java









---

# 参考

* [面试时，问哪些问题能试出一个 Android 应用开发者真正的水平？](https://www.zhihu.com/question/19765032)
* [2018年年底Android悲催的面试之路](https://mp.weixin.qq.com/s/EpFEWQqW5D9VHzr_19ldAw)
* [2018年Android面试题整理](https://juejin.im/post/5a82a07df265da4e7071c78f)
* [2017下半年，一二线互联网公司Android面试题汇总](https://juejin.im/entry/59dd75cd51882578d5037626)
* [非常强势的Android面试题](https://segmentfault.com/a/1190000016117569)
* [2018 Android 面试心得](https://blog.csdn.net/H176Nhx7/article/details/79956377)

---

