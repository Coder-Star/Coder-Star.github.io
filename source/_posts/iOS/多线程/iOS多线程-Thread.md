---
title: Thread
category:
  - iOS
  - 多线程
tags:
  - iOS
  - 多线程
  - Thread
date: 2021-01-26 23:27:03
---

使用起来比较轻量级。每一个 Thread 都对应一个线程。需要自己管理线程的生命周期、线程同步、加锁、睡眠以及唤醒等。

**创建**

```swift
// 闭包形式
// 类方法
@available(iOS 10.0, *)
open class func detachNewThread(_ block: @escaping () -> Void)
// 实例方法
@available(iOS 10.0, *)
public convenience init(block: @escaping () -> Void)

// 方式形式
// 类方法
open class func detachNewThreadSelector(_ selector: Selector, toTarget target: Any, with argument: Any?)
// 实例方法
@available(iOS 2.0, *)
public convenience init(target: Any, selector: Selector, object argument: Any?)

//类方法创建的线程自动运行，利用类方法创建没有返回值，所以如果需要获取创建的thread，需要在相应的selector方法中调用获取当前线程的方法
//实例方法创建的线程需要手动调用start方法进行运行
```

**方法**

```swift
// 实例方法或属性，一般用在实例方式创建线程
//启动
start()
//取消，取消线程并不会马上停止并退出线程，仅仅只作（线程是否需要退出）状态记录
cancel()


// 类方法或属性，一般是用在线程的执行体中，用在用类方法创建的线程中
//线程停止，停止方法会立即终止除主线程以外所有线程（无论是否在执行任务）并退出，需要在掌控所有线程状态的情况下调用此方法，否则可能会导致内存问题。
exit()
//当前线程
current
//线程休眠，会有阻塞当前线程的效果
sleep()
```

Thread 现在提供的阻塞方法为 sleep，只能控制其休眠多长时间或休眠到什么时间，外部无法手动唤醒。如果想实现手动唤醒的效果，可以借助

Thread 可以进行继承，重写 main 方法
