---
title: Android服务框架概览
date: 2021-12-02 17:59:47
tags: 
- android
- android系统开发
categories:
- Android系统开发
---

# 说明

Android服务框架包括Java服务框架（Java层）和本地服务框架（C++层），两层通过JNI交互。

代码分析版本：Android 10

# 框架概览

### **Android服务框架结构图**

![](/images/android-service-framwork-structure.svg)

说明：Android服务框架可分为四层

1. 服务层：包含特定功能函数的服务层。IFooService规定服务提供的函数，继承IInterface；FooManager引用IFooService；FooService继承Binder和IFooService。
2. RPC层：客户端生成RPC代码和数据，服务端根据RPC代码查找对应函数并传递RPC数据。IFooService.Stub.Proxy持有BinderProxy引用，BinderProxy由用户检索服务时获得。IFooService.Stub为抽象类，由FooService或其成员实现。
3. IPC层：将RPC代码和数据封装为Binder IPC数据，并传递给Binder Driver。
4. Binder Driver层：根据Binder IPC数据查找指定服务并返回。

### **Android系统服务框架各类相互作用**

![](/images/android-service-framwork-class-work.svg)

1. 注册服务。FooService发起服务注册请求，Java系统服务一般在SystemServer.run方法里实例化并注册。在Binder驱动里将服务注册为Binder节点，ContextManager持有该节点句柄。
2. 检索服务。用户使用Context.getSystemService获取FooService，实际上是通过IPC从ContextManager 获取FooService，返回的是FooService的句柄，但客户端不能直接使用FooService，最终返回的是通过FooService生成的代理类(BinderProxy)的包装类FooManager。注意这个过程中会生成客户端的BinderProxy实例以及本地BpBinder实例，FooManager间接持有BinderProxy引用，BinderProxy间接持有本地BpBinder指针，这些对象在客户端使用服务时会涉及到。
3. 使用服务。FooManager调用foo()函数，实际上调用代理类BinderProxy.transact()函数，后者通过JNI与本地BpBinder交互，并将数据传给Binder驱动进行IPC通信，通过IPCThreadState会调用FooService对应的本地BBinder实例，实际为其实现类JavaBBinder，后者会调用Java层的FooService(继承IFooService.Stub，后者继承Binder)的execTransact()方法，最终根据RPC代码查找到指定函数并执行。

**注意：以上举例适用于Java服务，本地服务类似，需要将最上层的Java代码改为C++代码，细节待确定 TODO**

### 主要类和文件

```bash
java
/frameworks/base/core/java/android/app/AlarmManager.java
/frameworks/base/core/java/android/app/IAlarmManager.aidl
- 包含IAlarmManager.Stub和IAlarmManager.Stub.Proxy类
/frameworks/base/core/java/android/os/ServiceManagerNative.java
- 包含ServiceManagerProxy类
/frameworks/base/core/java/android/os/BinderProxy.java
/frameworks/base/core/java/android/os/Binder.java

c/c++
/frameworks/base/core/jni/android_util_Binder.cpp
- 包含JavaBBinder类
/frameworks/native/libs/binder/BpBinder.cpp
/frameworks/native/libs/binder/Binder.cpp  
- 该文件包含BBinder类
/frameworks/native/libs/binder/IPCThreadState.cpp
/frameworks/native/cmds/servicemanager/service_manager.c
```

# 示例分析-AlarmManagerService

以应用层使用Java系统服务AlarmManagerService为例，基于代码分析服务的注册、检索和使用三个流程。

```java
// 1. SystemSever启动时，注册服务。
mSystemServiceManager.startService(new AlarmManagerService(context));

// SampleActivity.java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // 2. 检索服务。获取AlarmManager包装类，持有服务代理的引用
        AlarmManager am = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        // 3. 使用服务。通过包装类调用服务的方法
        am.setTime(1000L);
    }
}
```

### 注册服务

Java系统服务在SystemServer进程启动时开始实例化并注册。首先调用SystemServer.main()方法，该方法内部会实例化SystemServer并调用其run()方法，run()方法中接着调用startOtherServices()方法，该方法中会执行AlarmManagerService的服务实例化和注册操作。

