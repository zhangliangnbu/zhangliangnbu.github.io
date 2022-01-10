---
title: Android应用开发基础四：Service、Broadcast和ContentProvider
date: 2018-08-10 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

# Service

使用服务两种方式

- 启动服务。外部通过调用`startService()`启动服务，后者回调方法 `onStartCommand()`。之后服务在后台无限期运行，直到调用`stopSelf()`、`stopService()`停止。
- 绑定服务。外部通过调用 `bindService()` 绑定服务，后者回调方法  `onBind()`。之后一直运行，直到取消绑定为止。

这两种方式可以同时使用，一般单独调用。

`stopSelf()` vs `stopSelf(int startId)` ：前者立即停止服务；后者在最近的startId与参数相匹配时才停止，防止有任务未完成时而服务停止的情况。

<!-- more -->

## 服务和线程的选择

如果在用户与应用交互期间，需要在主线程外执行操作，则应创建新线程，并虑使用 AsyncTask 或 HandlerThread，而非传统的 Thread 类。

服务默认会在应用的主线程中运行，因此，如果服务执行的是密集型或阻止性操作，则您仍应在服务内创建新线程。

## Service 和 IntentService

- Service。服务的基类。由于服务默认使用应用的主线程，继承此类时需要手动创建新线程。
- IntentService。Service 的子类，内部实现了工作线程，可逐一处理任务。不要求同时处理多个任务时可选择此类。实现 onHandleIntent()，该方法会接收每个启动请求的 Intent，以便执行后台工作。

## 前台服务

前台服务是用户主动意识到的一种服务，因此在内存不足时，系统也不会考虑将其终止。前台服务必须为状态栏提供通知，将其放在运行中的标题下方。

调用`startForeground(ONGOING_NOTIFICATION_ID, Notification);` 让服务在前台运行。

> 💡 注意：Android 9（API 级别 28）或更高版本并使用前台服务，则其必须请求 FOREGROUND_SERVICE 权限。这是一种普通权限，因此，系统会自动为请求权限的应用授予此权限。

## 生命周期

![android-service-lifecycle](/images/android-service-lifecycle.png)

## 通信

**本地通信（非跨进程通信）**

组件通过实现 Binder 类来创建接口，直接持有Service的引用，进而直接调用Service方法。

**跨进程通信**

方法一，使用Messenger，会将所有服务调用加入到队列，一次只能处理一个调用。Service中定义Messenger，组件通过Binder持有Messenger，发送消息到Service中处理。

```java
// Service 核心代码
static class IncomingHandler extends Handler {
    private Context applicationContext;
    IncomingHandler(Context context) {
        applicationContext = context.getApplicationContext();
    }
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_SAY_HELLO:
                Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show();
                break;
            default:
                super.handleMessage(msg);
        }
    }
}

@Override
public IBinder onBind(Intent intent) {
    mMessenger = new Messenger(new IncomingHandler(this));
    return mMessenger.getBinder();
}

// Activity 核心代码
private ServiceConnection mConnection = new ServiceConnection() {
    public void onServiceConnected(ComponentName className, IBinder service) {
        mService = new Messenger(service);
        bound = true;
    }
    public void onServiceDisconnected(ComponentName className) {
        mService = null;
        bound = false;
    }
};

@Override
protected void onStart() {
    super.onStart();
    bindService(new Intent(this, MessengerService.class), mConnection,
        Context.BIND_AUTO_CREATE);
}

@Override
protected void onStop() {
    super.onStop();
    // Unbind from the service
    if (bound) {
        unbindService(mConnection);
        bound = false;
    }
}

public void sayHello(View v) {
    if (!bound) return;
    Message msg = Message.obtain(null, MessengerService.MSG_SAY_HELLO, 0, 0);
    try {
        mService.send(msg);
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```

方法二：使用aidl，利用多线程可同时处理多个调用。

定义.aidl文件，Android SDK 工具会生成基于该 .aidl 文件的 IBinder 接口，并将其保存到项目的 gen/ 目录中。服务必须视情况实现 IBinder 接口。然后，客户端应用便可绑定到该服务，并调用 IBinder 中的方法来执行 IPC。

如要使用 AIDL 创建绑定服务，请执行以下步骤：

1. [创建 .aidl 文件](https://developer.android.google.cn/guide/components/aidl?hl=zh-cn#Create)。此文件定义带有方法签名的编程接口。
2. [实现接口](https://developer.android.google.cn/guide/components/aidl?hl=zh-cn#ImplementTheInterface)。Android SDK 工具会基于您的 `.aidl` 文件，使用 Java 编程语言生成接口。此接口拥有一个名为 `Stub` 的内部抽象类，用于扩展 `Binder` 类并实现 AIDL 接口中的方法。您必须扩展 `Stub` 类并实现这些方法。
3. [向客户端公开接口](https://developer.android.google.cn/guide/components/aidl?hl=zh-cn#ExposeTheInterface)。实现 `Service` 并重写 `onBind()`，从而返回 `Stub` 类的实现。

```java
// IBookManager 为aidl借口
// Service 核心代码
private Binder binder = new IBookManager.Stub() {
    @Override
    public List<Book> getBookList() throws RemoteException {
        return bookList;
    }
    ...
};

@Override
public IBinder onBind(Intent intent) {
	return binder;
}

// Activity 核心代码
IBookManager iBookManager;
private ServiceConnection connection = new ServiceConnection() {
	@Override
	public void onServiceConnected(ComponentName name, IBinder service) {
		iBookManager = IBookManager.Stub.asInterface(service);
		List<Book> bookList = iBookManager.getBookList();
	}
};
```

# BroadcastReceiver

广播可用于应用内和应用间IPC通信。使用流程：注册广播接收者，发送广播。

注册广播接收者的两种方式：

- 静态：在Manifest清单中声明。Android 8及以上有限制，只允许少数的系统级Action。
- 动态：在 Application 或 Activity 等组件中声明。

发送广播的三种方式：

- `sendOrderedBroadcast(Intent, String)` 顺序发送，可中止。
- `sendBroadcast(Intent)` 随机通知发送。
- `LocalBroadcastManager.sendBroadcast()` 仅限本地应用发送。

## 广播接收的限制

三种方式限制其他应用接收你的广播：

- 您可以在发送广播时指定权限。
- 在 Android 4.0 及更高版本中，您可以在发送广播时使用 `setPackage(String)` 指定[软件包](https://developer.android.google.cn/guide/topics/manifest/manifest-element?hl=zh-cn#package)。系统会将广播限定到与该软件包匹配的一组应用。
- 您可以使用 `LocalBroadcastManager` 发送本地广播。

三种方式限制你的应用可以接收的广播：

- 您可以在注册广播接收器时指定权限。
- 对于清单声明的接收器，您可以在清单中将 `android:exported`属性设置为“false”。这样一来，接收器就不会接收来自应用外部的广播。
- 您可以使用 `LocalBroadcastManager` 限制您的应用只接收本地广播。

# ContentProvider 

使其他应用安全地访问和修改您的应用数据。

ContentProvider 实例详解。**TODO**
