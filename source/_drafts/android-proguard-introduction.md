---
title: android-proguard-introduction
date: 2019-07-01 09:29:27
tags:
- Android
- 混淆
categories:
- Android开发
---



学习纲要

- 学什么？熟悉历史、标准、规则。 
- 学习程度？能都定位混淆问题、能够速查API 



ProGuard是一个代码压缩、优化、混淆工具。

使用Android Studio自带的apk分析工具，查看代码是否混淆。

常用proguard文件配置keep语句

```properties
# keep类名
-keep public class com.liang.test.MyTest
# keep包中类名
-keep public class com.liang.test.*
# keep包和子包中类名
-keep public class com.liang.test.**
# keep包和子包中类名及成员
-keep public class com.liang.test.**{*;}
```

代码压缩和资源压缩。

多模块混淆问题。

```groovy
android {
    defaultConfig {
        consumerProguardFiles 'lib-proguard-rules.txt'
    }
    ...
}
```

---



#参考

- [Android官方-压缩代码和资源](https://developer.android.com/studio/build/shrink-code.html)
- [ProGuard官方-使用](https://www.guardsquare.com/en/products/proguard/manual/usage)
- [Bugly-Android 混淆那些事儿](https://zhuanlan.zhihu.com/p/27582991)
- [解决Android混淆常见问题](https://yuweiguocn.github.io/troubleshooting-proguard-issues-on-android/)