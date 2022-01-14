---
title: Java基础五：Java虚拟机垃圾回收机制
date: 2018-09-15 15:14:22
updated: 2022-01-11 15:52:48
tags:
- java
categories:
- Java
---

# 简介

垃圾回收（Garbage Collection）是Java虚拟机垃圾回收器提供的一种用于在空闲时间不定时回收无任何对象引用的对象所占据内存空间的一种机制。

> 注意：垃圾回收器回收的是无用对象占据的内存空间。

<!--more-->

# 判断对象可被回收的方法

**引用计数算法**。给对象中添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不可能再被使用的。但是Java语言没有选用引用计数法来管理内存，因为引用计数法不能很好的解决循环引用的问题。

**根搜索算法**。在主流的商用语言中，都是使用根搜索算法来判定对象是否存活的。GC Root Tracing 算法思路就是通过一系列的名为"GC  Roots"的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连，即从GC Roots到这个对象不可达，则证明此对象是不可用的。

可作为GC Roots的对象：

- 方法区中的常量引用的对象
- 方法区中的类静态属性引用的对象
- 虚拟机栈（栈帧中的本地变量表）中的引用的对象
- 本地方法栈中JNI（Native方法）的引用对象

# GC流程

HotSpot JVM 将堆分成了 二个大区**Young** 和**Old**，而Young 区又分为 Eden、Servivor1、Servivor2。

所有新new出来的对象都会最先出现在Eden区中，当Eden区内存满了之后，就会触发一次Minor GC，这种收集通常比较快，因为新生代的大部分对象都是需要回收的，那些暂时无法回收的就会被移动到老年代。

> minor garbage collections都是**Stop the World**事件

升到老年代的对象大于老年代剩余空间时full gc，或者小于时被HandlePromotionFailure参数强制full gc。

# 参考

- [问答](https://www.iteye.com/topic/894148)
- https://www.zhihu.com/question/35164211/answer/68265045
- https://www.jianshu.com/p/5261a62e4d29
- https://blog.csdn.net/justloveyou_/article/details/71216049
- [https://liuchi.coding.me/2017/08/05/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90Java%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6/](
