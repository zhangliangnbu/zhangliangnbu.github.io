---
title: 编写Android的Java系统服务
date: 2021-12-03 17:55:19
tags: 
- android
- android系统开发
categories:
- Android开发
---

# 说明

参考闹钟服务编写一个HelloWorldService服务，基于编译后的系统，编写APP调用该服务。

```bash
闹钟服务
- /frameworks/base/core/java/android/app/IAlarmManager.aidl 接口定义
- AlarmManagerService.java
```



### 编写接口 IHelloWorld.aidl

```java
package android.app
/**
 * System private API for talking with the alarm manager service.
 *
 * {@hide}
 */
interface IHelloWorld {
    void printHello();
}
```

### 生成Stub

```java
package android.app;
// Declare any non-default types here with import statements

public interface IHelloWorld extends android.os.IInterface
{
  /** Default implementation for IHelloWorld. */
  public static class Default implements IHelloWorld
  {
    /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
    @Override public void printHello() throws android.os.RemoteException
    {
    }
    @Override
    public android.os.IBinder asBinder() {
      return null;
    }
  }
  /** Local-side IPC implementation stub class. */
  public static abstract class Stub extends android.os.Binder implements IHelloWorld
  {
    private static final String DESCRIPTOR = "com.example.helloworldservicedemo.IHelloWorld";
    /** Construct the stub at attach it to the interface. */
    public Stub()
    {
      this.attachInterface(this, DESCRIPTOR);
    }
    /**
     * Cast an IBinder object into an com.example.helloworldservicedemo.IHelloWorld interface,
     * generating a proxy if needed.
     */
    public static IHelloWorld asInterface(android.os.IBinder obj)
    {
      if ((obj==null)) {
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&&(iin instanceof IHelloWorld))) {
        return ((IHelloWorld)iin);
      }
      return new Proxy(obj);
    }
    @Override public android.os.IBinder asBinder()
    {
      return this;
    }
    @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
    {
      String descriptor = DESCRIPTOR;
      switch (code)
      {
        case INTERFACE_TRANSACTION:
        {
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_printHello:
        {
          data.enforceInterface(descriptor);
          this.printHello();
          reply.writeNoException();
          return true;
        }
        default:
        {
          return super.onTransact(code, data, reply, flags);
        }
      }
    }
    private static class Proxy implements IHelloWorld
    {
      private android.os.IBinder mRemote;
      Proxy(android.os.IBinder remote)
      {
        mRemote = remote;
      }
      @Override public android.os.IBinder asBinder()
      {
        return mRemote;
      }
      public String getInterfaceDescriptor()
      {
        return DESCRIPTOR;
      }
      /**
           * Demonstrates some basic types that you can use as parameters
           * and return values in AIDL.
           */
      @Override public void printHello() throws android.os.RemoteException
      {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
          _data.writeInterfaceToken(DESCRIPTOR);
          boolean _status = mRemote.transact(Stub.TRANSACTION_printHello, _data, _reply, 0);
          if (!_status && getDefaultImpl() != null) {
            getDefaultImpl().printHello();
            return;
          }
          _reply.readException();
        }
        finally {
          _reply.recycle();
          _data.recycle();
        }
      }
      public static IHelloWorld sDefaultImpl;
    }
    static final int TRANSACTION_printHello = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    public static boolean setDefaultImpl(IHelloWorld impl) {
      // Only one user of this interface can use this function
      // at a time. This is a heuristic to detect if two different
      // users in the same process use this function.
      if (Proxy.sDefaultImpl != null) {
        throw new IllegalStateException("setDefaultImpl() called twice");
      }
      if (impl != null) {
        Proxy.sDefaultImpl = impl;
        return true;
      }
      return false;
    }
    public static IHelloWorld getDefaultImpl() {
      return Proxy.sDefaultImpl;
    }
  }
  /**
       * Demonstrates some basic types that you can use as parameters
       * and return values in AIDL.
       */
  public void printHello() throws android.os.RemoteException;
}
```

### 配置接口

在 framwork/base/Android.bp中配置接口

```java
java_defaults {
    name: "framework-defaults",
    installable: true,

    srcs: [
        ...
        "core/java/android/app/IAlarmManager.aidl",
				"core/java/android/app/HelloWorld.aidl",
				...
		],
}
```

### 实现HelloWorldService服务

参考AlarmManagerService：http://aospxref.com/android-10.0.0_r47/xref/frameworks/base/services/core/java/com/android/server/AlarmManagerService.java#mService

```java
package com.android.server;

import android.app.IHelloWorld;
import android.content.Context;
import android.os.IBinder;
import android.os.RemoteException;
import android.util.Slog;

public class HelloWorldService extends SystemService{
    /**
     * Initializes the system service.
     * <p>
     * Subclasses must define a single argument constructor that accepts the context
     * and passes it to super.
     * </p>
     *
     * @param context The system server context.
     */
    public HelloWorldService(Context context) {
        super(context);
    }

    @Override
    public void onStart() {
				// 先在Context里添加HELLO_WORLD常量
        publishBinderService(Context.HELLO_WORLD, mService);
    }

    private final IBinder mService = new IHelloWorld.Stub() {
        private static final String TAG = "HelloWorldService";
        @Override
        public void printHello() throws RemoteException {
            Slog.i(TAG, "Hello, World!");
        }
    };
}
```

### 注册HelloWorldService服务

在SystemServer.java里向服务管理者注册服务

```java
private void startOtherServices() {
		...
		traceBeginAndSlog("StartAlarmManagerService");
    mSystemServiceManager.startService(new AlarmManagerService(context));
    traceEnd();

		traceBeginAndSlog("StartHelloWorldService");
    mSystemServiceManager.startService(new HelloWorldService(context));
    traceEnd();
		...
}
```

### 编写客户端HelloWorldManager

应用开发者可以通过HelloWorldManager使用服务HelloWorldService

```java
package android.app;

import android.annotation.SystemService;
import android.content.Context;

@SystemService(Context.HELLO_WORLD)
public class HelloWorldManager {
    private static final String TAG = "HelloWorldManager";

    private final IHelloWorld mService;

    HelloWorldManager(IHelloWorld mService) {
        this.mService = mService;
    }

    public void printHello() {
        try {
            mService.printHello();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 获取HelloWorldManager

`ContextImpl.getSystemService(String name)` 用于获取各种服务的Manager，在SystemServiceRegistry里注册相应的ServiceFetcher即可，如下。

```java
final class SystemServiceRegistry {
		static {
				...
				registerService(Context.HELLO_WORLD, HelloWorldManager.class,
		            new CachedServiceFetcher<HelloWorldManager>() {
		        @Override
		        public HelloWorldManager createService(ContextImpl ctx) throws ServiceNotFoundException {
		            IBinder b = ServiceManager.getServiceOrThrow(Context.HELLO_WORLD);
		            IHelloWorld service = IHelloWorld.Stub.asInterface(b);
		            return new HelloWorldManager(service);
		        }});
				...
		}
}
```

### 编译和调试

```java
$ m update-api
```

### APP使用

```java
HelloWorldManager hwm = (HelloWorldManager)getSystemService(Context.HELLO_WORLD);
```
