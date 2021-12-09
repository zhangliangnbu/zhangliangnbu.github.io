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

参考闹钟服务编写一个HelloWorldService服务，功能为打印字符串"Hello, World!"，然后基于编译后的系统，编写APP使用该服务。

```bash
闹钟服务相关接口和类
- /frameworks/base/core/java/android/app/IAlarmManager.aidl  接口定义
- out/soong/.intermediates/frameworks/base/framemwork/android_common/
  gen/aidl/frameworks/base/core/java/android/app/IAlarmManager.java  自动生成的接口以及Stub、Proxy
- /frameworks/base/services/core/java/com/android/server/AlarmManagerService.java  服务实现
- /frameworks/base/core/java/android/app/AlarmManager.java  客户端使用的服务代理

```

# 编写Framwork代码

### 编写和配置 IHelloWorld.aidl

**编写接口**。目录：`/frameworks/base/core/java/android/app`

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

待编译时，会自动生成对应的java接口和Stub、Proxy类，不用手动处理，生成的代码位于：out/soong/.intermediates/frameworks/base/framemwork/android_common/gen/aidl/frameworks/base/core/java/android/app

**配置接口**。在 `/framwork/base/Android.bp`中配置接口，在`srcs`数组里增加`HelloWorld.aidl`的路径地址即可：

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

### 编写和注册HelloWorldService

**配置常量**。在编写服务之前需要在Context.java里配置常量，参照ALARM_SERVICE，进行如下配置。

```java
// 定义常量
public static final String HELLO_WORLD_SERVICE = "hello_world";

// 将常量添加到注解值列表中
@StringDef(suffix = { "_SERVICE" }, value = {
	...
    HELLO_WORLD_SERVICE,
})
@Retention(RetentionPolicy.SOURCE)
public @interface ServiceName {}
```

**编写服务**。参考[AlarmManagerService](http://aospxref.com/android-10.0.0_r47/xref/frameworks/base/services/core/java/com/android/server/AlarmManagerService.java#mService) ，目录: `/frameworks/base/services/core/java/com/android/server`

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
        // 之前已经在Context中定义过常量
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

**注册HelloWorldService服务**。在SystemServer.java里向服务管理者注册服务。

目录：` /frameworks/base/services/java/com/android/server/SystemServer.java`

```java
private void startOtherServices() {
    ...
    traceBeginAndSlog("StartAlarmManagerService");
    mSystemServiceManager.startService(new AlarmManagerService(context));
    traceEnd();

    traceBeginAndSlog("StartHelloWorldService");
    // 此处实例化并注册HelloWorldService
    mSystemServiceManager.startService(new HelloWorldService(context));
    traceEnd();
    ...
}
```

### 编写和注册HelloWorldManager

**编写HelloWorldManager**。参考AlarmManager编写客户端的HelloWorldManager，基于此应用开发者可以使用HelloWorldService。

目录：`/frameworks/base/core/java/android/app`

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

**注册ServiceFetcher**。要利用`ContextImpl.getSystemService(String name)` 获取服务的Manager，需要首先在`SystemServiceRegistry`里注册相应的`ServiceFetcher`：

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

### 配置SELinux权限

在system/sepolicy 目录下的service.te 和 service_contexts 中配置 HelloWorldService的权限。**TODO，待详细了解**

# 编译和烧录

编译Android系统，并烧录到实机设备或虚拟设备中：

```java
$ m update-api
```

# 编写APP并验证

将系统编译出的class.jar导入到工程中，编写APP，获取服务并调用服务的方法。

在主Activity的onCreate()里加入如下两代码：

```java
protected void onCreate(Bundle savedInstanceState) {
    ...
    HelloWorldManager hwm = (HelloWorldManager)getSystemService(Context.HELLO_WORLD);
    hwm.printHW();
}
```

运行APP并查看Log



# 参考

- 金泰廷等著《Android框架解密》，人民邮电出版社，2012。该书分析的代码较老，但框架机制基本不变，非常好的书籍。
- 源码查看网站：http://aospxref.com/。基于OpenGrok的源码查看服务网站，速度很快，主要用来查看C/C++代码，Java代码可下载到本地查看。
- [AOSP官方-SELinux](https://source.android.google.cn/security/selinux?hl=zh-cn)：详细介绍了SELinux
- [android 10 添加系统服务步骤](https://blog.csdn.net/a546036242/article/details/118221349)：主要参考里面添加SELinux权限那块。

