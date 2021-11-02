---
title: iOS 中的定时器
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 定时
date: 2021-02-15 10:42:29
---

## 前言

Hi Coder，我是 CoderStar！

我们平时开发时，或多或少都会使用到定时器，今天我们来聊聊 iOS 中的定时器。

iOS 中的定时器一般会包含三种：
- Timer
- CADisplayLink
- DispatchSourceTimer

## Timer

老规矩，我们先罗列一下 Timer 常用的方法及属性。

```swift
// MARK: - 构造函数
// MARK: - 静态方法
open class func scheduledTimer(timeInterval ti: TimeInterval, target aTarget: Any, selector aSelector: Selector, userInfo: Any?, repeats yesOrNo: Bool) -> Timer

@available(iOS 10.0, *)
open class func scheduledTimer(withTimeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void) -> Timer

// MARK: - 实例构造方法
public init(timeInterval ti: TimeInterval, target aTarget: Any, selector aSelector: Selector, userInfo: Any?, repeats yesOrNo: Bool)

@available(iOS 10.0, *)
public init(timeInterval interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void)

// MARK: - 设置起始触发时间
@available(iOS 10.0, *)
public convenience init(fire date: Date, interval: TimeInterval, repeats: Bool, block: @escaping (Timer) -> Void)

public init(fireAt date: Date, interval ti: TimeInterval, target t: Any, selector s: Selector, userInfo ui: Any?, repeats rep: Bool)

// MARK: - invocation方式
/// 因Swift中没有 NSInvocation，所以没法直接使用，如果非要使用可以借助OC进行中转
public init(timeInterval ti: TimeInterval, invocation: NSInvocation, repeats yesOrNo: Bool)

open class func scheduledTimer(timeInterval ti: TimeInterval, invocation: NSInvocation, repeats yesOrNo: Bool) -> Timer

// MARK: - 常用方法
/// 宽容度
@available(iOS 7.0, *)
open var tolerance: TimeInterval

/// 触发，启动
open func fire()
/// 从当前Runloop移除timer
open func invalidate()
```

从上述代码我们可以发现 Timer 生成实例的方式有八种，除了可以设置触发起止时间的两个之外，剩余六个为`Block`、`Target-Action`以及`NSInvocation`三种形式并且都提供`类方法`以及`构造函数`两种形式。我们可以根据自己的需求选择合适的构造函数。

1、`NSInvocation`在 Swift 中已经被禁止使用了，所以一般很少使用，如果非得使用需要借助 OC 进行中转；
2、`Block`方式是在 iOS 10 之后的，目的就是方便使用，并且避免了`Target-Action`这种方式惯有的循环引用问题，建议大家优先使用这种形式；
3、`Target-Action`方式最开始就存在，所以使用也比较频繁。

> `NSInvocation`的禁止其实也会影响到 `NSProxy` 在 Swift 中的使用，在 OC 中，我们一般会采用继承 `NSProxy`中的方式实现一个弱代理来解决常见的循环引用问题，比如常用的`YYKit`中的 [YYWeakProxy](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYWeakProxy.h)，但是在 Swift 中这种方式是不行的，需要继承`NSObject`来进行实现，这个具体后面会有介绍。

Timer 的运行需要依赖于 Runloop，如果 Timer 所处线程没有开启 Runloop，Timer 也是无法正常启动的，普通构造函数与类方法创建的 Timer 两者在这部分会有差异，`init`创建的 Timer 不会自动加入到 Runloop 中，需要再手动进行添加，而`scheduledTimer`形式会自动加入到当前线程对应的`Runloop`中。
> `scheduledTimer`形式自动加入的是`Runloop`的`default`模式，如果在滚动状态下仍然需要保持计时，则需要再手动加入到`eventTracking`或者`common`模式下，如果在子线程中，因为其 Runloop 没有默认启动，所以还是需要自己手动加入。

