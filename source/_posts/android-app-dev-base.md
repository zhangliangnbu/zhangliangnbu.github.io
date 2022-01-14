---
title: Android应用开发基础一：应用和Application
date: 2018-08-10 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

总结Android应用开发基础知识中的应用和Application类。

<!-- more -->

# 应用

Android 应用的生命周期由系统控制，包括三个阶段：启动、运行和结束。每个阶段会触发Application 对象中对应的回调方法。

## 应用的启动

通过Launcher启动App的流程：

- Launcher进程请求SystemServer进程创建Activity。
- SystemServer进程解析Intent等，并检查目标APP进程是否存在，不存在则去创建进程。
- SystemServer进程通过Socket方式请求Zygote进程创建APP进程，返回进程号等信息。
- App进程入口为`ActivityThread.main()`方法，执行此方法，初始化进程并请求SystemServer绑定App进程。绑定进程的工作包括，一，通知App进程去启动Application；二，通知APP进程去启动Activity。

<img src="/images/android-app-start-simple-flow.png" width="70%" height="70%">

总结：当用户点击应用图标的时候，系统会去检测进程是否存在，如果不存在，则通过Zygote创建进程，创建完进程后，则需要将进程与App进行绑定，将App的资源加载到内存中，当加载完毕，各种条件准备就绪，接下来就是启动Application和Activity了，如此，App的界面就展示出来了。

## 应用的销毁

每个 Android 应用都在各自的 Linux 进程中运行，应用进程的终止由系统通过综合因素来决定。因素包括：当前运行的应用、应用对用户的重要程度、系统的可用内存等。

当系统内存过低时，会根据不同进程的优先级进行内存回收。系统根据进程的状态，将进程分为四个等级：

**前台进程（Foreground Process）**

前台进程是优先级最高的进程，用户当前操作交互的必须是前台进程。当一个进程包含以下条件的时候，可以认为是前台进程。

1. 用户正在交互的，处在屏幕最上层的`Activity`（`onResume`被调用过后）。
2. 包含一个正在运行的`BroadcastReceiver`（`onReceive`正在执行）。
3. 包含一个正在运行它的回调（`onCreate`、`onStart`、`onDestroy`）的`Service`。

在一般状况下，杀死前台进程需要用户交互。当被系统杀死的时候，说明此时系统连该进程所需要的内存都无法满足，是最后才被杀死的。

**可见进程（Visible Process）**

可见进程是当前用户关心但是杀掉它会显著的影响用户体验的进程。当包含如下情况时，该进程可以被当做可见进程。

1. 包含一个对用户可见但不在前台（`onPause`被调用过后）的`Activity`。
2. 包含一个通过`startForeground`启动，正在运行的作为前台`Service`的进程。
3. 包含一些用户可感知的特定需求的`Service`，例如动态壁纸、输入服务等。

可见进程一般不会被销毁，除非是为了保证所有前台进程的运行，而不得不杀死可见进程。

**服务进程（Service Process）**

服务进程是包含用`startService`所创建`Service`的进程。这类进程对用户不是直接可见，但是用户会关心的，例如后台上传服务等等。所以系统会尽量维持它们的运行，除非系统内存不足以维持前台进程和可见进程的运行需要。

当`Service`运行了很长的时间，例如超过30分钟，系统就会对其降级，以使该进程会被更容易的回收。

**缓存进程（Cached Process）**

缓存进程是当前不被需要的进程，因此系统可以在任何需要内存的时候，释放掉它们的内存。

这类进程通常包含一个或多个当前不可见（`onStop`被调用）的`Activity`实例。当系统杀掉这类进程的时候，不会影响用户的体验。

**LMK进程管理**

关于进程的管理，首先需要知道Low Memory Killer（LMK）这个概念。它是基于Linux的Out of Memory Mechanism（OOM机制）改进而来的。LMK是一个内核层组件，是一个进程杀手。它的主要作用是在系统低内存状态时，释放掉那些不太重要进程的内存，让系统更加流畅。

