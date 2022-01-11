---
title: Android应用开发基础五：事件和绘制
date: 2018-08-14 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

本章介绍触摸事件分发机制和View绘制流程

<!-- more -->

# 事件分发机制

点击事件中相关类的传递顺序为：Activity -> Window -> View。

对于ViewGroup，发生点击事件后的流程为：

1. `ViewGroup.dispatchTouchEvent()` 。父布局分发事件
2. 事件拦截。若拦截则进入View处理事件的流程，不拦截则分发事件给子视图。事件拦截判断由`ViewGroup.onInterceptTouchEvent()`和`FLAG_DISALLOW_INTERCEPT`共同决定，其中`FLAG_DISALLOW_INTERCEPT`优先级高。如果`FLAG_DISALLOW_INTERCEPT`为不拦截，则最终结果为不拦截，否则才会根据`ViewGroup.onInterceptTouchEvent()`的结果判断。`FLAG_DISALLOW_INTERCEPT`可以通过子视图调用父视图的`ViewParent.requestDisallowInterceptTouchEvent()`方法进行设置。
3. 不拦截且ViewGroup有子视图处理事件。子视图调用`View.dispatchTouchEvent()`继续处理事件分发流程，直到事件被拦截或视图没有子视图处理事件为止。如果`View.dispatchTouchEvent()`返回False，表示子视图不消费事件，则事件会返回给父视图处理，如果父视图不处理，则事件会继续往根视图方向传递。
4. 拦截或ViewGroup没有子视图处理事件。调用`super.dispatchTouchEvent()`，之后先调用`OnTouchListener.onTouch()`，如果没有设置OnTouchListener或返回为False，则采取调用`View.onTouchEvent()`方法。在`View.onTouchEvent()`方法内，Touch事件会被解析成Click之类的事件。

一旦一个元素拦截了某事件，那么一个事件序列里面后续的Move、Down事件都会交给它处理。并且它的`onInterceptTouchEvent()`不会再调用。

`View.onTouchEvent()`默认都会消耗事件，除非它的clickable和longClickable都是false(不可点击)，但是enable属性不会影响。

**ViewGroup事件分发流程：**

<img src="\images\android-view-group-touch.png" alt="android-view-group-touch" style="zoom: 80%;" />

**View事件流程：**

![示意图](/images/android-view-touch.png)

# View 绘制机制

当View需要绘制的时候，首先计算脏区，然后遍历到顶层`ViewParent`即`ViewRootImpl`，调用流程的入口方法`ViewRootImpl.performTraversals()`，之后根据条件执行 measure、layout 和 draw三大操作。每个操作都是顺着视图树自父视图到子视图逐层进行的。

## Measure

该流程的结果是计算出视图树中每个视图的宽高。每一个 ViewGroup 负责测绘它所有的子视图，而最底层的 View 会负责测绘自身。

**一般流程：**measure 过程由`measure(int, int)`方法发起，从上到下有序的测量 View，在 measure 过程的最后，每个视图存储了自己的尺寸大小和测量规格MeasureSpec，子视图会根据父视图的MeasureSpec测量自身宽高。

**二次测量：**measure 过程会为一个 View 及所有子节点的 mMeasuredWidth 和 mMeasuredHeight 变量赋值，该值可以通过 `getMeasuredWidth()`和`getMeasuredHeight()`方法获得。而且这两个值必须在父视图约束范围之内，这样才可以保证所有的父视图都接收所有子视图的测量。如果子视图对于 Measure 得到的大小不满意的时候，父视图会介入并设置测量规则进行第二次 measure。比如，父视图可以先根据未给定的 dimension 去测量每一个子视图，如果最终子视图的未约束尺寸太大或者太小的时候，父视图就会使用一个确切的大小再次对子视图进行 measure。

**测量入参：**ViewGroup.LayoutParams，视图自身的布局参数，以及MeasureSpec，父视图对子视图的测量限制。

MeasureSpec有三种模式：

