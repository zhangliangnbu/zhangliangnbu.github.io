---
title: Android中的Handler机制
date: 2018-08-14 15:14:22
updated: 2022-01-10 17:52:48
tags:
- android
categories:
- Android开发
---

# 简介

Android中涉及线程操作的类或机制主要有：

- Thread。线程操作的最基本的类。
- ThreadPoolExecutor。基于线程池，处理并发多线程操作。
- Handler机制：Handler、Looper、MessageQueue等。Android中多线程处理的基本机制。
- HandlerThread。Thread的子类，维护了一个Looper，Handler机制的辅助类。
- AsyncTask。基于Handler机制。
- IntentService。基于Handler机制。

本文主要介绍Android中的Handler机制。

# Handler机制

Handler 发消息到MessageQueue中，Looper 循环从MessageQueue中获取消息并消费（执行Handler的回调方法）。

以线程B向线程A发消息为例，从时间顺序上说明如下：

- 在线程A中，创建Looper，通过sThreadLocal与当前绑定，启动Looper的消息循环，线程处于活跃状态；创建Handler，在Looper进行消息循环之前创建，Handler会与当前线程的Looper建立联系。
- 在线程B中，使用上面的Handler发送消息。
- 在线程A中，Looper循环消息获取消息，由于消息持有Handler引用，最终会执行Handler的回调方法。

<!-- more -->

代码示例：

```java
private  Handler handler;
public void testHandler() {
    // 创建线程A 、Looper、Handler
    Thread threadA = new Thread(() -> {
        Looper.prepare();
        // 创建 Handler。
        handler = new Handler() {
            @Override
            public void handleMessage(@NonNull Message msg) {
                super.handleMessage(msg);
                Log.d("handleMessage", "msg what == " + msg.what);
            }
        };
        Looper.loop();
    });
    threadA.start();

    // 创建线程B，并利用Handler 发送消息
    Thread threadB = new Thread(() -> {
        // 防止ThreadA没有执行完而导致handler为空
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        handler.sendEmptyMessage(111);
    });
    threadB.start();
}
```

注意：创建Handler有多种方式，以绑定线程A的Looper为例：

- 在`Looper.prepare()`和`Looper.loop()` 之间创建，`handler = new Handler()`。Android中主线程的第一个Handler就是如此创建。

- 在线程A的某个Handler的回调消息中创建，`handler = new Handler()`。一般开发时在Activity里创建的Handler就属于这种情况。

- 在其他线程中直接利用与线程A绑定的Looper进行创建，`handler = new Handler(looper)`。IntentService里使用HandlerThread，属于这种情况。

  

# 如何确保一个线程只有一个Looper？

机制：每个线程对象持有一个`ThreadLocalMap`成员，`ThreadLocalMap`以`ThreadLocal`为key，以实际目标值为value。如果以一个静态`ThreadLocal`对象为key，则每个线程里的`ThreadLocalMap`都会持有相同的key却互不并不影响，但在线程内部，由于`ThreadLocalMap`里的key唯一，则可以确保只有一个值。在Looper创建时则是判断当前线程是否有Looper，有则抛出异常，从而确保唯一。

相关代码：

```java
// Thread.java
ThreadLocalMap threadLocals;

// ThreadLocalMap
private ThreadLocal.ThreadLocalMap.Entry[] table;
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k); // key为弱引用，避免内存泄漏
        this.value = v;
    }
}

// ThreadLocal.java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); // t.threadLocals
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

// Looper.java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>(); // 静态成员对象，确保唯一
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

```

# HandlerThread

`Thread`的子类，维护了一个`Looper`，在`run()`中创建`Looper`实例、与线程绑定，开启消息循环。

```java
@Override
public void run() {
    mTid = Process.myTid();
		// 
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

一般使用：

```java
// 创建线程A 、Looper
HandlerThread threadA = new HandlerThread("handler thread a");
threadA.start();

// 创建Handler.
// threadA.getLooper() 中使用了wait()，保证looper不为空
handler = new Handler(threadA.getLooper()) {
    @Override
    public void handleMessage(@NonNull Message msg) {
        super.handleMessage(msg);
        Log.d("handleMessage", "msg what == " + msg.what);
    }
};

// 创建线程B，并利用Handler 发送消息
Thread threadB = new Thread(() -> {
    handler.sendEmptyMessage(111);
});
threadB.start();
```

# AsyncTask

基于Handler机制。在工作线程执行任务`doInBackground`，通过Handler发送消息到ui线程，更新进度`onProgressUpdate`和结果`onPostExecute`。

```java
public static class MyAsyncTask extends AsyncTask<String, Integer, String> {
    @Override
    protected void onPreExecute() {
        // on ui thread
        super.onPreExecute();
        Log.d("MyAsyncTask", "onPreExecute");
    }

    @Override
    protected String doInBackground(String... strings) {
        // on work thread
        publishProgress(0);
        Log.d("MyAsyncTask", "doInBackground ==" + Arrays.toString(strings));
        publishProgress(100);
        return Arrays.toString(strings);
    }

    @Override
    protected void onPostExecute(String s) {
        // on ui thread
        super.onPostExecute(s);
        Log.d("MyAsyncTask", "onPostExecute ==" + s);
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        // on ui thread
        super.onProgressUpdate(values);
        Log.d("MyAsyncTask", "onProgressUpdate ==" + Arrays.toString(values));
    }
}

public void testAsyncTask() {
    MyAsyncTask asyncTask = new MyAsyncTask();
    asyncTask.execute("a", "b", "c");
}
```

# IntentService

基于Handler，相关代码如下：

```java
@Override
public void onCreate() {
    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```
