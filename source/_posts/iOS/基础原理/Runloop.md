---
title: Runloop
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 基础原理
date: 2021-03-03 21:45:53
---

- RunLoop 本质上是一个对象,这个对象可以保持程序的持续运行并且处理程序中的各种事件(如触摸事件,定时器时间,selector 事件).
- RunLoop 没有事情处理时就会使线程进入睡眠状态.这样可以节省 CPU 资源,提高程序性能.

![](../../../img/iOS/底层/runloop.png)

- Entry 进入
- BeforeTimers
- BeforeSources
- BeforeWaiting 准备进入休眠
- AfterWaiting
- Exit 退出

**模式**

1. kCFRunLoopDefaultMode：App 的默认 Mode，通常主线程是在这个 Mode 下运行
2. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用，会切换到 kCFRunLoopDefaultMode
4. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
5. kCFRunLoopCommonModes: 这是一个占位用的 Mode，作为标记 kCFRunLoopDefaultMode 和 UITrackingRunLoopMode 用，并不是一种真正的 Mode

RunLoop 只会运行在一个模式下，要切换模式，就要暂停当前模式，重新启动一个运行模式

**作用**

- 保持程序持续运行
- 处理 App 中各类事件
- 节省 CPU 资源，提高程序性能

**实际应用**

- 控制线程生命周期（线程保活、线程永驻）
- TableView 延迟加载图片  
  把 setImage 放到 NSDefaultRunLoopMode 去做，也就是在滑动的时候并不会去调用赋值图片的方法，而是会等到滑动完毕切换到 NSDefaultRunLoopMode 下面才会调用 `imageView.perform(#selector(setImage), with: nil, afterDelay: 0, inModes: [.default])`
- 解决 NSTimer 在滑动时停止工作的问题  
  将 Timer 添加到 CommonMode 里面即可，`RunLoop.current.add(timer, forMode: .common)`
- 监测 RunLoop 的状态监测应用卡顿  
  根据 //TODO

* 每一条线程都有一个 Runloop 对应；但 Runloop 可以嵌套子 Runloop。
* 主线程的 Runloop 的对象系统已经自动帮我们创建好了,并且只有主线程结束时即程序结束时才会销毁；
* 子线程的 Runloop 对象需要我们主动创建并维护,子线程的 Runloop 对象在第一次获取时就会创建,销毁则是在子线程结束时. 并且创建出来的 runLoop 对象默认是不开启的,必须手动开启 RunLoop；
* Runloop 并不保证线程安全,我们只能在当前线程内部操作当前线程的 Runloop 对象,而不能在当前线程中去操作其他线程的 RunLoop 对象；

```swift
//获取当前线程的RunLoop对象,在子线程中调用时如果是第一次获取内部会帮我们创建RunLoop对象
let runloop = RunLoop.current
// 运行runloop
runloop.run()
```

配合 CADisplayLink 监听 FPS，基本原理就是统计每一秒中 CADisplayLink 执行的次数就 OK 啦

```
let displayLink = CADisplayLink(target: self, selector: #selector(displayLinkAction(displayLink:)))
displayLink.add(to: .current, forMode: .common)
```
