---
title: JNI开发概览
date: 2021-12-09 17:14:28
tags: 
- java
- android
- android系统开发
categories:
- Android开发
---

# 说明

JNI，即Java本地接口（Java Native Interface），允许Java代码与C/C++等本地语言编写的代码进行交互操作。

JNI提供的接口声明位于：`<JDK_HOME>/include/jni.h`中

> 注意：JNI是Java的特性而非Android独有特性，只因为Android使用了Java语言，因此可以利用JNI的功能。

## 基本原理

**Java虚拟机**利用**函数原型**将Java声明的本地方法与运行库中的C函数通过函数映射表对应起来，这样Java代码就能与C/C++代码交互。

> **函数原型**（英语：**Function prototype**）或**函数接口**（英语：**Function interface**）是用于指定函数的名称和[类型签名](https://zh.wikipedia.org/wiki/类型签名)（[元数](https://zh.wikipedia.org/wiki/元数)，参数的[数据类型](https://zh.wikipedia.org/wiki/資料類型)和返回值类型）的一种省略了函数体的[函数](https://zh.wikipedia.org/wiki/子程序)[声明](https://zh.wikipedia.org/w/index.php?title=声明&action=edit&redlink=1)。

>思考一：Java虚拟机作为桥梁。Java代码和C/C++本地代码运行环境不一样，因此不能直接交互，而Java代码的容器Java虚拟机是C/C++代码编写的，因此可以使用Java虚拟机作为桥梁将Java代码和C/C++本地代码联系起来。
>
>思考二：为什么用函数原型？Java方法和C/C++函数是不一样的，但它们可以在函数原型上变为一致，都有名称和签名，因此可以使用函数原型建立映射关系。

## Java代码与JNI本地函数的交互

使用JNI一般遵循以下步骤：

1. 声明本地方法。在Java类中使用`native`关键字声明本地方法并加载本地函数库。
2. 建立映射关系。使用`javah`命令，生成包含JNI本地函数原型的头文件，建立Java方法与C/C++本地函数间的映射。另外，在C/C++本地函数里使用`RegisterNatives()`也可直接创建映射关系，且速度较快。
3. 实现C/C++本地函数。根据上面提供的头文件实现本地函数。
4. 编译本地函数，生成C/C++共享库
5. Java代码通过JNI调用本地函数。本地函数可以使用虚拟机传递过来的`JNIEnv *`指针调用JNI函数，进而调用Java代码。

**不遵循以上步骤的场景**：如果仅仅只是编写C/C++应用，然后复用Java提供的库来实现某些功能，不涉及编写Java类调用C/C++函数的代码，则不需要遵循上面的步骤，而是直接在C函数里创建虚拟机并获取对应的`JNIEnv *`指针，然后通过JNI函数调用Java代码即可。

![aosp10-jni-guide.drawio](/images/aosp10-jni-guide.drawio.svg)

> 思考：能否基于JNI开发主要使用C/C++编写的本地应用程序？不妥，JNI函数调用的开销应该比较大，影响性能。

# JNI使用示例

## Java代码调用C/C++本地函数

1. 在Java类中声明本地方法并加载本地函数库

   ```java
   // HelloJNI.java
   class HelloJNI {
   	native void printString(String str); // 声明本地方法
   	static { System.loadLibrary("hellojni"); } // 加载本地函数库
   	public static void main(String args[]) {
   		HelloJNI myJNI = new HelloJNI();
   		myJNI.printString("Hello World from native");
   	}
   }
   ```

2. 使用`javah`命令，生成包含JNI本地函数原型的头文件，建立Java方法与C/C++本地函数间的映射

   ```bash
   $ javac HelloJNI.java
   $ javah HelloJNI
   $ ls
   HelloJNI.class  HelloJNI.h  HelloJNI.java
   $ cat HelloJNI.h
   ...
   /*
    * Class:     HelloJNI # 对应的Java类名
    * Method:    printString # 方法名
    * Signature: (Ljava/lang/String;)V  # 方法签名，括号内为入参，括号外为返回值
    */
   JNIEXPORT void JNICALL Java_HelloJNI_printString
     (JNIEnv *, jobject, jstring); # 本地函数原型
   ...
   ```

   `HelloJNI.h`头文件中包含基于Java方法生成的本地函数原型以及注释，本地函数原型说明如下：

   - JNIEXPORT、JNICALL：关键字，JNI需要此关键字才能正常调用函数。
   - Java_HelloJNI_printString：本地函数名，格式`Java_类名_方法名`。
   - JNIEnv *, jobject：默认参数，JNIEnv *指向JNI提供的基本函数集，可用来调用相关函数；jobject表示调用本地方法的Java本地对象，如果调用静态方法则为jclass，表示Java本地类。

3. 实现JNI本地函数：编写`hellojni.c`文件

   ```c
   #include "HelloJNI.h"
   #include <stdio.h>
   
   JNIEXPORT void JNICALL Java_HelloJNI_printString(JNIEnv *env, jobject obj, jstring string) {
       // jni.h文件里声明了GetStringUTFChars()函数
       const char *str = (*env)->GetStringUTFChars(env, string, 0);
       printf("%s\n", str);
       return;
   }
   ```

4. 生成C共享库。以Windows平台为例，生成`hellojni.dll`文件

   在`Visual C++ 2015 x86 x64 Cross Build Tools Command Prompt`编译代码

   ```bash
   # C:\Program Files\Java\jdk1.8.0_251 为JDK Home目录
   $ cl -I"C:\Program Files\Java\jdk1.8.0_251\include" -I"C:\Program Files\Java\jdk1.8.0_251\include\win32" -LD hellojni.c -Fehellojni.dll
   ```

5. Java代码通过JNI调用本地函数

   ```bash
   $ java HelloJNI
   Hello World from native!
   ```

## C/C++本地函数调用Java代码

C/C++函数调用Java代码的用法有：获取Java类的class，创建Java类的对象实例，调用类的静态成员变量和静态方法，调用对象的成员变量和方法。相关JNI接口都声明在`<JDK_HOME>/include/jni.h`中。

相关部分接口(C++)，通过`JNIEnv *`调用如下：

```cpp
// 查询并加载类
jclass FindClass(const char *name)
// 获取类的静态方法
jmethodID GetStaticMethodID(jclass clazz, const char *name, const char *sig)
// 调用类的静态方法 XXX表示Object、Boolean等方法返回值类型
jXXX CallStaticXXXMethod(jclass clazz, jmethodID methodID, ...)

// 查找方法. 构造方法： name="<init>", sig为签名如"(I)V"
jmethodID GetMethodID(jclass clazz, const char *name, const char *sig)
// 创建Java对象实例
jobject NewObject(jclass clazz, jmethodID methodID, ...)
// 调用普通对象方法
jxxx CallXxxMethod(jobject obj, jmethodID methodID, ...)
```

>上述接口，在C里需要在入参里添加env(JNIEnv *)。

**C函数主动调用Java代码注意事项**

调用JNI方法的前提是持有`JNIEnv *`指针，该指针一般通过Java调用本地方法传给C/C++函数。但如果C/C++函数主动调用Java的代码时没有持有该指针，或者此时Java虚拟机没有启动，则C/C++如何调用Java代码呢？答案是在C函数里主动生成虚拟机，获取与该虚拟机对应的`JNIEnv *`指针，然后就可以调用Java代码了。

```c
// 生成虚拟机的接口
JNI_CreateJavaVM(JavaVM **pvm, void **penv, void *args);
```

注意：通过主动生成虚拟机获取`JNIEnv *`指针，然后调用Java代码时，无需创建Java方法与本地C/C++函数间的映射表这一步骤。

# 使用Android NDK开发

在开发Android里的JNI相关的功能，可使用Android Studio配套的NDK工具包，一键编译，方便快速开发。

NDK使用的官方介绍：[Android NDK](https://developer.android.google.cn/ndk)

NDK使用示例见官方CodeLab: [Create Hello-CMake with Android Studio](https://developer.android.google.cn/codelabs/android-studio-cmake#0)



# 参考

- 金泰廷等著《Android框架解密》，人民邮电出版社，2012。该书分析的代码较老，但框架机制基本不变，非常好的书籍。
- 源码查看网站：http://aospxref.com/。基于OpenGrok的源码查看服务网站，速度很快，主要用来查看C/C++代码，Java代码可下载到本地查看。
- [Create Hello-CMake with Android Studio](https://developer.android.google.cn/codelabs/android-studio-cmake#0):NDK开发示例