```Swift
let timer = Timer(timeInterval: 1, repeats: true) { _ in
    // TODO:
}
RunLoop.current.add(timer, forMode: .default)

let timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
    // TODO:
}

/// 子线程中
DispatchQueue.global().async {
    let timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) {_ in
       // TODO:
    }
    RunLoop.current.add(timer, forMode: .default)
    RunLoop.current.run()
}
```

因为 Timer 依赖 Runloop 的特殊性，所以其对应的 Target 释放不了的原因会包含两方面：

**Timer 与 Target 存在循环引用**

对于这个问题，我们通常会使用一个 [WeakProxy](https://github.com/Coder-Star/LTXiOSUtils/blob/master/LTXiOSUtils/Classes/Util/WeakProxy.swift) 来 `Warp` 一下`Target`。对于这个`WeakProxy`，我们需要注意重写一些父类方法。本质就是切断`Timer`对`Target`的强引用。
> 其实平时为了避免该问题，我们还是尽量使用`Block`这种形式。

```swift
import Foundation

/// WeakProxy 实现
final public class WeakProxy: NSObject {
    private weak var target: NSObjectProtocol?

    public init(_ target: NSObjectProtocol?) {
        self.target = target
        super.init()
    }

    public class func proxy(with target: NSObjectProtocol?) -> WeakProxy {
        return WeakProxy(target)
    }
}

extension WeakProxy {
    // 消息转发
    public override func forwardingTarget(for aSelector: Selector!) -> Any? {
        // 判断是否实现了Selector，如果实现了，就将消息转发给它
        if target?.responds(to: aSelector) == true {
            return target
        } else {
            return super.forwardingTarget(for: aSelector)
        }
    }

    public override func responds(to aSelector: Selector!) -> Bool {
        return target?.responds(to: aSelector) == true
    }

    public override func conforms(to aProtocol: Protocol) -> Bool {
        return target?.conforms(to: aProtocol) == true
    }

    public override func isEqual(_ object: Any?) -> Bool {
        return target?.isEqual(object) == true
    }

    public override var superclass: AnyClass? {
        return target?.superclass
    }

    public override func isKind(of aClass: AnyClass) -> Bool {
        return target?.isKind(of: aClass) == true
    }

    public override func isMember(of aClass: AnyClass) -> Bool {
        return target?.isMember(of: aClass) == true
    }

    public override var description: String {
        return target?.description ?? ""
    }

    public override var debugDescription: String {
        return target?.debugDescription ?? ""
    }
}
```

**Runloop 会强引用 Timer**

如果是一次性调用的 `Timer`(即`repeats`参数设置为`false`)，会在调用完毕之后自动 `invalidate` 掉自身，当然只执行一次这种使用场景也是比较少见。但是如果是多次重复调用的 `Timer`，就需要我们就需要我们自己在某个特定的时刻来 `invalidate` 掉 `Timer`，这个时刻一般是根据我们业务的实际情况去处理，常见的便是`deinit`时刻。

Timer 的实时性是不准确的，会存在延迟，这与 timer 所在的线程有很大关系，当线程在进行大量计算时，这期间有可能会错过很多次 NSTimer 的循环周期，但是 NSTimer 并不会将前面错过的执行次数在后面都执行一遍，而是继续执行后面的循环，也就是在一个循环周期内只会执行一次循环。无论循环延迟的多离谱，循环间隔都不会发生变化，在进行完大数据处理之后，有可能会立即执行一次 NSTimer 循环，但是后面的循环间隔始终和第一次添加循环时的间隔相同。

**引申**

当调用 NSObject 的 `performSelecter:afterDelay:` 方法，实际上其内部会创建一个 `Timer` 并添加到当前线程的 `RunLoop` 中。**所以如果当前线程没有 RunLoop，则这个方法会失效。**

`performSelector:onThread:`方法同理。

## CADisplayLink

## DispatchSourceTimer(GCD)

`mach_absolute_time`

## 最后

要更加努力呀！

Let's be CoderStar!
