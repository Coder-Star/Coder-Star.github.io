---
title: iOS 多线程 -GCD
category:
  - iOS
  - 多线程
tags:
  - iOS
  - 多线程
  - GCD
date: 2021-01-26 23:28:08
---

## 前言

Hi Coder，我是 CoderStar！

今天给大家带来多线程系列的第二篇文章 -- `GCD`，其大概率是我们在使用多线程时最常用的方式了。

GCD 全称是 `Grand Central Dispatch`，翻译过来就是大规模中央调度。根据官方文档，它的作用是：通过向系统管理的调度队列中提交任务，在多核硬件上同时执行代码。它提供了一套机制，让你可以充分利用硬件的多核性能，并且让你不用再调用那些繁琐的底层线程 API，编写易于理解和修改的代码。

## 队列

串行、并行是队列的属性；

下列为自定义队列的构造函数

```swift
public convenience init(label: String,
                        qos: DispatchQoS = .unspecified,
                        attributes: DispatchQueue.Attributes = [],
                        autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit,
                        target: DispatchQueue? = nil)
```

介绍一下各个参数的作用：

**label**
**qos**
**attributes**
**autoreleaseFrequency**

**target**

这个属性设置的是队列的目标队列，即实际上会将该队列的任务放到指定队列中运行。

