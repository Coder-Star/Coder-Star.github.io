---
title: iOS 多线程 - Operation
category:
  - iOS
  - 多线程
tags:
  - iOS
  - 多线程
  - Operation
date: 2021-01-26 23:29:20
---

## 前言

Hi Coder，我是 CoderStar！

我们之前已经讲过 [iOS多线程-Thread](../iOS多线程-Thread) 以及 [iOS多线程-GCD](../iOS多线程-GCD)，我们今天来聊一聊iOS多线程中最后一种比较常用的方式--`Operation`。

## 概览

对于 Operation 而言，其相关的类比 GCD 要少的多。`Operation`本身是一个`抽象类`，不能直接进行使用，其定义了相关的方法及属性，需要靠子类进行相应的实现，系统已经实现了一个 --`BlockOperation`。（在 OC 中，还有一个是`NSInvocationOperation`，但在 Swift 中，该子类已经在 Swift4 里去掉，想必去掉的原因大家也很容易理解，因为 Swift 语言本身就不推荐 `selector` 这种形式）。

`Operation` 底层建立在`GCD`之上，是更高一级的抽象，使我们可以面向对象（Cocoa 对象）的方式进行多线程编程。

> 其实 `NSOpertion` 是先于`GCD`引进的，在当时，`NSOperationQueue` 接收 `NSOperation` 对象并创建一个线程，并在对应线程上运行 `main`方法 ，运行完成之后再杀死该线程。那这种方式相对于后面出现的`GCD`底层的线程池而言，效率就很低，所以在 Mac OS 10.5 以及 iOS 2 开始便对`NSOpertion`底层在基于`GCD`的基础上进行完全重写，利用`GCD`的相关特性提高性能并提供了一些新功能。如果想简单佐证下，可以看到`OperationQueue`拥有一个`unowned(unsafe) open var underlyingQueue: DispatchQueue?`属性。

## 属性 & 方法

先罗列一下`Operation`及`OperationQueue`主要的属性及方法。

> 注释会有相应说明。

### Operation

``` swift
// Operation

// MARK: - 属性

/// 下列几个属性为Operation的状态，只读属性

open var isReady: Bool { get }
open var isExecuting: Bool { get }
open var isCancelled: Bool { get }
open var isFinished: Bool { get }


/// 该属性已经被下列isAsynchronous属性替代
open var isConcurrent: Bool { get }

/// 用来标识当前操作是否为异步
/// 需要注意该属性是当直接调用start方法时才会生效，但添加Operation到OperationQueue，队列将忽略该属性的值
/// 默认值为false
@available(iOS 7.0, *)
open var isAsynchronous: Bool { get }


/// 操作优先级
/// 当队列中operation很多时，我们可以通过设置该属性来调整Operation在 同一队列中 的优先级，同时这个前提是Operation都是处于readey状态ati
open var queuePriority: Operation.QueuePriority

/// 该属性与Thread所拥有的服务质量等级属性一致
/// 主要用来描述任务在进程中整体的优先级
@available(iOS 8.0, *)
open var qualityOfService: QualityOfService

/// 任务完成后的回调方法
/// 当isFinished属性设置为YES时才会执行该回调
@available(iOS 4.0, *)
open var completionBlock: (() -> Void)?

// MARK: - 方法

/// 启动
/// 并发Operation时需要重写该方法
/// 可以不把operation加入到队列中，手动触发执行，与调用普通方法一样
open func start()

/// 非并发Operation需要重写该方法
open func main()

/// 取消操作
open func cancel()

/// 添加依赖
open func addDependency(_ op: Operation)

/// 移除依赖
open func removeDependency(_ op: Operation)
```

对`Operation`几个属性、方法再进行详细的说明：

`cancel`方法
如果这个操作正在执行，调用 cancel() 只会将状态 `isCanceled` 置为 true，但不会影响操作的继续执行。
如果操作还没执行，调用 cancel() 会将状态 `isCanceled` 和 `isReady` 置为 true, 如果执行取消后的操作，会直接将状态 `isFinished` 置为 true 而不会执行操作。也会触发`completionBlock`方法。

`addDependency`方法
- 需要注意在设置时不要设置成循环依赖，比如 A 依赖 B、B 又依赖 A，这样会形成死锁，导致谁也不会执行。
- 可以跨操作队列设置依赖。
- 当给某个 Operation 添加依赖的 Operation 后，只有其所依赖的所有 Operation 都执行完毕，当前的 Operation 才能开始执行。不管依赖的 Operation 是执行成功了还是失败了，或者是取消了，都认为是执行完毕了。

**OperationQueue**

``` swift
// Operation

// MARK: - 属性

/// 最大并发操作数，也就是该队列中最多允许几个Operation在同时运行
open var maxConcurrentOperationCount: Int

// MARK: - 方法

/// 取消所有操作
open func cancelAllOperations()

/// 调用该方法会阻塞当前线程，等待所有任务完成之后才会执行后续逻辑
/// 和 DispatchGroup 一定条件下是类似的
open func waitUntilAllOperationsAreFinished()

/// 添加 Operation
open func addOperation(_ op: Operation)

/// 闭包形式添加 Operation
@available(iOS 4.0, *)
open func addOperation(_ block: @escaping () -> Void)

/// 类似 GCD 的栅栏函数
@available(iOS 13.0, *)
open func addBarrierBlock(_ barrier: @escaping () -> Void)
```