1. UNSPECIFIED。父视图不对子视图有任何约束，它可以达到所期望的任意尺寸。比如 ListView、ScrollView，一般自定义 View 中用不到
2. EXACTLY。父视图为子视图指定一个确切的尺寸，而且无论子视图期望多大，它都必须在该指定大小的边界内，对应的属性为match_parent 或具体值，比如 100dp，父控件可以通过`MeasureSpec.getSize(measureSpec)`直接得到子控件的尺寸。
3. AT_MOST。父视图为子视图指定一个最大尺寸。子视图必须确保它自己所有子视图可以适应在该尺寸范围内，对应的属性为 wrap_content。这种模式下，父控件无法确定子 View 的尺寸，只能由子控件自己根据需求去计算自己的尺寸，这种模式就是我们自定义视图需要实现测量逻辑的情况。

**核心方法：**

- `View.measure(int widthMeasureSpec, int heightMeasureSpec)` 。不可被复写，但 measure 调用链最终会回调 View/ViewGroup 对象的 `onMeasure()`方法，因此自定义视图时，只需要复写 `onMeasure()` 方法即可。
- `View.onMeasure(int widthMeasureSpec, int heightMeasureSpec) `。该方法就是我们自定义视图中实现测量逻辑的方法，该方法的参数是父视图对子视图的 width 和 height 的测量要求。在我们自身的自定义视图中，要做的就是根据该 widthMeasureSpec 和 heightMeasureSpec 计算视图的 width 和 height，不同的模式处理方式不同。
- `View.setMeasuredDimension()`。 测量阶段终极方法，在 `onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 方法中调用，将计算得到的尺寸，传递给该方法，测量阶段即结束。该方法也是必须要调用的方法，否则会报异常。在我们在自定义视图的时候，不需要关心系统复杂的 Measure 过程的，只需调用`setMeasuredDimension()`设置根据 MeasureSpec 计算得到的尺寸即可。

## Layout

 layout 过程由`layout(int, int, int, int)`方法发起，也是自上而下进行遍历。在该过程中，每个父视图会根据 measure 过程得到的尺寸来摆放自己的子视图。

首先要明确的是，子视图的具体位置都是相对于父视图而言的。View 的 onLayout 方法为空实现，而 ViewGroup 的 onLayout 为 abstract 的，因此，如果自定义的 View 要继承 ViewGroup 时，必须实现 onLayout 函数。

在 layout 过程中，子视图会调用getMeasuredWidth()和getMeasuredHeight()方法获取到 measure 过程得到的 mMeasuredWidth 和 mMeasuredHeight，作为自己的 width 和 height。然后调用每一个子视图的layout(l, t, r, b)函数，来确定每个子视图在父视图中的位置。

**LinearLayout 的 onLayout 源码分析**

```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}

/**
  * 遍历所有的子 View，为其设置相对父视图的坐标
  */
void layoutVertical(int left, int top, int right, int bottom) {
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        if (child == null) {
            childTop += measureNullChild(i);
        } else if (child.getVisibility() != GONE) 
            final int childWidth = child.getMeasuredWidth();//measure 过程确定的 Width
            final int childHeight = child.getMeasuredHeight();//measure 过程确定的 height
            setChildFrame(child, childLeft, childTop + getLocationOffset(child),childWidth, childHeight);
        }
    }
}

private void setChildFrame(View child, int left, int top, int width, int height) {        
    child.layout(left, top, left + width, top + height);
}   

// View.java
public void layout(int l, int t, int r, int b) {
    setFrame(l, t, r, b)
}

```



## Draw

Draw流程也是自顶（根视图）向下（子视图）进行Draw操作的。执行的入口为`View.draw()`，该方法实现步骤为先调用`View.onDraw()`绘制自身，然后在调用`View.dispatchDraw()`绘制子视图。

`View.onDraw（Canvas）`默认是空实现，自定义绘制过程需要复写的方法，绘制自身的内容。

```java
    /**
     * Manually render this view (and all of its children) to the given Canvas.
     * The view must have already done a full layout before this function is
     * called.  When implementing a view, implement
     * {@link #onDraw(android.graphics.Canvas)} instead of overriding this method.
     * If you do need to override this method, call the superclass version.
     *
     * @param canvas The Canvas to which the View is rendered.
     */
    @CallSuper
    public void draw(Canvas canvas) {
        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         *      7. If necessary, draw the default focus highlight
         */
    }
  
```

> Draw流程其实比较复杂，涉及到软件绘制和硬件加速，Surface、SurfaceFlinger等，详情请见我的相关博客。此处略

# 参考

- https://juejin.im/post/5cc030686fb9a0323c526adf
