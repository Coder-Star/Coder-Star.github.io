---
title: iOS 中的锁
category:
  - iOS
  - 多线程
tags:
  - iOS
  - 多线程
date: 2021-03-02 22:52:45
---

![iOS锁性能对比](../../../img/iOS/多线程/iOS锁性能对比.png)

- 自旋锁
  - OSSpinLock
- 互斥锁
  - 可递归
    - pthread_mutex(recursive)
      - NSRecursiveLock
    - os_unfair_recursive_lock（私有）
      - recursive_mutex_t （内部基于 os_unfair_lock）
        - synchronized
  - 不可递归
    - pthread_mutex
      - NSLock
    - os_unfair_lock
  - 条件锁
    - NSCondtion（pthread_mutex_t + pthread_cond_t）
      - NSConditionLock
  - 信号量
    - semaphore
- 读写锁
  - pthread_rwlock（读写锁）

> - pthread 头文件为 `#import <pthread.h>`
> - os_unfair_lock 头文件为`#import <os/lock.h>`

## iOS 中锁分析

### @synchronized

@synchronized（objc_sync_enter/objc_sync_exit）

互斥递归锁

```swift
// NSLock形式
let lock = NSLock()
lock.lock()
// 执行内容
lock.unlock()
```

```swift
// 两本质相同，其中

// oc
// 这种方式使用简单，支持多线程递归嵌套
// 需要注意使用期间self这个参数不可为nil，其底层会执行objc_sync_enter及objc_sync_exit这两个函数，其最终底层是依靠recursive_mutex_t锁来进行加锁的。这也是synchronized可以支持递归调用的原因。
// 当传入的self对象是nil的时候，线程安全就会失效。
// 为什么  @synchronized 是性能最差的呢？因为其包含的操作极为复杂，除了常规的加锁解锁操作以外，还需要考虑哈希表寻址，缓存获取/创建缓存等，最差情况下即 N 个 不同的 obj 创建多个不同的  SyncData，并且会调用命名为自旋锁的互斥锁 os_unfair_lock 来实现缓存.

/// 括号内内容不可为nil
@synchronized (self) {
    // 执行内容
}

// swift
objc_sync_enter(self)
// 执行内容
objc_sync_exit(self)
```

> `recursive_mutex_t` 是一个互斥递归锁，它是基于 `os_unfair_recursive_lock` 互斥锁的封装（再早些版本是对 `pthread_mutex_t` 的封装），而 `os_unfair_recursive_lock` 底层是对 os_unfair_lock 的封装，进一步跟进代码会发现。`os_unfair_recursive_lock` 是 iOS 12.0 之后才可用的，即 iOS 12.0 之后 `@synchronized` 是一个封装了 `os_unfair_lock` 的互斥递归锁。

### 互斥锁

- pthread_mutex（支持递归）
- NSLock(底层基于`pthread_mutex`)

```objective-c
// pthread_mutex形式
var mutex = pthread_mutex_t()
func initLock() {
  pthread_mutex_init(&mutex, nil)
}
pthread_mutex_lock(&mutex)
// 执行内容
pthread_mutex_unlock(&mutex)
```

### 自旋锁

自旋锁不会使线程状态发生切换，不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快，比较适用于锁使用者保持锁时间比较短的情况。如果不能在很短的时间内获得锁，CPU 效率降低。

> 自旋锁与互斥锁的区别？

自旋锁与互斥锁类似，它们都是为了解决对某项资源的互斥使用，在任何时刻最多只能有一个线程获得锁；
对于互斥锁，如果资源已经被占用，调用者将进入睡眠状态；
对于自旋锁，如果资源已经被占用，调用者就一直循环在那里，看是否自旋锁的保持者已经释放了锁；

> 线程之间的切换也是非常耗性能的，大概需要 20 毫秒的时间。

优先级反转问题：

A 线程优先级高，B 线程优先级中，C 线程优先级低；
1. C 占据资源 X；
2. A 启动，也要资源 X，但是因为 C 占据，所以阻塞；
3. B 启动，因为优先级高，所以抢夺了 CPU；

此时 A 优先级最高却因为 C 的原因导致了优先级更低的 B 更优先执行，这就被称为优先级反转问题。

> 优先级继承：就是为了解决优先级反转问题而提出的一种优化机制。其大致原理是让低优先级线程在获得同步资源的时候 (如果有高优先级的线程也需要使用该同步资源时)，临时提升其优先级。以便其能更快的执行并释放同步资源。释放同步资源后再恢复其原来的优先级。
> 而这套机制要求系统知道当前持有锁的线程相关信息；这也是目前`mutex`系列锁解决优先级反转问题的办法；

