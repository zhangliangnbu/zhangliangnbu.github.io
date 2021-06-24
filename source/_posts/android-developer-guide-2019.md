---
layout: '[post]'
title: 文档-Android Developer Guide 2019
date: 2019-01-19 09:13:38
tags:
  - android
  - guide
  - 文档
categories:
  - Android开发
---


# 前言

温故知新，官方文档每年都应该读一遍。

---

# Animations & Transitions

**动画实现方式** 

Property Animation。最常用、最主要的动画，应用于UI可见性和运动、布局改变过渡、Activity切换过渡等。 

- API。主要位于`android.aniamtion`包，插补器在`android.view.animation`包。 基础API有：`ValueAnimator`、`ObjectAnimator`、`AnimatorListenerAdapter`、`AnimatorSet`。其他API有：过渡-`LayoutTransition`、状态-`StateListAnimator`、多动画和便捷-`ViewPropertyAnimator`。
- XML。位于`res/animator`，可使用`AnimatorInflater`加载。

Drawable Animation。图形动画，静态图片或矢量图动画 

- 逐帧图形动画。`AnimationDrawable`，资源为`res/drawable/animation-list`。 
- 矢量图动画。`AnimatedVectorDrawable`，资源为 `res/drawable/animated-vector`。 

View Animation。有局限，一般情况下，可用Property Animation替代。 

> 之前许多文档把动画分为属性动画、补间动画和图形动画三种，这里的补间动画就是View Animation，但感觉最新官方文档并没有这样划分。
>
> 对于最新的文档，论述最多的是Property Animation，其他两种动画说的很少，所以主要掌握Property Animation，其他两种动画了解即可。

**应用场景** 

- View显示和隐藏。前后连续的View的显示切换。crossfade、card flip？circular reveal animation 
- View移动。直线、曲线、速度控制等。使用API有：`ObjectAnimator` 、`Path` 、`Interpolar`、`FlingAnimation`等。 
- View缩放过渡。图片的缩放过渡，如从小图过渡到大图预览。利用属性有：`alpha`、`x`、`sacleX`等。 
- 布局变更过渡。布局更改后的多个View位置改变动画；利用系统动画直接设置`android:animateLayoutChanges="true”`即可，或自定义动画，利用API有：Scene、Transition、TransitionManager等。 
- Activity及其内部元素切换过渡。用到的API有`ActivityOptions.makeSceneTransitionAnimation()`等 

---

# Resources

资源组织。位于`res/`和`assets/`目录，独立于代码，利于复用。

可选性资源。提供可选性资源用于不同配置的设备。