对`OperationQueue`几个属性、方法再进行详细的说明：

`maxConcurrentOperationCount`属性
- maxConcurrentOperationCount 如果不设置值时，默认值会取`defaultMaxConcurrentOperationCount`，也就是 -1，此时默认最大操作数由 `OperationQueue` 对象根据当前系统条件（系统内存与 CPU）动态确定。
- maxConcurrentOperationCount 为 0 时，队列中的 Operation 不会执行。
- maxConcurrentOperationCount 为 1 时，队列串行执行。
- maxConcurrentOperationCount 大于 1 时，队列并发执行，当然这个值不应超过系统限制，即使自己设置一个很大的值，系统也会自动调整为 min{自己设定的值，系统设定的默认最大值}，系统默认限制应该是 64。

> 需要注意，因为有`queuePriority`的存在，同一个 Queue 的 Operation 之间有优先级，所以先进入 Queue 的 Operation 不一定先运行，所以当`maxConcurrentOperationCount`设置为 1 时并不是一个真正意义上的串行队列，优先级较高后加入的 Operation 有可能会先执行。

> 64 这个值在 GCD 下应该也是默认最大线程数，但是可以调整目标队列的优先级进行调整。这里涉及到一个线程爆炸的概念，后面可能还会出一篇文章写这些东西。

## 使用

对于一般的任务，我们可以直接使用`BlockOperation`。使用示例如下：

```swift
let operation = BlockOperation {
  // do something
}
let queue = OperationQueue()
queue.addOperation(operation)
```

但是很多时候，我们需要继承`Operation`进行一些自定义操作。

## GCD VS Operation

使用 GCD 还是使用 Operation 这个问题其实在社区已经争论了很久，从斯坦福大学的 CS193p 课程推荐使用 GCD，到 WWDC 2012 时演讲者推荐使用`Operation`，也能看出开发者对该问题的看法不一致，该节我们主要来聊一聊两者各自优势以及差别。

> 目前网络上的很多文章都是基于没有`DispatchWorkItem`对象前提下对 GCD 和`Operation`做的对比，大家阅读时需要注意一下。

1、从两者所在层次来讲：GCD 底层是 C 语言的 API，而 Operation 是 GCD 基础上更高层次的抽象，那 GCD 相对 Operation 来说肯定是又快又轻的；

但是反过来来说因为 Operation 是更高层次的抽象，按照一般的经验法则来看，**我们应首先使用最高级别的 API，然后在根据需要完成内容进行降级**。从这一角度来看，使用 Operation 抽象度更高，更符合面向对象的思想，也有利于底层的无痕变更。

2、从两者提供的 API 来讲：其实 GCD 和 Operation 两者之间是很相似的，特别是当`DispatchWorkItem`对象（@available(macOS 10.10, iOS 8.0, *)）出来之后，从一定意义上讲，`DispatchWorkItem`可以类比到`Operation`对象，`DispatchQueue`可以类比到`OperationQueue`对象。比如`DispatchWorkItem`和`Operation`对象都可以进行`cancel`等操作，`DispatchQueue`、`OperationQueue`对象都可以添加任务或操作（对象以及闭包两种形式），栅栏函数，进行挂起以及恢复（前者是两个对应方法，后者是一个属性）等。但两者还是有一些区别的，比如：

- `OperationQueue`可以设置并发操作的最大数量`maxConcurrentOperationCount`。
  > 在一定条件下可以类比到 GCD 的信号量
- 在不同的任务之间建立依赖关系`addOperation`；
  > 在一定条件下可以类比到 GCD 的 `DispatchWorkItem`的`public func notify(queue: DispatchQueue, execute: DispatchWorkItem)` 方法
- `OperationQueue`可以取消队列中的所有操作。
- ...

## 应用

应用：Alamofire 中使用 SDWebImage

[AsyncOperation](https://github.com/Coder-Star/LTXiOSUtils/blob/master/LTXiOSUtils/Classes/Util/AsyncOperation.swift)

## 最后

要更加努力呀！

Let's be CoderStar!

推荐学习资料

- [NSOperation vs Grand Central Dispatch](https://stackoverflow.com/questions/10373331/nsoperation-vs-grand-central-dispatch/10375616#10375616)
- [guide-to-blocks-grand-central-dispatch](https://cocoasamurai.blogspot.com/2009/09/guide-to-blocks-grand-central-dispatch.html)
- [When to use NSOperation vs. GCD](http://eschatologist.net/blog/?p=232)
- [Operation and OperationQueue Tutorial in Swift](https://www.raywenderlich.com/5293-operation-and-operationqueue-tutorial-in-swift)
- [operation](https://developer.apple.com/documentation/foundation/operation)
- [Advanced NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/)
