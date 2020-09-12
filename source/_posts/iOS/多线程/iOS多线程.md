---
title: iOS多线程
date: 2019-12-17 21:08:14
categories: [iOS]
tags: [iOS, 多线程]
---
## 前言

iOS中进行多线程的操作其实比较常见，应用场景比较多，凡是耗时操作一般都应该放在子线程中进行。进行多线程操作的方式包括以下几个
* Thread
* Operation
* GCD
* Pthreads  
  这个是基于C语言的框架，可以跨平台使用，这个因为比较底层而且不常用，所以本文中就不进行讨论了。

## Thread
使用起来比较轻量级。每一个Thread都对应一个线程。需要自己管理线程的生命周期、线程同步、加锁、睡眠以及唤醒等。  

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
// 实例方法或属性
//启动
start() 
//取消，取消线程并不会马上停止并退出线程，仅仅只作（线程是否需要退出）状态记录
cancel() 


// 类方法或属性，一般是用在线程的执行体中
//线程停止，停止方法会立即终止除主线程以外所有线程（无论是否在执行任务）并退出，需要在掌控所有线程状态的情况下调用此方法，否则可能会导致内存问题。
exit() 
//当前线程
current
//线程休眠，会有阻塞当前线程的效果
sleep()
```

Thread现在提供的阻塞方法为sleep，只能控制其休眠多长时间或休眠到什么时间，外部无法手动唤醒。如果想实现手动唤醒的效果，可以借助

Thread可以进行继承，重写main方法

## Operation

Operation是苹果基于GCD封装的，面向对象使用；可控性比GCD要强，其中一个非常重要的特性是可以设置各操作之间的依赖，即强行规定操作A要在操作B完成之后才能开始执行，成为操作A依赖于操作B：  
相关的类有Operation和OperationQueue。其中Operation是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的子类：BlockOperation。创建Operation子类的对象，把对象添加到OperationQueue队列里执行。

## GCD
### 一、相关概念

Grand Central Dispatch (GCD) 是Apple开发的一个多核编程的较新的解决方法，它自动管理线程的生命周期，弱化了线程的概念；

iOS中的多线程会涉及到以下概念，其中GCD中的核心概念就是队列以及任务；

* **队列**
* **任务**
* **信号量**
* **任务组**
* **线程** 


### 二、使用场景介绍


#### 1、任务组

任务组的主要应用场景：当需要一组任务结束后再统一去执行一些操作；如等到几个没有顺序要求的网络请求成功之后再去统一刷新UI。

任务组（DispatchGroup）主要职责：当队列中所有任务都执行完毕之后，会发出一个通知表示任务执行完毕。其中任务组判断任务执行完毕的时机是**入组任务数等于出组任务数**。  
任务组与队列需要关联来实现上述操作，关联方式包括两种：自动关联及手动关联；

group.enter()和group.leave()需要成对存在。

当组内没有任务时，`group.notify`会直接执行,当任务组的入组数大于出组数，`group.notify`永远不会执行，而当出组数大于入组数，程序会崩溃。手动关联时需要注意这两种情况。

##### 1、自动关联  

自动关联方式会自动控制任务入组出组，其中出组是时机是闭包里面内容执行完毕，如果需要在闭包里面添加需要开线程的操作，如网络请求等，这种自动关联的方式不适合，因为闭包将网络请求发送出去之后就认为其内容执行完毕，然后出组，不会等结果返回，你也可以理解成闭包里面执行逻辑需要是串行执行的。

```swift
 // 任务组
let group = DispatchGroup()
// 并行队列
let queue = DispatchQueue(label: "label",attributes: [.concurrent])
queue.async(group: group) {
    Thread.sleep(forTimeInterval: 2)
    print("第二个显示")
}
queue.async(group: group) {
    hread.sleep(forTimeInterval: 1)
    print("第一个显示")
}

group.notify(queue: queue) {
    print("任务完成")
}

//运行结果
第一个显示
第二个显示
任务完成
```

##### 2、手动关联

上面说了自动关联的方式不适合需要在任务闭包里面并行执行的逻辑，而这种需求又非常常见，这个时候，我们就可以使用手动关联的方式，人工管理任务的出入组。

```swift
 // 任务组
let group = DispatchGroup()
// 并行队列
let queue = DispatchQueue(label: "label",attributes: [.concurrent])
group.enter()
group.enter()

queue.async() {
    Thread.sleep(forTimeInterval: 2)
    print("第二个显示")
    group.leave()
}

queue.async() {
    Thread.sleep(forTimeInterval: 1)
    print("第一个显示")
    group.leave()
}

group.notify(queue: queue) {
    print("任务完成")
}

//运行结果
第一个显示
第二个显示
任务完成
```

##### 3、建议写法

```swift
 // 任务组
let group = DispatchGroup()
// 并行队列
let queue = DispatchQueue(label: "label",attributes: [.concurrent])
queue.async(group: group) { //参数传入group
    group.enter()  //调用方法手动管理
    Thread.sleep(forTimeInterval: 2)
    print("第二个显示")
    group.leave()
}
queue.async(group: group) {
    group.enter()
    hread.sleep(forTimeInterval: 1)
    print("第一个显示")
    group.leave()
}

group.notify(queue: queue) {
    print("任务完成")
}

group.notify(queue: DispatchQueue.main) {
    print("在这里刷新UI")
}

//运行结果
第一个显示
第二个显示
在这里刷新UI
任务完成
```

#### 2、信号量（控制最大并发数）

任务组能保证几个网络请求全部完成之后再进行统一的操作，但是无法控制网络请求执行的顺序，如果需要控制网络请求执行的顺序（比如第二个网络请求的参数需要根据第一个网络请求返回值进行控制），我们就需要用到信号量了。

```swift
// 并行队列
let queue = DispatchQueue(label: "label",attributes: [.concurrent])
let semaphore = DispatchSemaphore(value: 1) //设置数量为1的信号量，即限制正在运行的任务只有一个

queue.async() {
    semaphore.wait()

   // 网络请求并获得回调
   semaphore.signal()
}
queue.async() {

}
```

### 3、串行、并行队列，同步、异步任务

串行、并行是队列的属性。




## 常用操作

### 对方法加锁，保证只能同一个线程进行访问
* 使用信号量
  ```swift
  let lock = DispatchSemaphore(value: 1)
  // 进入锁
  lock.wait()
  // 释放锁
  lock.signal()
  ```
* objc_sync_enter、objc_sync_exit（其实是OC里面的`@synchronized`）
  ```swift
  objc_sync_enter(object)
  objc_sync_exit(object)
  ```
* 使用NSLock
  ```swift
  let lock = NSLock()
  lock.lock()
  lock.unlock()
  ```