iOS官方对这套机制相关描述位于 [Prioritize Work with Quality of Service Classes](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html#//apple_ref/doc/uid/TP40015243-CH39-SW1)的 `Priority Inversions` 章节。

[优先级反转那点事儿](https://zhuanlan.zhihu.com/p/146132061)

`OSSpinLock`
- `OSSpinLock`不会记录持有它的线程信息，所有当发生优先级反转时，系统找不到低优先级的线程，可能因此无法通过提高优先级解决优先级反转问题
- `OSSpinLock`因为自旋的性质导致

我个人柑橘

**OSSpinLock**，已废弃，编译会报警告，大家已经不再使用它了，因为它在一些场景下已经不安全了。**os_unfair_lock** 是苹果官方推荐的替换 OSSpinLock 的方案，但是它在 iOS10.0 以上的系统才可以调用。os_unfair_lock 是一种互斥锁，它不会向自旋锁那样忙等，而是等待线程会休眠。

[Thread safety of weak properties](https://lists.swift.org/pipermail/swift-dev/Week-of-Mon-20151214/000343.html)

有人会问这样的问题：为什么不用其他的互斥锁来替换`OSSpinLock`，而是选用`os_unfair_lock`，我个人感觉其实是 apple 的一个选择而已。

> 查看 CoreFoundation 的源码能够发现，苹果至少在 2014 年就发现了这个问题，并把 CoreFoundation 中的 spinlock 替换成了 pthread_mutex，具体变化可以查看这两个文件：CFInternal.h(855.17)、CFInternal.h(1151.16)。
>
> 在 iOS 10/macOS 10.12 发布时，苹果提供了新的 os_unfair_lock 作为 OSSpinLock 的替代，并且将 OSSpinLock 标记为了 Deprecated。
>
> google/protobuf 内部的 spinlock 被全部替换为 dispatch_semaphore，详情可以看这个提交：https://github.com/google/protobuf/pull/1060。用 dispatch_semaphore 而不用 pthread_mutex 应该是出于性能考虑。

`os_unfair_lock` 是一个不公平的锁，其和`OSSpinLock`一样，`os_unfair_lock`也没有加强公平性和顺序。例如，释放锁的线程可能立即再次加锁，而之前等待锁的线程唤醒后没有机会尝试加锁。这样有利于提高性能，但也造成了饥饿（`starvation`）。

> Starvation: 指贪婪线程占用共享资源太长时间，其他线程无法访问共享资源、无法取得进展。例如，某对象的同步方法占用时间过长，并且频繁调用，其他线程尝试调用该方法时会被堵塞，处于饥饿状态。

作为非公平锁的实现，`os_unfair_lock`直接由 CPU 来调度，不会像`mutex`那样保证等待队列的出列顺序，也就减少了线程切换的资源消耗。

对比 `mutex`：
- 优点：
  - 申请锁的内存消耗更少，os_unfair_lock 为 int32，而 mutex 的结构体显然比 os_unfair_lock 大
  - 减少由于要保持队列顺序，而产生线程上下文切换带来的消耗。
- 缺点：
  - 无法保证被锁住线程的执行顺序。很可能会出现某个线程长时间不会执行的情况

### 条件锁

- NSCondition(底层也是`pthread_mutex`)
- NSContionLock(基于 NSCondition 实现，只不过比 NSCondition 少了一些繁琐的操作，可以用来做多任务顺序执行)

### 读写锁

一般使用多读单写的场景

- 使用栅栏函数。
- pthread_rwlock

**栅栏函数**
使用一个并行队列，读使用异步任务，写使用栅栏函数。这里使用并行队列的原因是因为如果串行队列，多读的时候就是串行的，影响效率

```swift
let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)

/**
  使用栅栏函数使用多读单写
 */
func readFile() -> String {
  // 这里使用同步任务，阻塞进入的线程，保证即读即得
  concurrentQueue.sync {
    // 读写文件
  }
}

func writeFile() {
    // 这里使用异步任务，因为存入后不需要及时得到反馈结果
    concurrentQueue.async(flags: [.barrier]) {
    }
}
```

**pthread_rwlock**

```swift
class FileTest: XCTestCase {

  // 文件读写锁
  var pthreadRwlock = pthread_rwlock_t()


  func initRwlock() {
    pthread_rwlock_init(&pthreadRwlock, nil)
  }

  func readFile() -> String {
    pthread_rwlock_rdlock(&pthreadRwlock)
    defer {
      pthread_rwlock_unlock(&pthreadRwlock)
    }
    // 读文件
    return ""
  }

  func writeFile() {
    pthread_rwlock_wrlock(&pthreadRwlock)
    defer {
      pthread_rwlock_unlock(&pthreadRwlock)
    }
    // 写文件
  }

  deinit {
    pthread_rwlock_destroy(&pthreadRwlock)
  }
}

```

### DispatchSemaphore

```swift
let lock = DispatchSemaphore(value: 1)
// 进入锁
lock.wait()
// 释放锁
lock.signal()
```

> 信号量因为持有者有多个，导致不知道该调整谁的优先级，也会存在相应的优先级反转问题；
> dispatch_group 类似，在调用 enter() 方法时，无法预知谁会调用 leave()，所以系统也无法知道其 owner 是谁

### 其他类型

我们可以借助一些其他的数据结果来实现锁；

如串行队列、OperationQueue 将最大操作数设置为 1。

### 递归锁 / 可重入锁

- NSRecursiveLock
- pthread_mutex(recursive)

```
pthread_mutex_t lock;
pthread_mutexattr_t attr;
pthread_mutexattr_init(&attr);
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
pthread_mutex_init(&lock, &attr);
pthread_mutexattr_destroy(&attr);
pthread_mutex_lock(&lock);
pthread_mutex_unlock(&lock);
```

## 死锁

死锁产生的四个条件，当四个条件均满足时，必然造成死锁。

- 互斥
- 占有且等待
- 不可抢占
- 循环等待

避免死锁的办法

- 死锁预防 ----- 确保系统永远不会进入死锁状态
- 避免死锁 ----- 在使用前进行判断，只允许不会产生死锁的进程申请资源
- 死锁检测与解除 ----- 在检测到运行系统进入死锁，进行恢复

- [Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)

-[iOS Teach Team iOS 常用锁的底层实现及使用详解](https://github.com/minhechen/iOSTechTeam/blob/main/Blogs/iOSTechTeam_04.md)

- [深入理解iOS中的锁](https://devyang.space/2020/03/25/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3iOS%E4%B8%AD%E7%9A%84%E9%94%81/)

- [不再安全的 OSSpinLock](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)