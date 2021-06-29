---
title: Android编程权威指南第2版
date: 2018-12-20 06:57:46
tags:
- android
- 书
categories:
- Android开发
---

# 前言

优秀的Android开发入门书籍，对于有几年开发经验的开发人员来说，也值得一读。

比较基础，读得很快，说一下日常开发容易忽视的地方。

---

# Activity回退栈

获取应用任务栈（或回退栈）信息：

>不局限于单个应用，回退栈（Back Stack）作为一个整体共享给操作系统及设备使用。

我们不仅可以获取自己APP的任务栈信息，也可以通过获取系统服务如`ACTIVITY_SERVICE`等得到其他APP任务栈信息。

的确可以获取自己以及其他APP任务栈信息，但不同API等级的SDK获取的方式不一样，且获取其他APP任务栈信息越来越严格。

**API 20及以下**

* 通过`ActivityManager.getRunningTasks()`可以获取自己和其他APP的任务栈信息，包括栈基和栈顶Activity。

**API 21及以上**

* 通过`ActivityManager.getAppTasks()`可以获取自己APP的任务栈信息，包括栈基和栈顶Activity。
* `API = 21`时，通过`ActivityManager.getRunningAppProcesses()`可以获取相关信息。
* `API = 22`及以上时，通过`UsageStatsManager.queryEvents()`获取相关信息，但似乎还是只能获取自己APP信息，其他的获取不了。

---

# Fragment

尽可能在Fragment里实现UI用户界面，便于复用和扩展。

> 对于fragment，我们坚持AUF（AlwaysUseFragments）原则，即“总是使用fragment”。不值得为使用fragment还是activity而伤脑筋，相信我们，总是使用fragment！

Activity的Extra和Fragment的Argument Bundle参数的Key值，应该存在使用或消耗这些值的地方。而不应该是其他仅仅传参的地方。

> 一个比较好的做法是，将crimeID存储在属于CrimeFragment的某个地方，而不是保存在CrimeActivity的私有空间里。这样，无需依赖于CrimeActivity的intent内指定extra的存在，CrimeFragment就能获取自己所需的extra数据。属于fragment的“某个地方”实际就是它的argument bundle。

一般不要在Fragment里直接定义实例变量，需要的话则记得在`onSaveInstanceState(Bundle)`里保存。

> 创建实例变量的方式并不可靠。因为，在操作系统重建fragment时，设备配置发生改变时，用户暂时离开当前应用时，甚至操作系统按需回收内存时，任何实例变量都不复存在了。

---

# 样式和主题

覆盖主题属性，便于统一管理。要找到需要覆盖的属性，一层层从下往上定位、查询即可。

主题和样式应该在开发初期设置，避免后期改动太多。

覆盖页面背景和按钮样式示例如下：

```xml
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>

        <!--页面和对话框背景颜色-->
        <item name="android:colorBackground">@color/white_light</item>
     
      	<!--页面和背景颜色-->
      	<item name="android:colorBackground">@color/white_light</item>
      
        <item name="buttonStyle">@style/NormalButton</item>
    </style>


```



You can also define:

- textColor - The default text color of any given view
- textColorPrimary - The default text color for enabled buttons and Large Textviews
- textColorSecondary - The default text color for Medium and Small Textviews
- textColorTertiary - ?

(Source [TextColor vs TextColorPrimary vs TextColorSecondary](https://stackoverflow.com/questions/39070040/textcolor-vs-textcolorprimary-vs-textcolorsecondary))





---

# 参考

1. [Android 获取前台应用的方法总结](https://blog.csdn.net/AdobeSolo/article/details/77375714)
2. [关于 Android，用多个 activity，还是单 activity 配合 fragment？](https://www.zhihu.com/question/39662488)
3. [The Real Best Practices to Save/Restore Activity's and Fragment's state](https://inthecheesefactory.com/blog/fragment-state-saving-best-practices/en)
4. [保存/恢复Activity和Fragment状态的最佳实践(译)](https://zhuanlan.zhihu.com/p/22141193)

----