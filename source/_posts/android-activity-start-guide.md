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
- 根Activity启动：也称为APP冷启动，App还没有启动，一般从Launcher界面启动。

两者主流程基本一致，区别在于根Activity启动流程还包含启动APP进程和实例化Application的流程。

<!-- more -->

# 普通Activity启动流程

## 流程概述

分析方法`Activity.startActivity()` 到`Activity.onCreate()`的流程，流程图如下。

![aosp10-activity-start-create](/images/aosp10-activity-start-create.svg)

流程说明：

1. 客户端发起启动Activity请求。客户端里的Activity通过`Instrumentation.execStartActivity()`请求服务端的ATMS启动Activity。
2. 服务端完成Intent解析和数据、权限、状态等检查的准备工作。ATMS通过ActivityStarter、ActivityStack、ActivityStackSupervisor等完成Intent解析、检查等准备工作，并将数据传递给客户端的ApplicationThread，请求后者完成剩余启动工作。
3. 客户端完成Activity的创建、数据绑定、onCreate回调等工作。ApplicationThread通过ActivityThread将数据传到其主线程的Handler中进行处理，完成Activity的创建、数据绑定和onCreate回调等工作。

## 方法说明

### `Instrumentation.execStartActivity()`

请求服务端的ATMS启动Activity

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

### `ActivityStarter.startActivityMayWait()`

解析Intent

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

### `ActivityStarter.startActivity()`

检查权限等

```java
private int startActivity(IApplicationThread caller, Intent intent ...) {
    // 检查启动方的权限，系统方或同一个APP等则可授权
    boolean abort = !mSupervisor.checkStartAnyActivityPermission(intent, aInfo, resultWho ...);
    abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid ...);
    abort |= !mService.getPermissionPolicyInternal().checkStartActivity(intent, callingUid,callingPackage);
}
```

### `ActivityStackSupervisor.realStartActivityLocked()`

生成事务，为IPC到客户端创建Activity准备

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

### `ActivityThread.performLaunchActivity()`

创建Activity，绑定数据等，执行`Activity.onCreate()`回调。

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

# 根Activity启动流程

## 流程概述

一般也称为APP冷启动流程，分析从Launcher界面点击APP图标到APP显示完成的流程。相比从普通Activity启动流程，App的冷启动流程多了两个步骤：创建进程，绑定并创建Application对象。

![aosp10_app_start_cold](/images/aosp10-app-start-cold.svg)

图中虚线部分是创建APP进程和启动Application的流程。

普通Activity启动流程走到`ActivityStackSupervisor.startSpecificActivityLocked()`时，有一个进程相关的判断：如果应用进程存在则去启动Activity，否则去创建进程。冷启动时，App进程还不存在，因此需要去创建进程，其入口便是这里。

```java
void startSpecificActivityLocked(ActivityRecord r, boolean andResume, boolean checkConfig) {
    // Is this activity's application already running?
    final WindowProcessController wpc =
            mService.getProcessController(r.processName, r.info.applicationInfo.uid);
    boolean knownToBeDead = false;
	// APP进程存在时，启动Activity
    if (wpc != null && wpc.hasThread()) {
        // 启动 Activity
        realStartActivityLocked(r, wpc, andResume, checkConfig);
        return;
    }
    // APP进程不存在时，创建进程
    // Message 里设置了一个回调，即 ActivityManagerInternal.startProcess
    final Message msg = PooledLambda.obtainMessage(
            ActivityManagerInternal::startProcess, mService.mAmInternal, r.processName,
            r.info.applicationInfo, knownToBeDead, "activity", r.intent.getComponent());
		// 发送消息到"android.display"线程处理
    mService.mH.sendMessage(msg);
}
```

APP的冷启动过程主涉及到四个进程：Launcher进程（启动APP的进程）、SystemServer进程、APP进程、Zygote进程；三个重要工作：启动进程、启动Application、启动Activity。

整个流程简述为：

- Launcher进程请求SystemServer进程创建Activity
- SystemServer进程解析Intent等，并检查目标APP进程是否存在，不存在则去创建进程。
- SystemServer进程通过Socket方式请求Zygote进程创建APP进程，返回进程号等信息。
- App进程入口为`ActivityThread.main()`方法，执行此方法，初始化进程并请求SystemServer绑定APP进程。绑定进程的工作包括，一，通知APP进程去启动Application；二，通知APP进程去启动Activity。

<aside> 💡 这里说的启动包含创建实例以及执行回调方法等操作。

## 方法说明

与普通Activity启动流程重复的方法不再进行说明

### `ProcessList.startProcessLocked()`

指定进程执行入口为`android.app.ActivityThread.main()`方法

```java
// 设置进程启动相关参数
boolean startProcessLocked(ProcessRecord app, HostingRecord hostingRecord ...) {
    // 进程执行入口
    final String entryPoint = "android.app.ActivityThread";
    return startProcessLocked(hostingRecord, entryPoint, app, uid, gids,
                              runtimeFlags, mountExternal, seInfo, requiredAbi, 
                              instructionSet, invokeWith,startTime);
}
```