[DispatchQueue setTarget问题](https://bugs.swift.org/browse/SR-1859)

**其实在程序中手动创建的队列，最终都会指向系统自带的主队列或者全局队列。**

### 串行队列

串行队列主要是保证队列中的任务按照加入顺序依次执行，也就说后加入的任务必须等到同队列前面的任务都执行完毕之后才会执行。串行队列执行任务时候不允许**被当前队列中的任务**阻塞（会发生死锁），但可以被其他队列任务阻塞。

```swift
let serialQueue = DispatchQueue(label: "com.star.serialQueue")
```

### 并行队列

```swift
// 并行队列创建
let concurrentQueue = DispatchQueue(label: "com.star.concurrentQueue", attributes: .concurrent)
```

并行队列不需等待之前的任务执行完毕，任务并行执行。

### 主队列

```swift
let mainQueue = DispatchQueue.main
```

主队列，是一个特殊的**串行队列**，其永远运行在主线程中，它主要处理 UI 相关任务，也可以处理其他类型的任务。

同时需要注意一下主队列与主线程之间的区别。主队列一定是运行在主线程中，但是主线程却不只运行主队列，还可以运行其他的队列。

所以我们一般可以看到下列这样的代码，这段代码在[Kingfisher](https://github.com/onevcat/Kingfisher/blob/master/Sources/Utility/CallbackQueue.swift)中有相应使用。

```swift
extension DispatchQueue {
    /// 切换队列
    ///
    /// - Parameter block: 代码块
    public func safeAsync(_ block: @escaping () -> Void) {
        if self === DispatchQueue.main && Thread.isMainThread {
            block()
        } else {
            async { block() }
        }
    }
}
```

咋一看，觉得这样写是不是没必要，其实不然，这样写有两个好处

- 避免某些情况下切换非主队列到主队列，造成不必要的切换队列的开销；
- 同时避免切换队列造成的执行时序问题；

代码举例，解释见相应注释

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    /// sync会阻塞当前线程，使任务在当前线程也就是主线程中执行，执行队列却是全局并行队列
    DispatchQueue.global().sync {
        print("主队列代码前")
        /// 执行环境为主线程，可以直接执行代码，
        /// 不需要将队列切换到主队列，而是直接在当前队列执行即可，避免切换队列的开销
        ///
        /// 不切换到主队列，可以直接执行，先输出 主队列代码内 再输出 主队列代码后
        /// 切换到主队列，会先输出 主队列代码后 再输出 主队列代码内
        DispatchQueue.main.safeAsync {
            print("主队列代码内")
        }
        print("主队列代码后")
    }
}
```

### 全局并行队列

全局并行队列，存在 5 个不同的 Qos 级别，可以使用默认优先级，也可以单独指定。

```swift
// 构造函数
public class func global(qos: DispatchQoS.QoSClass = .default) -> DispatchQueue

// 使用方法
let globalQueue = DispatchQueue.global() // qos: .default
let globalQueue = DispatchQueue.global(qos: .background) // 后台运行级别
```

* background
* utility
* default
* userInitiated
* userInteractive
* unspecified

## 任务

同步、异步是对任务的描述，不是对线程的描述。

### 同步任务

```swift
// 同步任务
queue.sync {
}
```

同步任务会阻塞当前线程，不会开辟线程；任务会直接在当前线程执行，任务完成后恢复线程原任务；

### 异步任务

```swift
// 异步任务
queue.async {
}
```

异步任务不会阻塞当前线程，会开辟新的线程（主队列除外）。

## 使用 sync 产生死锁的情况

一般出现错误为`EXC_BAD_INSTRUCTION`

- 在主线程使用 sync

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    DispatchQueue.main.sync {

    }
}
```

- 串行队列同步任务中开启同步任务
> 队列 1 中有同步任务 A 正在执行，A 任务执行过程中又向队列 1 中加入了一个新的同步任务 B，此时会发生死锁。解释一下死锁发生的原因，因为是串行队列，所以 B 任务需要等到 A 任务执行完毕之后才能执行，但是 A 任务被 B 任务阻塞了线程，需要 B 任务执行完毕之后才可以继续执行，就造成了 A 等 B，B 等 A 的现象，产生死锁。

```swift
let serialQueue = DispatchQueue(label: "serialQueue")
serialQueue.sync {
    serialQueue.sync {

    }
}
```

- 串行队列异步任务中开启同步任务

```swift
let serialQueue = DispatchQueue(label: "serialQueue")
serialQueue.async {
  serialQueue.sync {

  }
}
```

## 栅栏函数

栅栏函数需要放在并行队列中才能发挥其作用。其作用是拦截栅栏函数前的任务，等到栅栏函数执行完后，在执行后面的并发任务。

**栅栏函数不能用在全局并发队列中，不起作用，作用会与普通的同步、异步任务相同。苹果官方也规定了不允许在全局并发队列中使用栅栏函数。**

> 全局队列是进程内的共享资源。系统框架需要能够依赖于不受未知联锁影响的全局队列，否则低级别框架进程可能会被更高级别的用户活动阻止，可能导致死锁。GCD 使开发人员能够在全局队列之上建立任意并发抽象和互锁，因此抱怨它们不在全局队列中是没有实际意义的。问为什么不能在全局队列上设置障碍就像问为什么抢占式多任务系统上的一个进程不能阻塞所有其他进程。

## 信号量

## 使用场景介绍

### 任务组

任务组的主要应用场景：当需要一组任务结束后再统一去执行一些操作；如等到几个没有顺序要求的网络请求成功之后再去统一刷新 UI。

任务组（DispatchGroup）主要职责：当队列中所有任务都执行完毕之后，会发出一个通知表示任务执行完毕。其中任务组判断任务执行完毕的时机是**入组任务数等于出组任务数**。
任务组与队列需要关联来实现上述操作，关联方式包括两种：自动关联及手动关联；

group.enter() 和 group.leave() 需要成对存在。

- 当组内没有任务时，`group.notify`会直接执行；
- 当任务组的入组数大于出组数，`group.notify`永远不会执行；
- 当出组数大于入组数，**程序会崩溃**。

> group.notify 是异步执行的，如果想要阻塞当前线程，使任务组的任务执行完毕，可以使用 group.wait()。

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

### 信号量（控制最大并发数）

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

## 最后

要更加努力呀！

Let's be CoderStar!

参考资料

- [DispatchQueueTest](https://github.com/pmtao/DispatchQueueTest)
