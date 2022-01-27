---
title: Java基础一：并发概览
date: 2018-09-11 15:14:22
updated: 2022-01-10 17:52:48
tags:
- java
categories:
- Java
---

本文介绍Java里并发相关的内容，包括多线程、线程池、锁等。

当多个线程同时访问对象并要求操作相同资源时，分割了原子操作就有可能出现数据的不一致或数据不完整的情况，为避免这种情况的发生，需要采取同步机制，以确保在某一时刻，方法内只允许有一个线程。

<!--more-->

# 基本概念

并发编程涉及的三个特性：原子性、可见性、有序性

- 原子性：要么全都做，要么全都不做。
- 可见性：多个线程访问同一个变量时，这个变量被修改后，能被其他的线程看到。
- 有序性。

并发与并行：

- 并发：同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；
- 并行： 单位时间内，多个任务同时执行。

线程的生命周期：初始 、运行(调用`start()`方法后)、阻塞(被锁阻塞)、等待（调用`wait()`方法）、超时等待（可在指定时间自己返回）、终止。

线程死锁：多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。

死锁必须具备以下四个条件：

1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:线程已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

# `synchronized`关键字

`synchronized` 是 Java 中的关键字，是利用锁的机制来实现同步的。如果多个线程访问的是同一个对象，哪个线程先执行带`synchronized`关键字的方法，则哪个线程就持有该方法，那么其他线程只能呈等待状态。

