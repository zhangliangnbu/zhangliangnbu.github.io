---
title: Android图形系统概览
date: 2021-10-23 17:01:27
tags:
- android
- android系统开发
categories:
- Android系统开发
---

当我们调用`TextView.setTextColor()`之后，UI就会更新，这一过程是怎样的？本文通过较完整的视图绘制流程分析Android的图形系统。

> 这里的绘制流程是指UI从数据更改触发到最终更新的流程。

代码基于Android 10。

<!-- more -->

# 概述

对于Android图形系统，可以这样理解：App中我们见到状态栏、Activity界面、弹窗等都是一个个窗口，对应App端的Window和服务端的WindowState；每个窗口包含多个控件，对应View和ViewGroup；每个窗口独占单独的画布，对应Surface（持有Canvas）；多个画布内容通过SurfaceFlinger合成一帧画面，最后通过显示设备输出。

> 说得比较粗糙，实际情况较复杂，待以后优化 **TODO**

简单的对应关系如图：

![wms-client-server-ui](/images/wms-client-server-ui.svg)

Window和WindowState都间接持有Surface的引用，WindowState持有与SurfaceFlinger的连接Session。可通俗理解为，多个View控件构成一个Surface内容，多个Surface内容通过SurfaceFlinger合成为一帧最终输出的图像。

下面以具体代码来分析Android图形系统，分析`TextView.setTextColor()`从调用到UI更新的调用链流程，可分为：

- 准备阶段：计算脏区位置和宽高（Dirty Rect，就是待更新区域）、测量、布局，之后调用`ViewRootImpl.performDraw()`尝试硬件绘制，不能或不成功则进行软件绘制。
- 绘制操作：区分软件绘制和硬件绘制，绘制数据有所差别，但最终都是将绘制数据填充到GraphicBuffer，通知SurfaceFlinger进行合成。
- 合成展示：SurfaceFlinger利用硬件(OpenGL 和 HardWare Composer)将 GraphicBuffer 数据合成并交给Display Buffer去显示。

整个流程如下：

![android-draw-total-flow](/images/android-draw-total-flow.svg)

