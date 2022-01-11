---
title: View的clickable和enable属性
date: 2018-08-23
updated: 2022-01-11
tags:
- android
categories:
- Android开发
---

# 介绍

控件的enable属性影响有：聚焦、事件交互。

控件的clickable属性影响事件交互。

<!--more-->

# 在事件分发中的区别

- View 为“DISABLED”时，不执行`OnTouchListener.onTouch()`方法；
- View 为“DISABLED”时，`View.onTouchEvent()`被调用，其返回值为clickable 的值。clickable 为 true时，就说明即使View为“DISABLED”状态也可以消费点击事件。
- 当View为“ENABLED”，且设置了触摸代理时，onTouchEvent()的返回值依据代理对象返回的结果。

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean result = false;
    if (onFilterTouchEventForSecurity(event)) {
        // disable时不能处理滚动拖拽
        if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
            result = true;
        }
        ListenerInfo li = mListenerInfo;
        // disable时 不回调OnTouchListener.onTouch方法
        if (li != null && li.mOnTouchListener != null
            && (mViewFlags & ENABLED_MASK) == ENABLED
            && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }
        // 如果处理了页面滚动拖拽或回调了OnTouchListener.onTouch方法，就不调用onTouchEvent
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }
    return result;
}

public boolean onTouchEvent(MotionEvent event) {
    final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                               || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
        || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
    // 如果disable，会直接返回而不处理后续程序，是否消费事件看 clickable
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
        return clickable;
    }
    // enable = true, clickable = false时，如果设置了触摸代理则依然可以返回true!
    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
}
```

# 其他-TouchDelegate

一个View可以通过设置代理对象而让其他View执行、消费事件。

# 参考

- [Android TouchDelegate详解及优化](https://www.jianshu.com/p/cb5181418c7a)
