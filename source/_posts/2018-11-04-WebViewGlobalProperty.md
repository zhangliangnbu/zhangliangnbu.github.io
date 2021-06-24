---
title: WebView全局属性设置问题
date: 2018-11-04 11:58:42
tags:
- Android
- SpannableString
categories:
- Android开发
---

# 问题描述

两个Activity A和B里都使用了WebView控件。

1. 先进入A，webview可以正常加载；然后进入B，WebView也可以正常加载。
2. 先进入B，webview可以正常加载；然后进入A，WebView不能正常加载！

-- -- --

<br>

# 查找原因

刚开始遇到这个问题，一脸懵。A Activity是我写的，B Activity是我的小伙伴写的，经过对比两个WebView的设置后，发现B Activity的生命周期里多了两行代码`wbv_video.resumeTimers()`和`wbv_video.pauseTimers()`，如下所示：

```kotlin
   override fun onResume() {
        super.onResume()
        wbv_video.resumeTimers()
    }

    override fun onPause() {
        super.onPause()
        wbv_video.pauseTimers()
    }
```
查看方法文档，如下：
>`wbv_video.resumeTimers()` Resumes all layout, parsing, and JavaScript timers for all WebViews. This will resume dispatching all timers.(resume所有WebView的布局、解析、JS的任务计时器)
>`wbv_video.pauseTimers()`Pauses all layout, parsing, and JavaScript timers for all WebViews. This is a global requests, not restricted to just this WebView. This can be useful if the application has been paused.(暂停所有WebView的布局、解析、JS的任务计时器。**这是全局设置，不仅仅针对当前WebView**。在应用暂停后使用这个方法比较有用，可以减少CPU的使用，省电)

坑，就是这里设置的问题，`wbv_video.resumeTimers()`和`wbv_video.pauseTimers()`影响的是应用里所有的WebView，不只是当前WebView。
回到前面遇到的问题，当进入B Activity时，然后离开时，会调用`wbv_video.pauseTimers()`，此时会停止所有的WebView的布局和解析，所以再进入A Activity时，WebView当然就不能正常加载了。

-- -- --

<br>

# 解决问题

在B Activity里删掉`wbv_video.resumeTimers()`和`wbv_video.pauseTimers()`这两行代码即可