* 资源文件夹名称格式*<resources_name>-<config_qualifier>*，其中*<config_qualifier>*(限定符)命名有自己的规则：可多个、有顺序、目录不能嵌套、大小写不敏感、限定类型不能相同。
* 资源匹配规则。根据[匹配规则-BestMatch](https://developer.android.com/guide/topics/resources/providing-resources#BestMatch)并依据[限定符表单](https://developer.android.com/guide/topics/resources/providing-resources#table2)从上到下依次匹配。
* 必须要设置默认资源，即无限定符的资源，避免APP找不到资源时崩溃。

代码里引用资源。

* 引用一般`res/`资源，`[<package_name>.]R.<resource_type>.<resource_name>`；
* 引用`res/raw`资源，` Resources.openRawResource()`；
* 引用`assets/`资源，`AssetManager.open("filename")`。

XML里引用资源。

* 引用文件资源：`@[<package_name>:]<resource_type>/<resource_name>`。
* 引用样式属性：`?[<package_name>:][<resource_type>/]<resource_name>`

本地化。使用可选资源、总是在`strings.xml`定义字符串、使用`<xliff:g>`保持部分文本不变。

Unicode和国际化。[ICU](http://userguide.icu-project.org/icufaq#TOC-How-is-the-ICU-licensed-)

使用`<aapt:attr>`内联复杂的XML资源。

字符串。数量字符串（Quantity Strings）`plurals`；样式设置-HTML、Spannables、Annotation。

---

# Permissions

Android安全架构的核心设计点是，默认情况下，任何应用都无权执行任何会对其他应用，操作系统或用户产生负面影响的操作。

概述

* 可选硬件功能。如无必须，设置`android:required="false"`。
* 权限强制。对服务的任何调用都可以强制执行任意细粒度的权限。四大组件、URI等。
* 自动权限适配。低`targetSdkVersion `的APP在高SDK版本平台运行时，可无视新增加的权限限制。
* 保护级别。*normal*, *signature*, and *dangerous*。另外，特殊权限如 `SYSTEM_ALERT_WINDOW` and `WRITE_SETTINGS`需要发起Intent请求。
* 权限组。仅影响*dangerous*权限用户交互体验，组内被手动授权后，其他自动被授权。
* 总是显示请求每个需要的权限。权限级别、组的划分可能会随着版本的更新而有所改变。
* ADB命令自动授权所有权限。`$ adb shell install -g MyApp.apk`

申请权限

* 声明。在Manifest中声明，可附加API等级、最大SDK版本信息等。
* 检查。`ContextCompat.checkSelfPermission()`。
* 申请。` ActivityCompat.requestPermissions()`。
* 处理。在`onRequestPermissionsResult()`处理。

权限使用原则

- 只添加必要权限。如能用其他方式实现则不要添加权限，如用Intent调用其他APP、使`onAudioFocusChange()`判断电话状态、使用`com.google.android.gms.iid`或`Advertising Identifier`获取身份凭证等。
- 注意依赖库的权限。添加依赖则会集成其所需权限。
- 对用户透明。让用户明白为何需要权限。
- 显示使用权限功能。让用户知道你在使用相关权限功能，不要偷偷使用。

自定义权限。(⊙o⊙)

---

# Testing

概述

* 测试驱动开发 Test-Driven Development。工作流：Failing Test -> Passing Test -> Refactor -> Failing Test -> ... 
* 测试层级。Small Tests（Unit Tests）70% -> Medium Tests 20% -> Large Tests 10%。

Small Tests或单元测试

- Local Unit Tests。最主要的单元测试。运行于本地JVM，源码位于*[module-name]/src/test/java/*；如果测试单元引用了Android层框架或三方依赖代码，可使用Robolectric或Mockito等来模拟。依赖库包括JUnit 4、Robolectric、Mockito。
- Instrumented Unit Tests。运行于模拟器或真机，源码位于*[module-name]/src/androidTest/java/*。依赖库包括AndroidX test runner和rules、Hamcrest、Espresso、UI Automator。

资源

- [googlesamples/**android-testing**](https://github.com/googlesamples/android-testing)
- [Test-Driven Development on Android with the Android Testing Support Library (Google I/O '17)](https://www.youtube.com/watch?v=pK7W5npkhho&start=111)
- [googlesamples/android-sunflower](https://github.com/googlesamples/android-sunflower)
- [Android Testing Codelab](https://codelabs.developers.google.com/codelabs/android-testing/index.html#0)。教程不错，可惜依赖库不是最新的，也没有用Robolectric。

实践。优化既有项目，测试工具类（单元测试或小型测试）、测试页面逻辑（简单UI测试或中型测试，可先优化MVP结构）、测试页面间逻辑（集成UI测试或大型测试）。

---

# Activities

概述。Activity而不是APP，是交互的基本单位。

生命周期。

- `onCreate()`。一个实例只执行一次，此处完成初始化工作。
- `onStart()`。可见，不能交互；执行很快。
- `onResume()`。可见，获取焦点能交互。
- `onPause()`。失去焦点不能交互；执行很快，不能做较重的操作，如操作数据库等。
- `onStop()`。可执行较重操作，以及释放资源。
- `onDestory()`。释放资源。

UI状态的保存和恢复。持久化信息系统或本地，并从系统或本地恢复到Activity。

回退栈和启动模式。哪个状态进栈的？

SingleTask。

- 官方说会开启新栈，但实际上是否进入新栈与"taskAffinity"有关，启动Activity与新建Activity的"taskAffinity"不同则进入新栈，否在在同一栈内。两种方式，一，AndroidManifest文件中设置，`android:launchMode="singleTask"`,如果不设置"taskAffinity"，则不会开启新栈，只会在当前栈新建实例；二，如果使用代码`flags = Intent.FLAG_ACTIVITY_NEW_TASK`，则必须设置不同的"taskAffinity"，否则不能开始"singleTask"模式。
- 发现了一个奇怪的情况，启动一个已经存在的"singleTask"的Activity，只是将其所在的任务栈带回前台，显示这个栈顶的Activity，而不是要启动的Activity。示例 Activity启动顺序： A->B->C，A和C设置为"singleTask"和不同的"taskAffinity"，执行C->A，则出现B，返回才看到A。

> 参考
>
> - ["singleTask"模式 切换到新的栈中](https://blog.csdn.net/u012889434/article/details/47150739)
> - [启动模式"singleTask"和FLAG_ACTIVITY_NEW_TASK具有不同的行为！](https://blog.csdn.net/lincyang/article/details/6802017)

进程和应用生命周期。

- 进程优先级取决于其中所运行的组件活动状态。
- 优先级从高到低：前台进程->可见进程->服务进程->缓存进程。

Parcelables和Bundles。每个进程的Binder事务缓冲区目前限制在1M以内，对于操作比如`savedInstanceState`建议缓冲数据不超过50K。

Fragments

- 设计哲学。模块化(避免从Fragment操作其他Fragment)、可复用、UI布局适配。
- 测试。验证交互一致性；API `FragmentScenario`。

App Links。

- Deep Link。可选择的APP；local intent-filter
- Android App Link。直接进入APP；local intent-filter & remote Digital Asset Links JSON file

---

# Android Architecture Components  

Lifecycle

- 概述。继承`LifecycleOwner`的UI控件如Activity或Fragment，能够注册观察者`LifecycleObserver`来监测其生命周期方法。如此则能够让生命周期敏感的组件在自己内部处理相应的生命周期方法，从而保持UI控件如Activity或Fragment的简洁性。
- 涉及类或接口。`LifecycleOwner`是`Lifecycle`的容器，使用`Lifecycle`添加观察者`LifecycleObserver`。
- 使用。创建生命周期敏感的组件继承`LifecycleObserver`；添加方法或构造方法获取`Lifecycle`，并使用`Lifecycle.addObserver()`绑定自身；在UI控件中实例化生命周期敏感的组件即可。

LiveData

- 概述。LiveData只通知处于active状态（`STARTD`和`RESUMED`）的` LifecycleOwner `观察者更新，并在`DESTROYED`与观察者解绑。通知更新有两种情况：一，数据改变；二，观察者状态从inactive切换到active。
- 优点。UI和数据保持同步（观察者模式）、无内存泄漏（自动解绑）、Stoppe状态Activities无Crash（inactive状态不通知更新）、无需手动处理生命周期事件、保持最新数据（后台切前台、配置改变如旋转）、资源分享（继承LiveData的系统服务）。
- 使用。创建`LiveData`、UI线程创`Observer`并定义`onChanged()`方法、绑定`Observer`（生命周期相关或无关）。
- 结合Room。Room提供返回LiveData对象的可观察查询，数据库更新时自动更新LiveData对象。
- 其他。集成扩展、变换、组合。

ViewModel

- 概述。用生命周期管理的方式，存储和管理UI相关数据。
- 生命周期。与所绑定的Lifecycle相关，在Created创建，在Activity Finished或Fragment Detached（并不再马上重新创建）之后销毁。
- 用处。Fragment间共享数据，使用相同的Key的ViewModel。在Lifecycle由于Configuration改变而重新创建期间存储数据。

Room

- Dao。自动更新`LiveData`。返回`LiveData`的方法会监测数据库（可查看生成的方法实现），当数据库更新数据时，对应的`LiveData`也会更新。如：

  ```kotlin
     @Query("SELECT * from word_table ORDER BY word ASC")
     LiveData<List<Word>> getAllWords();
  ```

参考

- [Github-googlesamples/android-architecture-components](https://github.com/googlesamples/android-architecture-components)
- [Codelabs-android-room-with-a-view](https://codelabs.developers.google.com/codelabs/android-room-with-a-view/#0)
- [Github-googlesamples/android-sunflower](https://github.com/googlesamples/android-sunflower)

---

# User interface & navigation

隐藏状态栏。

```kotlin
// Hide the status bar.
window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_FULLSCREEN
// Remember that you should never show the action bar if the
// status bar is hidden, so hide that too if necessary.
actionBar?.hide()
```

三种菜单。应用层级别的*Options Menu*、当前页面的行为响应或内容菜单、下拉列表*Popup Menue*。

---

# Images and graphics

优先使用WebP图片格式（API 17+）

---

# Background Tasks

几种解决方案。WorkManager、Foreground services、AlarmManager、DownloadManager。

异步编程解决方案。回调、Java 8的 CompletableFuture、Rx 编程、Kotlin协程。

- [Kotlin 协程](http://johnnyshieh.me/posts/kotlin-coroutine-introduction/)
- [Java线程的挂起和恢复](线程的挂起和恢复)