### `ZygoteProcess.startViaZygote()`

设置zygote启动进程的参数

### `ZygoteProcess.attemptZygoteSendArgsAndGetResult()`

发送数据以启动进程，获取进程启动后的进程号等

```java
// 发送数据以启动进程，获取进程启动后的进程号等
private Process.ProcessStartResult attemptZygoteSendArgsAndGetResult(
        ZygoteState zygoteState, String msgStr) throws ZygoteStartFailedEx {
    final BufferedWriter zygoteWriter = zygoteState.mZygoteOutputWriter;
    final DataInputStream zygoteInputStream = zygoteState.mZygoteInputStream;
	// 发消息给Zygote进程，各种参数
    zygoteWriter.write(msgStr);
    zygoteWriter.flush();
    
	// 启动进程中...最终会执行ActivityThread.main方法
    
	// 获取启动进程后结果，进程号等
    Process.ProcessStartResult result = new Process.ProcessStartResult();
    result.pid = zygoteInputStream.readInt();
    result.usingWrapper = zygoteInputStream.readBoolean();
    return result;
}
```

### `ActivityThread.main()`

进程入口方法，初始化主线程，绑定进程，启动应用

```java
public static void main(String[] args) {
	// 主线程Looper
    Looper.prepareMainLooper();
	// 当前启动中进程的标识符，从ZygoteProcess相关方法传入
    long startSeq = 0;
	ActivityThread thread = new ActivityThread();
	// 绑定进程，false 表示非系统进程，会调用AMS.attachApplication()
    thread.attach(false, startSeq);
	// 主线程绑定的Handler
    sMainThreadHandler = thread.getHandler();
	// 防止线程结束，等待如Application.onCreate()、Activity.onCreate()等周期回调方法
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

### `ActivityManagerService.attachApplicationLocked()`

 启动Application、启动Activity

```java
// 获取并设置Application Record，启动Application，启动Activity等
private boolean attachApplicationLocked(@NonNull IApplicationThread thread,int pid, int callingUid, long startSeq) {
    // zygote启动进程返回结果后，pid会被保存到mPidsSelfLocked中
    // 见 ProcessList.startProcessLocked 方法中执行的
    // ProcessList.handleProcessStartedLocked，启动进程后更新状态
    // 有概率发生当前方法先于handleProcessStartedLocked执行，则app=null
    ProcessRecord app = mPidsSelfLocked.get(pid);
    // 设置app参数
    ...
    // ipc 启动Application, 最终在APP进程调用ActivityThread.handleBindApplication()
    thread.bindApplication(processName, appInfo, providers, ...);
    // 去启动Activity。最终调用ActivityStackSupervisor.realStartActivityLocked()
    didSomething = mAtmInternal.attachApplication(app.getWindowProcessController());
    return true;
}
```

### `ActivityThread.handleBindApplication()`

设置APP数据，创建Application实例，调用`Application.onCreate()`方法

```java
private void handleBindApplication(AppBindData data) {
    // 设置APP名称
    Process.setArgV0(data.processName);
    android.ddm.DdmHandleAppName.setAppName(data.processName, UserHandle.myUserId());
    VMRuntime.setProcessPackageName(data.appInfo.packageName);
    // 设置应用缓存数据目录
    VMRuntime.setProcessDataDirectory(data.appInfo.dataDir);
    // 设置时区和本地化
    TimeZone.setDefault(null);
    LocaleList.setDefault(data.config.getLocales());
    // 创建应用包的数据
    data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
    if (agent != null) {
        handleAttachAgent(agent, data.info);
    }
    // 设置屏幕分辨率(用于兼容)、时间格式、网络代理
    ...
    // 判断并获取或创建 Instrumentation，之后会用来创建Application
    ...
    // 创建Application、重写资源相关的R常量
    Application app = data.info.makeApplication(data.restrictedBackupMode, null);
	mInstrumentation.onCreate(data.instrumentationArgs);
	// 执行Application.onCreate回调
	mInstrumentation.callApplicationOnCreate(app);
	// 加载字体资源
	...
}
```

# 相关类

```bash
frameworks/base/services/core
    com.android.server.wm.ActivityTaskManagerService 
    com.android.server.wm.ActivityStarter
    com.android.server.wm.RootActivityContainer
    com.android.server.wm.ActivityStack
    com.android.server.wm.ActivityStackSupervisor
    com.android.server.wm.ClientLifecycleManager
    com.android.server.am.ActivityManagerService
    com.android.server.am.ActivityManagerService.LocalService
    com.android.server.am.ProcessList
frameworks/base/core
    android.app.Activity
    android.app.Instrumentation
    android.app.ActivityThread
    android.app.ActivityThread.ApplicationThread
    android.app.servertransaction.LaunchActivityItem
    android.app.servertransaction.TransactionExecutor
    android.app.servertransaction.ClientTransaction
    android.os.Process
    android.os.ZygoteProcess
    android.app.LoadedApk
    android.app.AppComponentFactory
    android.app.Application
```

# 参考

- [Android 10 源码](http://aospxref.com/android-10.0.0_r47/)
