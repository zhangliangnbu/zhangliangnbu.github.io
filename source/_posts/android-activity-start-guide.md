---
title: Andoird 10 Activity启动流程概览
date: 2021-12-17 15:49:34
tags:
- android
- android系统开发
categories:
- Android系统开发
---

# 前言

基于Android 10 源码 。Activity启动分为两种：

- 普通Activity启动：APP已经启动，一般在APP内从一个Activity启动另一个Activity。
- 根Activity启动：App还没有启动，一般从Launcher界面启动。

两者主流程基本一致，区别在于根Activity启动流程还包含启动APP的流程。

# 普通Activity启动流程

分析方法`Activity.startActivity()` 到`Activity.onCreate()`的流程，流程图如下。

![aosp10_activity_start_create.drawio](../images/aosp10_activity_start_create.drawio.svg)

流程说明：

1. 客户端发起启动Activity请求。客户端里的Activity通过`Instrumentation.execStartActivity()`请求服务端的ATMS启动Activity。
2. 服务端完成Intent解析和数据、权限、状态等检查的准备工作。ATMS通过ActivityStarter、ActivityStack、ActivityStackSupervisor等完成Intent解析、检查等准备工作，并将数据传递给客户端的ApplicationThread，请求后者完成剩余启动工作。
3. 客户端完成Activity的创建、数据绑定、onCreate回调等工作。ApplicationThread通过ActivityThread将数据传到其主线程的Handler中进行处理，完成Activity的创建、数据绑定和onCreate回调等工作。

**关键方法说明**

**`Instrumentation.execStartActivity()`：请求服务端的ATMS启动Activity**

```java
public ActivityResult execStartActivity(Context who, IBinder contextThread, IBinder token ...) {
    // IPC：调用 ActivityTaskManagerService
    int result = ActivityTaskManager.getService().startActivity(
        whoThread, // 发起方 IApplicationThread
        who.getBasePackageName(), // 发起方 package name
        intent, // 目标
        // Data MIME 类型，没有数据时为空
        intent.resolveTypeIfNeeded(who.getContentResolver()),
        token,  // 发起方 IBinder
        target != null ? target.mEmbeddedID : null, // mEmbeddedID 作用？TODO
        requestCode, 
        0, // startFlag
        null, // ProfilerInfo profilerInfo 作用？TODO
        options);
}
```

**`ActivityStarter.startActivityMayWait()`：解析Intent**

```java
private int startActivityMayWait(IApplicationThread caller, int callingUid ...) {
    // 解析Intent获取ResolveInfo，核心是获取其成员ActivityInfo。
    // 最终调用 PMS#queryIntentActivitiesInternal()：
    // 1. 通过Intent中ComponentName获取安装apk或扫描时解析过并保存的ActivityInfo;
    // 2. 设置ActivityInfo的metaData和applicationInfo
    // 上面的ActivityInfo作为新建的ResolveInfo成员，跟随后者一起被返回
    ResolveInfo rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId...);
    // 普通情况仅仅将rInfo的数据设置到intent中
    ActivityInfo aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, profilerInfo);
}
```

**`ActivityStarter.startActivity()`：检查权限等**

```java
private int startActivity(IApplicationThread caller, Intent intent ...) {
    // 检查启动方的权限，系统方或同一个APP等则可授权
    boolean abort = !mSupervisor.checkStartAnyActivityPermission(intent, aInfo, resultWho ...);
    abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid ...);
    abort |= !mService.getPermissionPolicyInternal().checkStartActivity(intent, callingUid,callingPackage);
}
```

**`ActivityStackSupervisor.realStartActivityLocked()`: 生成事务，为IPC到客户端创建Activity准备**

```java
boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc...){
    // 准备工作：锁定屏幕、开始计时、绑定进程、确保可见和配置...
    ...
    // 添加启动Activity事务项
    final ClientTransaction clientTransaction = ClientTransaction.obtain(proc.getThread(), r.appToken);
    final DisplayContent dc = r.getDisplay().mDisplayContent;
	clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent), ...));
    // 添加使Activity处于resume事务项
    // 在TransactionExecutor.execute()里会依次处理callback、LifecycleState
    final ActivityLifecycleItem lifecycleItem;
    if (andResume) {
        lifecycleItem = ResumeActivityItem.obtain(dc.isNextTransitionForward());
    } else {
        lifecycleItem = PauseActivityItem.obtain();
    }
    clientTransaction.setLifecycleStateRequest(lifecycleItem);
    // Schedule transaction.
	// ActivityTaskManagerService
    mService.getLifecycleManager().scheduleTransaction(clientTransaction);
}
```

**`ActivityThread.performLaunchActivity()`: 创建Activity，绑定数据等，执行`Activity.onCreate()`回调。**

```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // activity 准备创建需要的相关数据，创建
    ActivityInfo aInfo = r.activityInfo;
    ComponentName component = r.intent.getComponent();
    ContextImpl appContext = createBaseContextForActivity(r);
    java.lang.ClassLoader cl = appContext.getClassLoader();
    // 反射创建Activity对象，还不能交互
    Activity activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    // 如果已经存在，直接返回
    Application app = r.packageInfo.makeApplication(false, mInstrumentation);
    // 绑定Activity与其他数据：绑定Context，创建PhoneWindow，绑定线程、Token、Application等
    activity.attach(appContext, this, getInstrumentation(), r.token...);
    // 触发onCreate调用
    mInstrumentation.callActivityOnCreate(activity, r.state);
}
```





### 涉及的主要类

```java
core/
    android.app.Activity
    android.app.Instrumentation
    android.app.ActivityThread
    android.app.ActivityThread.ApplicationThread
    android.app.servertransaction.LaunchActivityItem
    android.app.servertransaction.TransactionExecutor
    android.app.servertransaction.ClientTransaction

services/core/
    com.android.server.wm.ActivityTaskManagerService 
    com.android.server.wm.ActivityStarter
    com.android.server.wm.RootActivityContainer
    com.android.server.wm.ActivityStack
    com.android.server.wm.ActivityStackSupervisor
    com.android.server.wm.ClientLifecycleManager
```
