---
title: Android源码与字节码不匹配问题
date: 2019-04-11 17:50:17
tags:
- android
- debug
categories:
- Android开发
---

# 问题

在调试应用时经常会碰到如下提示：

> source code does not match the bytecode （源码与字节码不匹配）

一般发生在调试的手机或模拟器运行的SDK版本与应用编译版本`compileSdkVersion`不一致导致的。比如你的手机Android版本是7.1.1即SDK版本为25，而你的IDE配置的`compileSdkVersion`不是25而是其他的，比如28，那么在调试的过程中，跟随断点进入Android源码时，就会出现以上提示。而此时一般是没法定位具体代码的。

这是其实Android Studio的Bug，详见：

- [Source Code does not match the bytecode](<https://issuetracker.google.com/issues/37123373>)
- [Debug android sources always default to compileSdkVersion and not the version on the running emulator/device](<https://issuetracker.google.com/issues/37058409>)

---

# 解决

以下两种方案都不是很好的方案，各有利弊

## 更改应用编译版本

更改Android Studio配置的`compileSdkVersion`，与调试的手机版本保持一致。

但有个问题，改变编译版本号后，编译可能会出问题。

## 更改源码文件路径

两种方法，其实原理是一样的，就是给Android Studio配置与调试的手机版本号保持一致源码。

**更改源码文件夹名**。更改`sdk/sources`下的文件夹名称，将与调试手机版本号对应的源码文件夹名称改为`compileSdkVersion`名称，那么在调试的时候，IDE是通过选择`compileSdkVersion`对应的文件夹名称作为源码的，这样就能与调试手机版本保持一致。但这样的话，开发过程中就不能看源码了，因为不能对应。

**更加源码配置路径**。找到AndroidStudio的`options/jdk.table.xml`，更改里面的`sourcePath`对应的版本号，然后重启AS即可。

> `jdk.table.xml`文件路径：
>
> - Mac或Linux中：`/Users/{username}/Library/Preferences/AndroidStudio3.3/options/`
> - Windows中：`C:\Users\用户.AndroidStudioXXX\config\option\`（由于本人使用Mac，这种情况没有验证）

---

# 参考

- [AndroidStudio debug source code和运行代码不匹配](<https://juejin.im/post/5c6aadc7f265da2dc538abd4>)
- [android studio无法关联源码](<https://blog.csdn.net/lonewolf521125/article/details/51331084>)
- [Android studio调试源码版本不对应问题](<https://blog.csdn.net/leilba/article/details/79198221>)