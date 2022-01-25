---
title: Activity绑定视图流程概览
date: 2021-09-20 15:59:11
tags:
- android
- android系统开发
categories:
- Android系统开发
---

我们在`Activity.onCreate()`方法里调用`setContentView()`方法后，待Activity处于RESUME状态时就会呈现视图。这一过程涉及到Activity的启动以及视图的创建和绘制，那么Activity是如何与视图(Window、View)产生联系或绑定在一起的？

# 概述

流程关键节点：

- 启动Activity。`Activity.startActivity()`。
- 创建窗口。Activity启动流程中会调用`ActivityThread.performLaunchActivity()`方法，该方法内部通过`Activity.attach()`方法创建PhoneWindow实例，并与Activity绑定。
- 创建视图树。`ActivityThread.performLaunchActivity()`方法中待创建PhoneWindow后，接着会执行`Instrumentation.callActivityOnCreate()`、`Activity.onCreate()`, 最终会调用`Activity.setContentView()`解析视图树并与Window绑定。
- 绘制视图。在Activity启动流程的方法`ActivityStackSupervisor.realStartActivityLocked()`中会依次添加*Launch*和*Resume*事务，待执行完`ActivityThread.performLaunchActivity()`等*Launch*方法后，会接着执行`ActivityThread.handleResumeActivity()`等*Resume*方法，该方法内部通过`WindowManager.addView()`方法绘制视图呈现UI。

简略时序图如下：

![](/images/android-activity-bind-ui-flow.png)

经过上面的流程，最终在App中视图相关的各类对象间关系为：

- Activity持有PhoneWindow引用，PhoneWindow持有视图树根节点DecorView引用，DecorView对象又持有抽象根节点ViewRootImpl。
- ViewRootImpl由WindowManagerGlobal创建和管理，而它自身管理视图树的绘制。
- ViewRootImpl通过Session与WindowManagerService建立联系，由服务端管理窗口。
- Activity、PhoneWindow、DecorView、ViewRootImpl、Session和WindowState是一一对应关系。
- WindowManagerGlobal为单例，App进程全局唯一。

![](/images/android-window-view-relation.svg)

上面的箭头表示有联系，直接持有引用或间接联系。

<!-- more -->

# 启动Activity

Activity的启动流程见我的博客：{% post_link android-activity-start-guide %}

# 创建窗口

在`Activity.attach()`方法创建PhoneWindow实例：

```java
final void attach(Context context, ActivityThread aThread ...) {
    // 没有保存的window时，window为空
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
}

public PhoneWindow(Context context, Window preservedWindow...) {
    // 有保存的window时，直接绑定之前的DecorView
    if (preservedWindow != null) {
        mDecor = (DecorView) preservedWindow.getDecorView();
    }
}
```

PhoneWindow对象实例化时，先判断是否有缓存window时，有就直接绑定之前的DecorView，DecorView是视图树的根节点；没有缓存window则会在之后的`Activity.setContentView()`时绑定DecorView。

# 创建视图树

## 概述

在`Activity.setContentView(@LayoutRes int layoutResID)`中解析并创建视图树。在PhoneWindow中创建DecorView实例，利用LayoutInflater解析布局文件。主要流程：

- 生成`DecorView`（继承FrameLayout）实例。
- 解析`screen_*.xml`布局（包含ID为content的FrameLayout，对应引用为`mContentParent`）并填充到`DecorView`中。
- 解析App自定义的布局文件并填充到`mContentParent`引用的FrameLayout中。

简略时序图：

![](/images/android-set-content-view-sq.svg)

最终生成的视图树结构：

![](/images/android-activity-ui-structure.svg)