SystemServer.run()方法最后会调用Looper.loop()方法阻止线程退出，SystemServer实例会常住内存且持有SystemServiceManager实例引用，SystemServiceManager又持有系统服务集合，因此各个系统服务实例会常住系统内存，以供客户端调用。

```java
// 前置流程：SystemServer.main() -> SystemServer.run() -> startOtherServices()
private void startOtherServices() { 
    // SystemServiceManager.startService内部调用流程:
    // -> SystemService.onStart(), 实际调用的是子类AlarmManagerService.onStart()
    // -> SystemService.publishBinderService()
    // -> ServiceManager.addService() 
    //    先通过getIServiceManager构造ServiceManagerProxy，再调用其addService()方法
    // -> ServiceManagerProxy.addService(), 方法内的mRemote为BinderProxy, 详情见(1)
    // 进入IPC流程:
    // -> BinderProxy.transact() -> native transactNative()
    // -> android_os_BinderProxy_transact(), android_util_Binder.cpp
    // -> BpBinder.transact(), BpBinder.cpp
    // -> IPCThreadState.transact(mHandle, code, data, ...), IPCThreadState.cpp
    // -> ...Binder Driver IPC... 最终进入ContextManager服务进程中...
    // -> IPCThreadState::executeCommand(), IPCThreadState.cpp
    // -> BBinder.transact() -> onTransact() -> ...??, Binder.cpp (2)
    // 注册操作的服务端实现:
    // -> binder_loop(), service_manager.c, loop()方法在main()方法里被调用并等待消息
    // -> binder_parse(), service_manager.c, 进入BR_TRANSACTION分支
    //    func在service_manager.c:main()方法里预设为svcmgr_handler
    // -> svcmgr_handler(bs, &txn, &msg, &reply), service_manager.c
    //    进入分支：case SVC_MGR_ADD_SERVICE
    // -> do_add_service(), service_manager.c, 保存服务名称、节点handle等数据到svclist
    mSystemServiceManager.startService(new AlarmManagerService(context));
}
```

(1) mRemote初始化为BinderProxy流程

mRemote被声明为IBinder，在ServiceManagerProxy构造方法内进行初始化，而ServiceManagerProxy是通过ServiceManager.getIServiceManager()内调用静态方法ServiceManagerNative.asInterface(IBinder b)进行构造的。

ServiceManagerNative.asInterface(IBinder b)方法接收IBinder类型的参数，并最终赋值给ServiceManagerProxy里的mRemote变量。IBinder类型的参数由BinderInternal.getContextObject()通过JNI方式从本地获取。

```java
private static IServiceManager getIServiceManager() {
    // BinderInternal.getContextObject() 返回的IBinder即为mRemote的值
    sServiceManager = ServiceManagerNative
        .asInterface(Binder.allowBlocking(BinderInternal.getContextObject()));
    return sServiceManager;
}

// 通过JNI方式实例化BinderProxy流程:
// -> android_util_Binder.cpp: android_os_BinderInternal_getContextObject()
// -> android_util_Binder.cpp: javaObjectForIBinder()
// -> android_util_Binder.cpp: jobject object = env->CallStaticObjectMethod(
//        gBinderProxyOffsets.mClass,gBinderProxyOffsets.mGetInstance, ...);
//    即最终会待用Java层静态方法BinderProxy.getInstance()
// -> BinderProxy.getInstance()
// -> new BinderProxy(nativeData) 最终通过从本地服务获取的数据构造BinderProxy
public static final native IBinder getContextObject()
```

**(2) 如何从BBinder进入service_manager.c的binder_loop()方法中? TODO**

### 检索服务

调用Context.getSystemService()方法，具体是调用ContextImpl.getSystemService()方法。