内部机制：`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit`指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。 当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 `monitor`(monitor对象存在于每个Java对象的对象头中，`synchronized` 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 `monitorexit` 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

`synchronized`对应两种锁：对象锁和类锁

- 对象锁。每个Java对象都有一个 monitor 对象，即 Java 对象的锁，通常被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。
-  类锁。在 Java 中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

内置锁的特性：

- 可重入性。如果某个线程试图获得一个已经由它自己持有的锁，那么这个请求就会成功。即同一个线程可以再次获取自己的锁而不会阻塞。比如可以在同步方法里再次调用相同锁的同步方法。

使用说明：

- 同步普通方法，锁的是当前对象。如：`public synchronized void method(){...}`
- 同步静态方法，锁的是当前的 Class对象。如：`public static synchronized void method(){...}`
- 同步块，锁的是 `()` 中的对象或Class对象。比如`synchronized (this){...}` 或 `synchronized (A.class){...}`。

**其他锁对象：Lock、ReenTrantLock等**

TODO

# `volatile`关键字

`volatile`，中文含义为“易改变”。用于修饰变量，告知系统该变量易改变，并发处理需直接读写主存，而非各自CPU内的缓存。

`volatile`只能保证变量的可见性、有序性，但是不能保证原子性。比如变量的自增操作，包括三个操作，读、加法、写，就是不是原子操作。在执行加法之前，其他线程可能更改了值，这样最终的结果可能不是预期的结果。

局限：

1. 不能替代synchronized。无法提供完全同步的能力，它只能提供改变可见性的能力，可称为一种轻量级的同步，多线程写的时候会出现问题。
2. 性能低。由于总是读写主存，它的读写性能要低于普通的变量。 

一般使用的情景为：一个线程写，多个线程读。 大部分情况下，完全不需要使用这个关键字。 

# 线程相关方法：wait、notify、sleep、interrupt等

- `Object.wait()`：在同步方法或同步块中调用；将当前线程置入休眠状态，直到接到通知或被中断为止；调用后会并释放锁。
- `Object.notify()`：在同步方法或同步块中调用；通知等待该对象的其他线程，多个线程时随机通知一个；当前程序退出`synchronized`代码块后，等待线程程序才能获取锁，即被唤醒。
- `Object.notifyAll()`：与`Object.notify()`一样，通知待该对象的所有线程。
- `Thread.sleep()`：暂停执行，超时后继续执行，不释放锁。
- `Thread.interrupt()`：中断睡眠中的线程，并抛出中断异常信息。

`wait()`和`sleep()`比较：

- 两者最主要的区别在于：sleep 方法没有释放锁，而 wait 方法释放了锁 。
- 两者都可以暂停线程的执行。
- Wait 通常被用于线程间交互/通信，sleep 通常被用于暂停执行。
- wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用 wait(long timeout)超时后线程会自动苏醒。

# 线程池与Executor 框架

线程池的作用：减少资源消耗，提高资源利用率，提高响应速度。

Executor 框架包括线程池管理、线程工厂、队列以及拒绝策略等，Executor 框架让并发编程变得更加简单。

Executor 框架结构，主要由三大部分组成：

- 任务(`Runnable` /`Callable`)：执行任务需要实现的 **`Runnable` 接口** 或 **`Callable`接口**。
- 任务的执行(`Executor`)：包括任务执行机制的核心接口 **`Executor`** ，以及继承自 `Executor` 接口的 `ExecutorService` 接口，以及继承自`ExecutorService`的`ThreadPoolExecutor`类。
- 异步计算的结果(`Future`)：**`Future`** 接口以及 `Future` 接口的实现类 **`FutureTask`** 类都可以代表异步计算的结果。当我们把 **`Runnable`接口** 或 **`Callable` 接口** 的实现类提交给 `ThreadPoolExecutor` 或 `ScheduledThreadPoolExecutor` 执行即调用 `submit()` 方法时，会返回一个 `FutureTask` 对象

`ThreadPoolExecutor` 的构造

```java
/**
 * 用给定的初始参数创建一个新的ThreadPoolExecutor。
 */
public ThreadPoolExecutor(
    int corePoolSize,//线程池的核心线程数量
    int maximumPoolSize,//线程池的最大线程数
    long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
    TimeUnit unit,//时间单位
    BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
    ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
    RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
) {...}
```

两种方法构造线程池：

1. 通过`ThreadPoolExecutor`构造函数实现（推荐）
2. 通过 Executor 框架的工具类 `Executors` 来实现。三种特定的线程池
   - `FixedThreadPool`
   - `SingleThreadExecutor`
   - `CachedThreadPool`

四种拒绝策略：

1. AbortPolicy：任务队列装不下，直接拒绝，抛异常。默认策略
2. CallerRunsPolicy：当前线程直接运行任务
3. DiscardPolicy：丢弃
4. DiscardOldestPolicy：丢弃最旧的任务

# ThreadLocal

通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。如果想实现每一个线程都有自己的专属本地变量该如何解决呢？ JDK中提供的`ThreadLocal`类正是为了解决这样的问题。 

`Thread`有成员`ThreadLocalMap`，而`ThreadLocalMap`存储以`ThreadLocal`为key的键值对。 比如我们在同一个线程中声明了两个 `ThreadLocal` 对象的话，会使用 `Thread`内部`ThreadLocalMap`存放数据的，`ThreadLocalMap`的 key 就是 `ThreadLocal`对象，value 就是 `ThreadLocal` 对象调用`set`方法设置的值。

> 在Android中利用ThreadLocal来保证Looper对象的唯一性：每个线程对象持有一个`ThreadLocalMap`成员，`ThreadLocalMap`以`ThreadLocal`为key，以实际目标值为value。如果以一个静态`ThreadLocal`对象为key，则每个线程里的`ThreadLocalMap`都会持有相同的key却互不并不影响，但在线程内部，由于`ThreadLocalMap`里的key唯一，则可以确保只有一个值。在Looper创建时则是判断当前线程是否有Looper，有则抛出异常，从而确保唯一。

# 参考

- [volatile的作用及正确的使用模式](https://zhuanlan.zhihu.com/p/79602008)
- 《Java 并发编程的艺术》
- [Java Scheduler ScheduledExecutorService ScheduledThreadPoolExecutor Example](https://www.journaldev.com/2340/java-scheduler-scheduledexecutorservice-scheduledthreadpoolexecutor-example "Java Scheduler ScheduledExecutorService ScheduledThreadPoolExecutor Example")
- [java.util.concurrent.ScheduledThreadPoolExecutor Example](https://examples.javacodegeeks.com/core-java/util/concurrent/scheduledthreadpoolexecutor/java-util-concurrent-scheduledthreadpoolexecutor-example/ "java.util.concurrent.ScheduledThreadPoolExecutor Example")
- [ThreadPoolExecutor – Java Thread Pool Example](https://www.journaldev.com/1069/threadpoolexecutor-java-thread-pool-example-executorservice "ThreadPoolExecutor – Java Thread Pool Example")