`screen_*.xml`文件位于`base/core/res/res/layout`目录下，`screen_titil.xml`示例：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:fitsSystemWindows="true">
    <!-- Popout bar for action modes -->
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
        android:layout_width="match_parent" 
        android:layout_height="?android:attr/windowTitleSize"
        style="?android:attr/windowTitleBackgroundStyle">
        <TextView android:id="@android:id/title" 
            style="?android:attr/windowTitleStyle"
            android:background="@null"
            android:fadingEdge="horizontal"
            android:gravity="center_vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </FrameLayout>
    <!-- 此处填充App自定义的内容布局 -->
    <FrameLayout android:id="@android:id/content"
        android:layout_width="match_parent" 
        android:layout_height="0dip"
        android:layout_weight="1"
        android:foregroundGravity="fill_horizontal|top"
        android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```

## 关键方法

### `Activity.setContentView()`

```java
public void setContentView(@LayoutRes int layoutResID) {
    // mWindow此处其实现为PhoneWindow
    getWindow().setContentView(layoutResID);
    // 初始化ActionBar
    initWindowDecorActionBar();
}
```

### `PhoneWindow.setContentView()`

```java
public void setContentView(int layoutResID) {
    // 创建Decor和Activity内容视图的Parent
    if (mContentParent == null) {
        // 该方法的工作：
        // 1.生成DecorView实例： mDecor = generateDecor(-1) -> new DecorView(context...);
        // 2.填充App内容的父布局：mContentParent = generateLayout(mDecor);
        installDecor();
    }
    
    // 普通情况，解析App自定义布局文件并生成视图，添加到mContentParent中
    mLayoutInflater.inflate(layoutResID, mContentParent);
}
```

### `PhoneWindow.generateLayout()`

填充App内容的父布局，一般包括标题区域和内容区域，内容区域的视图ID固定为`ID_ANDROID_CONTENT = "com.android.internal.R.id.content"`。

```java
protected ViewGroup generateLayout(DecorView decor) {
    // 填充布局，工作如下：
    // 首先，利用LayoutInflater.inflate()方法解析布局文件，如`screen_*.xml`文件
    // 其次，将解析后的视图添加到DecorView中，DecorView作为整个新视图的根节点
    mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
    // 绑定Activity内容视图的父布局, 一般为FrameLayout, ID固定
    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
    return contentParent;
}
```

### `LayoutInflater.inflate()`

解析内容布局文件生成视图树并添加到内容视图容器(`ID_ANDROID_CONTENT`)里。

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
    final Context inflaterContext = mContext;
    final AttributeSet attrs = Xml.asAttributeSet(parser);
    // 资源根布局。这里的root是mContentParent引用, 指向ID为ID_ANDROID_CONTENT的ViewGroup
    final View temp = createViewFromTag(root, name, inflaterContext, attrs);
    // 递归解析
     rInflateChildren(parser, temp, attrs, true);
    // 添加内容视图树到mContentParent视图里
    if (root != null && attachToRoot) {
        root.addView(temp, params);
    }
    return result;
}
```

# 绘制视图

## 概述

视图绘制的入口为`ActivityThread.handleResumeActivity()`，该方法主要工作为：

- 添加Window。调用方法中的`WindowManager.addView()`创建`ViewRootImpl`，与`DecorView`绑定。在`ViewRootImpl`中创建`Window`，并将该`Window`通过`Session`添加到`WMS`里，其代表为`WindowState`。在此过程`WindowState`会创建与`SurfaceFlinger`的连接`SurfaceSession`（Java层）和`SurfaceComposerClient`（Native层），只有与SurfaceFlinger连接后绘制的视图才能渲染呈现出来。
- 执行具体绘制操作。与WindowManagerService建立连接的前、后都会去执行绘制操作，连接前调用`ViewRootImpl.requestLayout()`，连接后调用`Activity.makeVisible()`（该方法是否会被调用？**TODO**），但无论哪个绘制入口，最终都会调用`ViewRootImpl.scheduleTraversals()`。

> 本节只讨论添加Window的操作，不讨论具体View绘制以及与Surface、SurfaceFlinger相关的流程，具体绘制流程见我的相关博客。
>
> 本节WMS与Window逻辑撰写不够清晰，如待以后更新 **TODO**。

时序图：

![activity-perform-resume](/images/android-activity-perform-resume.svg)

## 关键类

这块涉及到的类比较多和繁杂，有必要单独详细说明一下。类图如下：

![](/images/android-ui-class-uml.svg)