```java
// ContextImpl.java 通过name获取ServiceFetcher
public Object getSystemService(String name) {
    // 内部调用流程: 
    // -> ServiceFetcher.getService(),  最初会调用 createService()
    // -> CachedServiceFetcher.createService(),
    //    CachedServiceFetcher是ServiceFetcher实现抽象类, 它是在
    //    SystemServiceRegistry中通过静态代码块调用静态方法registerService()时实例化的
    return SystemServiceRegistry.getSystemService(this, name);
}

// SystemServiceRegistry.java
static {
    // 注册服务Fetcher, 实例化CachedServiceFetcher
    registerService(Context.ALARM_SERVICE, AlarmManager.class,
            new CachedServiceFetcher<AlarmManager>() {
        @Override
        public AlarmManager createService(ContextImpl ctx) {
            IBinder b = ServiceManager.getServiceOrThrow(Context.ALARM_SERVICE);
            IAlarmManager service = IAlarmManager.Stub.asInterface(b);
            return new AlarmManager(service, ctx);
        }});
}

// CachedServiceFetcher匿名子类
public AlarmManager createService(ContextImpl ctx) throws ServiceNotFoundException {
    // 说明：此处获取的IBinder为BinderProxy
    // ServiceManager.getServiceOrThrow内部调用流程：
    // -> IBinder binder = ServiceManager.getService()
    // -> IBinder binder = ServiceManager.rawGetService()
    // -> IBinder binder = getIServiceManager().getService(name)
    // -> ServiceManagerProxy.getService(),  方法内mRemote为BinderProxy
    // 进入IPC流程:
    // -> BinderProxy.transact() -> native transactNative()
    // -> BpBinder.transact()...ipc...BBinder.transact()..., 
    // 检索操作的服务端实现:
    // -> service_manager.c:do_find_service()并返回handle值, 结果存于reply
    // 待BinderProxy.transact()调用完毕后，会接着调用
    // -> IBinder binder = reply.readStrongBinder(), 此处会创建BinderProxy对象
    //    及其native对象BpBinder，之后使用服务时会用到
    IBinder b = ServiceManager.getServiceOrThrow(Context.ALARM_SERVICE);

    // 此处的service为IAlarmManager.Stub.Proxy
    // -> new android.app.IAlarmManager.Stub.Proxy(obj);
    // 即基于Service构造BinderProxy, 基于BinderProxy构造Stub.Proxy, 
    // 基于Stub.Proxy构造Manager
    IAlarmManager service = IAlarmManager.Stub.asInterface(b);
    return new AlarmManager(service, ctx);
});
```

### 使用服务

调用AlarmManager.setTime()

```java
public void setTime(long millis) {
    // 由检索服务流程知mService为Stub.Proxy对象
    // mService.setTime()内部调用流程如下：
    // -> IAlarmManager.Stub.Proxy.setTime()
    //    由检索服务流程知Stub.Proxy里的mRemote为BinderProxy对象
    // 进入IPC流程:
    // -> BinderProxy.transact(Stub.TRANSACTION_setTime, _data, _reply, 0)
    // -> native transactNative()
    // -> android_os_BinderProxy_transact(), android_util_Binder.cpp
    // -> BpBinder.transact(), BpBinder.cpp
    // -> IPCThreadState.transact(mHandle, code, data, ...), IPCThreadState.cpp
    // -> ...Binder Driver IPC... 
    // -> IPCThreadState::executeCommand(), IPCThreadState.cpp
    // -> BBinder.transact(), Binder.cpp
    // -> JavaBBinder(子类).onTransact(), android_util_Binder.cpp (1)
    // -> Binder.execTransact()
    // 服务方法的实现：
    // -> IAlarmManager.Stub(子类).onTransact(), Stub为Binder的实现类
    // -> AlarmManagerService.mService(子类).setTime(), mService为Stub的实现类
    mService.setTime(millis);
}
```

(1) JavaBBinder本地代码调用Java代码 Binder.execTransact()

```java
// android_util_Binder.cpp
status_t onTransact(){
    jboolean res = env->CallBooleanMethod(mObject, 
            gBinderOffsets.mExecTransact,  // 调用 Binder.execTransact()
            code, reinterpret_cast<jlong>(&data), 
            reinterpret_cast<jlong>(reply), flags);
}
```

# 参考

- 金泰廷等著《Android框架解密》，人民邮电出版社，2012。该书分析的代码较老，但框架机制基本不变，非常好的书籍。
- 源码查看网站：http://aospxref.com/。基于OpenGrok的源码查看服务网站，速度很快，主要用来查看C/C++代码，Java代码可下载到本地查看。