每一个进程根据其重要性，都包含一个oom_adj值，AMS会去动态的更新oom_adj这个值。当系统处在低内存状态时，LMK会根据oom_adj这个值，去杀死相关的进程。oom_adj值得范围是-17~15，一个进程的oom_adj值越高，它被杀死的概率就越大。

整个过程就是AMS更新oom_adj值，LMK去挑选并杀死进程。

# Application

维护全局的应用状态的基类

>  Base class for maintaining global application state.

```java
public class LifeCycleApplication extends Application {
    @Override
    public void onCreate() {
        // 当应用启动的时候调用，除了content providers之外，早于其他任何组件的创建。
        super.onCreate();
    }

    @Override
    public void onTrimMemory(int level) {
        // 程序在内存清理的时候执行（回收内存），会经常调用
        // HOME键退出应用程序、长按MENU键，打开Recent TASK都会执行
        super.onTrimMemory(level);
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }

    @Override
    public void onLowMemory() {
        // 整个系统低内存的时候执行，需要运行的应用减少内存使用。
        super.onLowMemory();
    }

    @Override
    public void onTerminate() {
        // 程序终止的时候执行
        // 由于系统结束进程采用的是kill的方法，因此不会在正式环境的真实设备上产生相关的回调。
        super.onTerminate();
    }
}
```

# 参考

- [官方指南：进程和应用生命周期](https://developer.android.com/guide/components/activities/process-lifecycle?hl=zh-cn)
- [喜闻乐见之Android应用的生命周期](https://juejin.cn/post/6844903603128238093)
- [Application的应用和生命周期](https://blog.csdn.net/maican666/article/details/77257878)
- [Android Application Launch explained: from Zygote to your Activity.onCreate()](https://medium.com/android-news/android-application-launch-explained-from-zygote-to-your-activity-oncreate-8a8f036864b#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjE3MTllYjk1N2Y2OTU2YjU4MThjMTk2OGZmMTZkZmY3NzRlNzA4ZGUiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2MjI1MzIxMjAsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjExMDMwODIyNTg1MDcyOTE0MjQ3OSIsImVtYWlsIjoiemhhbmdsaWFuZ25idUBnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiYXpwIjoiMjE2Mjk2MDM1ODM0LWsxazZxZTA2MHMydHAyYTJqYW00bGpkY21zMDBzdHRnLmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwibmFtZSI6ImxpYW5nIHpoYW5nIiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hLS9BT2gxNEdpdjdkeDNEcGN2S2lobmZhX1ZyUHMybDhNZDJQa043N2pJY0JhST1zOTYtYyIsImdpdmVuX25hbWUiOiJsaWFuZyIsImZhbWlseV9uYW1lIjoiemhhbmciLCJpYXQiOjE2MjI1MzI0MjAsImV4cCI6MTYyMjUzNjAyMCwianRpIjoiOTU2NmM4ZjdiMTEzYzUyZmZiYjVhY2EwNDBhMzFiNjE2YjgxNjVmMCJ9.by-YCUxlq18QvHAfr1O4pKt9PdYAyCa3ia2jFt37t4EZ9gt6HRIfYrh4ci21WWUlNOOQ7MWXKmSALn5hUER6NajQTaxUjDeLNeEWzbxNhK9TPEp_8b42lBDKvjZgaEQSpygcsGflplpqKbFwi5UGXaBA5DPzH4gzFaf1Ujvw9-S_AEifMqsV_YQajTCCXE9h_qXLvYQbbx2lS9hL5V3H8bjeZYNLa3L3Z1njXE5af6orWaud88CgbAJycPGxEBkqFLRkDJXZAfBDBvg0fgvSTqj12hiqsyUToxZ7ze0Jqvd6SSawI90uaLmleQj0jNGNObmk5oWwvnbO1nI2TmfhKw)
- [Android 9.0 source APP startup process](https://programmer.group/android-9.0-source-app-startup-process.html)
- [应用启动时间](https://developer.android.com/topic/performance/vitals/launch-time#cold)