App端：

- Window。用户与App交互的顶级视图，可理解为具体视图View的容器，与底层的Surface一一对应，多个Window或多个Surface最终形成一帧图像呈现给用户。
- PhoneWindow，继承Window。Activity持有窗口实例，持有成员：DecorView、状态栏各属性、主题、导航和标题等属性等。
- ViewRootImpl，继承ViewParent。App端视图树抽象的根，但不是实际的View，通过Session与服务端通信，并管理视图树的具体绘制。
- DecorView，继承FramLayout。窗口实际视图树的根View，持有ViewRootImpl作为父。
- WindowManager，继承ViewManger接口。包含View的添加、更新和移除等操作方法。
- WindowManagerImpl，实现WM。WMImp持有WMGlobal单例，WMImpl在生成实例时会生成全局（应用）唯一的WMGlobal。
- WindowManagerGlobal，单例。提供与WMS低级别通信的方法，与Context无关，持有成员：DecorView集合、ViewRootImpl集合以及布局参数集合等。

Service端：

- ViewRootImpl.W，实现IWindow。客户端与服务端通信时代表客户端的窗口，持有成员：ViewRootImpl和Session。这个与App端的Window不是同一个东西，是它的代表。
- Session，实现IWindowSession。用于App端与服务端间的通信，每个App端都有一个Session与服务端连接。
- WMS，实现IWindowManager。主要用于管理App端的窗口：为窗口分配Surface，创建与SurfaceFlinger的连接，掌管Surface的显示顺序（Z序）以及大小尺寸、控制窗口动画，另外还管理输入事件的通道。
- WindowState，实现WMPolicy.WindowState。用于管理IWindow，持有成员：IWindow、Session等。

## 关键方法

### ActivityThread.handleResumeActivity()

主要工作：执行Resume并最终执行Activity.onResume()方法、添加Window、执行绘制

```java
public void handleResumeActivity(IBinder token...) {
    // 该方法内部会依次调用Activity的onStart()或onRestart()、onResume()等方法
    final ActivityClientRecord r = performResumeActivity(token, ...);
    if (!a.mWindowAdded) {
        // 添加view绑定到Window中, 内部调用：
        // -> WindowManagerImpl.addView() -> WindowManagerGlobal.addView()
        a.mWindowAdded = true;
        wm.addView(decor, l);
    }
    // 执行绘制
    if (r.activity.mVisibleFromClient) {
        // 内部调用:
    	// DecorView.setVisibility(View.VISIBLE) -> View.invalidate(true)...
        r.activity.makeVisible();
    }
}
```

### WindowManagerGlobal.addView()

创建ViewRootImpl

```java
// android.view.WindowManagerGlobal
public void addView(View view, ViewGroup.LayoutParams params...) {
    // 创建ViewRootImpl实例，被WMG持有
    // DecorView也被WMG持有
    ViewRootImpl  root = new ViewRootImpl(view.getContext(), display);
    view.setLayoutParams(wparams);
    mViews.add(view);
    mRoots.add(root);
    mParams.add(wparams);
    root.setView(view, wparams, panelParentView);
    ...
}
```

### ViewRootImpl.setView()

```java
public void setView(View view, WindowManager.LayoutParams attrs, ...) {
    // 在添加到WMS之前先执行一次绘制
    requestLayout();
    // 通过Session添加window到WMS IPC, 
    // Session是ViewRootImpl在构造方法里从WMS获取的
    res = mWindowSession.addToDisplay(mWindow, mSeq, ...);
    // 设置DecorView的parent为ViewRootImpl
    view.assignParent(this);
}
```

### WindowManagerService.addWindow()

创建WindowState、SurfaceSession等

```java
public int addWindow(Session session, IWindow client, int seq,...){
    // WindowState代表WM里的Window
    final WindowState win = new WindowState(this, session, client, ...);
    // 一个window只能有一个输入通道，发送输入事件到window
    if  (openInputChannels) {
        win.openInputChannel(outInputChannel);
    }
    // session 绑定packagename，生成SurfaceSession实例
    win.attach();
}
```







