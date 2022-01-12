---
title: Java基础四：Java虚拟机基础
date: 2018-09-14 15:14:22
updated: 2022-01-11 11:52:48
tags:
- java
categories:
- Java
---

本文主要介绍字节码文件结构和加载、Java虚拟机结构等。

<!--more-->

# Class文件结构

Java虚拟机可以理解的代码就叫做`字节码`（即扩展名为 `.class` 的文件）。根据 Java 虚拟机规范，类文件由单个 ClassFile 结构组成：

```java
ClassFile {
    u4             magic; //Class 文件的标志, 确定文件是一个能被虚拟机接收的Class文件
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class 文件的字段属性
    field_info     fields[fields_count];//一个类会可以有个字段
    u2             methods_count;//Class 文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```

# Class文件的加载

Class文件需要加载到虚拟机中之后才能运行和使用。

## Class文件的加载过程

Java虚拟机加载 Class 类型的文件分为三步：加载->连接->初始化，其中连接过程又可分为三步：验证->准备->解析。

**加载阶段**

- 通过全类名获取定义此类的二进制字节流
- 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
- 在内存中生成一个代表该类的Class 对象，作为方法区这些数据的访问入口

该阶段会涉及到类加载器、双亲委派模型。

**连接阶段**

- 验证：验证文件格式、元数据、字节码、符号引用等。
- 准备：正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。
- 解析：虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行。

**初始化阶段**

类加载的最后一步，也是真正执行类中定义的 Java 程序代码(字节码)，初始化阶段是执行类构造器 `<clinit> ()`方法的过程。

## 类加载器

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader，其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1.  `BootstrapClassLoader`(启动类加载器) ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。
2.  `ExtensionClassLoader`(扩展类加载器) ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。
3.  `AppClassLoader`(应用程序类加载器) ：面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。

## 双亲委派模型

每一个类都有一个对应它的类加载器。系统中的 ClassLoder 在协同工作的时候会默认使用 **双亲委派模型** 。即在类加载的时候，系统会首先自低向上判断当前类是否被加载过，已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的`BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为`null`时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器。

<img src="/images/java-classloader-parent-delegate.png" alt="java-classloader-parent-delegate" style="zoom: 80%;" />

双亲委派模型的作用：保证了Java程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

# Java的内存区域介绍

Java虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。

<img src="/images/java-rt-aera.png" alt="java-rt-aera" style="zoom: 80%;" />

> 直接内存的分配不会受到 Java虚拟机的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制

**程序计数器**

线程私有。一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。

**虚拟机栈**

线程私有。由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息；命周期和线程相同。

Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后（return或抛出异常），都会有一个栈帧被弹出。

Java虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。

- `StackOverFlowError`： 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- `OutOfMemoryError`： 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出 OutOfMemoryError 错误。

**本地方法栈**

线程私有。虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。也会出现 StackOverFlowError 和 OutOfMemoryError 两种错误。

**堆**

线程共享。最大的内存区，存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。也是垃圾收集器管理的主要区域，因此也被称作GC 堆（Garbage Collected Heap）。容易出现的是 `OutOfMemoryError` 错误。

在 Java 堆中开辟了一块区域存放运行时常量池，用于存放编译期生成的各种字面量和符号引用。（1.8+）

**方法区或元空间(1.8+)**

存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

# 对象的创建和访问

虚拟机是如何创建和访问对象的？

**对象的创建**

- 类加载检查。虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
- 分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。分配方式有 “指针碰撞” 和 “空闲列表” 两种，选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。
- 初始化零值。存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。
- 设置对象头。初始化零值完成之后，虚拟机要对对象进行必要的设置，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。
- 执行 `init` 方法。 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `init` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

**对象的内存布局**

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：对象头、实例数据和对齐填充。

- 对象头。包括两部分信息，第一部分用于存储对象自身的运行时数据（哈希码、GC 分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。
- 实例数据。对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容。
- 对齐填充。不是必然存在的，也没有什么特别的含义，仅仅起占位作用。 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

**对象的访问定位**

 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有使用句柄和直接指针两种：

- 句柄： Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息。
- 直接指针：  如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。

这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

# 补充

**String 类和常量池**

String 对象的两种创建方式：

```java
String str1 = "abcd";//先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str2 = new String("abcd");//堆中创建一个新的对象
String str3 = new String("abcd");//堆中创建一个新的对象
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

这两种不同的创建方法是有差别的。

- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。

记住一点：**只要使用 new 方法，便需要创建新的对象。**

# 参考

- <https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html>
- <https://coolshell.cn/articles/9229.html>
- <https://blog.csdn.net/luanlouis/article/details/39960815>
- <https://blog.csdn.net/xyang81/article/details/7292380>
- <https://juejin.im/post/5c04892351882516e70dcc9b>
- <http://gityuan.com/2016/01/24/java-classloader/>
- <https://docs.oracle.com/javase/specs/index.html>
- <http://www.pointsoftware.ch/en/under-the-hood-runtime-data-areas-javas-memory-model/>
- <https://dzone.com/articles/jvm-permgen-%E2%80%93-where-art-thou>
- <https://stackoverflow.com/questions/9095748/method-area-and-permgen>
- <https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html>
- 《深入理解 Java 虚拟机：JVM 高级特性与最佳实践（第二版》
- 《实战 java 虚拟机》