> 这幅图参考[Android-Surface原理解析](https://ljd1996.github.io/2020/11/09/Android-Surface%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/)，图例比较清晰全面，我直接拿过来并重新绘制了全图。

# 准备阶段

准备阶段时序图如下：

![android-set-text-color-sq](/images/android-set-text-color-sq.svg)

> 略繁琐，逻辑不够清晰，待以后更新 **TODO**

大致流程为：首先计算脏区尺寸和位置，之后调用`ViewRootImpl.performTraversals()`进行测量、布局和绘制操作。



## 计算脏区在屏幕中的位置和宽高

计算时会调用`ViewGroup.invalidateChild()`遍历当前View的所有的`ViewParent`，直到`ViewRootImpl`为止。

```java
// ViewGroup.java
// 这个child是实例中待更新的TextView
public final void invalidateChild(View child, final Rect dirty) {
    ViewParent parent = this;
    final int[] location = attachInfo.mInvalidateChildLocation;
    location[CHILD_LEFT_INDEX] = child.mLeft;
    location[CHILD_TOP_INDEX] = child.mTop;
    do {
        // 遍历所有的ViewParentt以计算脏区. 首先进入ViewGroup，最后进入ViewRootImpl
        parent = parent.invalidateChildInParent(location, dirty);
    } while (parent != null);
}
```

## 测量视图树中所有View的宽高

在`ViewRootImpl.performTraversals()`中会依次调用`performMeasure()`、`performLayout()`、`performDraw()`三个主要方法。`performMeasure()`即测量。测量的工作是确定View的宽高，测量的主要规则通过父布局的MeasureSpec确定子View的宽高。

详细流程：首先确定根View的MeasureSpec；然后通过父布局的MeasureSpec确定子View的MeasureSpec；最后在onMeasure方法中根据这个MeasureSpec来确定View的测量宽高。

```java
// ViewRootImpl.java
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
    // 这里的mView指根View, 一般为DecorView
	mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    // ViewGroup控件调用 ViewGroup.onMeasure(), 非ViewGroup调用View.onMeasure()
    onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```

## 布局

在`ViewRootImpl.performTraversals()`中`performLayout()`执行布局。**TODO**

之后，在`ViewRootImpl.performDraw()`中首先尝试硬件绘制，不能或不成功则软件绘制。软件绘制会调用`ViewRootImpl.drawSoftware`，其中会通过`Surface`生成`Canvas`以传递给视图树使用。

# 绘制阶段

## Surface的创建

绘制流程中涉及到Surface，有必要先描述Surface相关对象的创建。相关对象有：SurfaceSession、SurfaceControl和Surface。这三个对象的Java层实例都持有其Native层的指针，实际绘制工作最终都由Native层完成。

### 概述

SurfaceSession是App端与SurfaceFlinger的连接，可用于创建的SurfaceControl，后者用于创建Surface。

> 这里说的创建对象，包括Java层和Native层

在 Java 层 ViewRootImpl 实例中持有一个 Surface 对象，该 Surface 对象中的 mNativeObject 属性指向 native 层中创建的 Surface 对象，native 层的 Surface 对应 SurfaceFlinger 中的 Layer 对象，它持有 Layer 中对应的 BufferQueueProducer 生产者指针，在 Surface 上绘制的内容最终会交由 SurfaceFlinger 来合成渲染送到显示器显示。

创建过程中，涉及到的Java层和Native层的类：

![android-surface-create-class-uml](/images/android-surface-create-class-uml.svg)

### **SurfaceSession**

代表Surface与SurfaceFlinger 的连接，通过该连接可以创建多个Surface实例；持有客户端SurfaceComposerClient的指针，连接的实现由其实现，可用于创建SurfaceControl。

SurfaceSession对象是在Window添加流程中创建的，在`WMS.addWindow() `方法中创建 WindowState，它会调用`WindowState.attach()`方法，该方法内调用`Session.windowAddedLocked()`，在这里创建Java端的SurfaceSession对象。SurfaceSession构造方法会通过JNI创建一个 Native端的SurfaceComposerClient 对象，后者又创建了一个 实现了ISurfaceComposerClient接口的Client 对象，通过此对象与SurfaceFlinge通信。

创建的时序图：

![android-surface-create-sq-flow](/images/android-surface-create-sq-flow-16403397442681.svg)

相关代码：由于涉及C++代码，稍微贴下较详细的代码

```c++
// Session.java
void windowAddedLocked(String packageName) {
    mSurfaceSession = new SurfaceSession();
}

// SurfaceSession.java
private long mNativeClient; // SurfaceComposerClient*
// 创建与Surface Flinger的连接
public SurfaceSession() {
    mNativeClient = nativeCreate();
}

// frameworks/base/core/jni/android_view_SurfaceSession.cpp
static jlong nativeCreate(JNIEnv* env, jclass clazz) {
    SurfaceComposerClient* client = new SurfaceComposerClient();
    client->incStrong((void*)nativeCreate);
    return reinterpret_cast<jlong>(client);
}

// 调用incStrong函数时，如果SurfaceComposerClient是第一次被引用则会调用onFirstRef()函数
// frameworks/native/libs/gui/SurfaceComposerClient.cpp
void SurfaceComposerClient::onFirstRef() {
    // 从服务端(surface flinger)获取服务
    sp<ISurfaceComposer> sf(ComposerService::getComposerService());
    // 通过服务创建连接
    sp<ISurfaceComposerClient> conn = sf->createConnection();
    mClient = conn;
}

// frameworks/native/services/surfaceflinger/SurfaceFlinger.cpp
sp<ISurfaceComposerClient> SurfaceFlinger::createConnection() {
    // initClient方法只是调用initCheck检查了一下
    return initClient(new Client(this)); 
}
```

### **SurfaceControl**

用于Surface的创建等管理。

客户端ViewRootImpl中持有一个SurfaceControl，但开始是无内容无效的，没有持有Native层对象的指针。`ViewRootImpl.performTraversals()`中会调用`ViewRootImpl.relayoutWindow()`方法，之后会通过Session将此SurfaceControl传给WMS，WMS创建WindowSurfaceController对象，该对象构造器内部创建并持有新的SurfaceControl实例，它通过`nativeCreate()`方法创建底层实例即客户端的SurfaceControl以及SurfaceFlinger端的Layer，并持有底层SurfaceControl的指针。

另外：创建SurfaceControl流程中会创建surface flinger端的layer对象：BufferQueueLayer，并创建它的两个重要成员：BufferQueueProducer 和 BufferQueueConsumer的包装类对象。SurfaceControl和Surface持有Layer中对应的BufferQueueProducer指针，只负责生产数据流。

创建的时序图如下：

![android-surface-control-create-sq-flow](/images/android-surface-control-create-sq-flow.svg)

关键代码：

```java
// ViewRootImpl.java
private void performTraversals() {
    // 需要进行布局操作时
    relayoutWindow(params, viewVisibility, insetsPending)
    // measure, layout, draw
}

private int createSurfaceControl(SurfaceControl outSurfaceControl...) {
    // 最终通过SurfaceSession创建SurfaceControl, 并创建Native层对象
    WindowSurfaceController surfaceController = winAnimator.createSurfaceLocked();
}

// frameworks/base/core/jni/android_view_SurfaceControl.cpp
static jlong nativeCreate(jobject sessionObj, ...) {
    // sessionObj 即SurfaceSession，在Window添加流程中已创建
    sp<SurfaceComposerClient> client = android_view_SurfaceSession_getClient(...sessionObj);;
    // 创建SurfaceControl, 最终通过Native层的连接ISurfaceComposerClient创建
    client->createSurfaceChecked(..., &surface, ..., parent, ...));
    return reinterpret_cast<jlong>(surface.get());
}

// frameworks/native/libs/gui/SurfaceComposerClient.cpp
status_t SurfaceComposerClient::createSurfaceChecked() {
    // 这里的mClient是指接口ISurfaceComposerClient, 详情见上面SurfaceSession创建流程
    status_t err = mClient->createSurface(..., &gbp);
    *outSurface = new SurfaceControl(this, handle, gbp, true /* owned */);
    return err;
}

// frameworks/native/services/surfaceflinger/Client.cpp
status_t Client::createSurface(...sp<IGraphicBufferProducer>* gbp) {
    // 通过SurfaceFlinger创建Layer
    return mFlinger->createLayer(...handle, gbp, parentHandle);
}
```

### Surface

在`ViewRootImpl.relayoutWindow()`中，首先调用`mWindowSession.relayout()`生成并初始化SurfaceControl，接着在Native 层中，通过SurfaceControl创建Surface，并将地址指针赋值给Java层Surface的mNativeObject成员。

![android-surface-self-create-sq-flow](/images/android-surface-self-create-sq-flow.svg)

关键代码：

```java
// ViewRootImpl.java
private int relayoutWindow(WindowManager.LayoutParams params, ...) {
    // 初始化SurfaceControl
    mWindowSession.relayout(mWindow,...,mSurfaceControl,...);
    // 按理说Binder通信会copy数据，但最最终是否会修改此处的mSurfaceControl？
    // 如果此处不修改，又在何处修改呢，找不出明确修改mSurfaceControl的地方，但感觉应该修改了
    mSurface.copyFrom(mSurfaceControl);
}

// frameworks/base/core/jni/android_view_Surface.cpp
static jlong nativeGetFromSurfaceControl(...jlong surfaceControlNativeObj) {
    Surface* self(reinterpret_cast<Surface *>(nativeObject));
    sp<SurfaceControl> ctrl(reinterpret_cast<SurfaceControl *>(surfaceControlNativeObj));
    // 首次需要创建
    sp<Surface> surface(ctrl->getSurface());
    return reinterpret_cast<jlong>(surface.get());
}

// frameworks/native/libs/gui/SurfaceControl.cpp
sp<Surface> SurfaceControl::generateSurfaceLocked() const {
    // gbp在SurfaceControl时已创建
    mSurfaceData = new Surface(mGraphicBufferProducer, false);
    return mSurfaceData;
}
```

## 绘制操作

绘制即在Android APP 进程中将 UI 绘制到一个图形缓冲区 GraphicBuffer 中的过程。绘制操作包括两种实现方式：软件绘制和硬件绘制（也叫硬件加速，下面统一称为硬件绘制）。硬件绘制会使用 GPU将图形 加速渲染到 GraphicBuffer 。

绘制入口为`ViewRootImpl.draw()`方法：

```java
private boolean draw(boolean fullRedrawNeeded) {
    if (mAttachInfo.mThreadedRenderer != null && mAttachInfo.mThreadedRenderer.isEnabled()) {
        mAttachInfo.mThreadedRenderer.draw(mView ...); // 硬件绘制
    } else {
        drawSoftware(surface, ...) // 软件绘制
    }
    return useAsyncReport;
}
```



### 概述

**软件绘制**

当 App 更新部分 UI 时，CPU 会遍历 View Tree 计算出需要重绘的脏区，接着在 View 层次结构中绘制所有跟脏区相交的区域，之后将绘制的内容写进一个 Bitmap 位图，这个 Bitmap 的像素内容会填充到 Surface 的缓存区里。软件绘制使用 Skia 库。

缺点：可能重绘无需更新的视图；主线程绘制容易造成卡顿情况。

**硬件绘制**

当 App 更新部分 UI 时，CPU 会计算出脏区，但是不会立即执行绘制命令，而是将 drawXXX 函数作为绘制指令(DrawOp)记录在一个列表(DisplayList)中，然后交给单独的 Render 线程使用 GPU 进行硬件加速渲染。

硬件绘制使用 OpenGL 在 GPU 上完成，OpenGL 是跨平台的图形 API，为 2D/3D 图形处理硬件制定了标准的软件接口。Skia 已接手 OpenGL，实现间接统一调用。

优点：

1. 无需重绘所有相交区域。只需要针对需要更新的 View 对象的脏区进行记录或更新，无需更新的 View 对象则能重用先前 DisplayList 中记录的指令。
2. 单独线程绘制可缓解卡顿情况。硬件加速是在单独的 Render 线程中完成绘制的，分担了主线程的压力，提高了响应速度。

缺点：兼容性（部分绘制函数不支持加速），内存消耗，电量消耗（GPU耗电）等。

>  从 Android 3.0(API 11)开始支持硬件加速，Android 4.0(API 14)默认开启硬件加速。

配置硬件绘制：

- Application: 在 Manifest 文件的 application 标签添加 `android:hardwareAccelerated="boolean"`
- Activity: 在 Manifest 文件的 activity 标签添加 `android:hardwareAccelerated="boolean"`
- Window: `getWindow().setFlags(WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED, WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED)`
- View: `setLayerType(View.LAYER_TYPE_HARDWARE/*View.LAYER_TYPE_SOFTWARE*/, mPaint)`

### 软件绘制

软件绘制的入口为`ViewRootImpl.drawSoftware()`，可以简单分为三个步骤：

1. `Surface.lockCanvas `方法通过 `BufferQueueProducer.dequeueBuffer()` 函数从 BufferQueue 中取出一个图形缓存区 GraphicBuffer(用来创建 Canvas 中的 Bitmap 对象) 并锁定该Surface，然后将 Surface 的地址返回给 Java 层 Surface 中的 mLockedObject属性。在这个方法中还会涉及到 Surface 的双缓冲逻辑。
2. 调用 `View.draw()` 方法将内容绘制到 Canvas 对应的 Bitmap 中，其实就是往上面的图形缓存区 GraphicBuffer 填充绘制数据。
3. `Surface.unlockCanvasAndPost()` 方法通过调用被锁定的 `surface->unlockAndPost()` 方法解锁 Surface 且通过 `queueBuffer()` 函数将填充了数据的图形缓存区 GraphicBuffer 存入 BufferQueue 队列中，然后通知给 SurfaceFlinger 进行合成(请求 Vsync 信号)。

对于软件绘制中的 Canvas 而言其绘制目标是一个 Bitmap 对象，绘制的内容会填充到 Surface 持有的缓存区(GraphicBuffer)里。

**简单流程即：GraphicBuffer出队→写数据到Bitmap中→GraphicBuffer入队合成。**

> 待补流程图 **TODO** 

### 硬件加速

硬件加速可以从两个阶段来看：

1. 构建阶段：将 View 抽象成 RenderNode 节点，将每个绘制操作(drawLine等)抽象成 DrawOp，它保存了绘制数据并与 OpenGL 绘制命令对应。这个阶段会递归遍历所有 View 并通过 Canvas.drawXXX将绘制操作转化成 DrawOp 存入 DisplayList 中，根据 ViewTree 模型，这个 DisplayList 虽然命名为List，但其实更像一棵树。
2. 绘制阶段：通过单独的 Render 线程，依赖 GPU 绘制上面的 DrawOp 数据。其中硬件加速的内存申请跟软件绘制一样都是借助 Layer 中的 BufferQueueProducer 生产者从 BufferQueue 中出队列一块空闲缓存区 GraphicBuffer 用来渲染数据的，之后也都会通知 SurfaceFlinger 进行合成。不一样的地方在于硬件加速相比软件绘制而言算法可能更加合理，同时采用了一个单独的 Render 线程，减轻了主线程的负担。

### 绘制中的双缓冲

一般来说将双缓冲用到的两块缓冲区称为 – 前缓冲区(front buffer) 和 后缓冲区(back buffer)。显示器显示的数据来源于front buffer 前缓存区，而每一帧的数据都绘制到 back buffer 后缓存区，在 Vsync信号到来后会交互缓存区的数据(指针指向)，这时 front buffer 和 back buffer 的称呼及功能倒转。

**软件绘制中的双缓冲**

`Surface.lock()`:

1. 将出队列的空闲缓存区 GraphicBuffer 赋给后缓存区 backBuffer，将正在显示的 mPostedBuffer 赋给前缓存区。
2. 计算新的脏区，并确定是否需要将前缓存区拷贝到后缓存区，依此计算出后缓存区 backBuffer 的最终数据。然后将 backBuffer 与应用层的 Canvas 关联，当操作 Canvas 绘图时会将数据绘制到 backBuffer 上。
3. 锁定 backBuffer 且将 backBuffer 指针赋值给 mLockedBuffer。

`Surface.unlockAndPost()`:

1. 将存有绘制数据的 mLockedBuffer 解锁并将其赋值给 mPostedBuffer。
2. 将 mLockedBuffer 入 BufferQueue 队列，等待被合成显示，在这里便相当于交换了前后缓冲区的指针，等到下次绘制时，接着重复上面的步骤。

**硬件绘制中的双缓冲**

硬件绘制最终会调用 CanvasContext.draw 方法来绘制，也用到双缓冲。略... **TODO**

# 合成和显示

待更新 **TODO**

# 结尾

写的太长，累(⊙o⊙)…准备阶段缺少测量和布局流程、绘制缺少时序图和关键代码，合成和显示缺少整个章节，待以后慢慢整理、补充。

另外说明的是，我的文章仅仅从UI更新的流程这一个角度来分析Android图形系统，依旧不全面，一些偏底层的部分涉及较少，详细情况可参考官方的文档：[Android 图形概览](https://source.android.google.cn/devices/graphics?hl=zh-cn#android-graphics-components)

# 参考

- [Android 图形概览](https://source.android.google.cn/devices/graphics?hl=zh-cn#android-graphics-components)
- [Android 10 源码](http://aospxref.com/android-10.0.0_r47/)
- [三方博客：Android-Surface原理解析](https://ljd1996.github.io/2020/11/09/Android-Surface原理解析/)
- [三方博客：C++中sp和wp](https://www.cnblogs.com/solomarge/p/14325080.html)
- [三方博客：C++创建对象时区分圆括号( )和大括号{ }](https://zhuanlan.zhihu.com/p/268894227)
