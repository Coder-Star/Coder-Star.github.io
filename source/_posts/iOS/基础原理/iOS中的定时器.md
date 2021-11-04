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

/// 是否可用
open var isValid: Bool { get }

/// 触发，启动
open func fire()

/// 从当前Runloop移除timer
open func invalidate()
```

> 上述 interval 参数如果小于 0，则会使用 0.1 毫秒来代替。

从上述代码我们可以发现 Timer 生成实例的方式有八种，除了可以设置触发起止时间的两个之外，剩余六个为`Block`、`Target-Action`以及`NSInvocation`三种形式并且都提供`类方法`以及`构造函数`两种形式。我们可以根据自己的需求选择合适的构造函数。

1、`NSInvocation`在 Swift 中已经被禁止使用了，所以一般很少使用，如果非得使用需要借助 OC 进行中转；
2、`Block`方式是在 iOS 10 之后的，目的就是方便使用，并且避免了`Target-Action`这种方式惯有的循环引用问题，建议大家优先使用这种形式；
3、`Target-Action`方式最开始就存在，所以使用也比较频繁。

> `NSInvocation`的禁止其实也会影响到 `NSProxy` 在 Swift 中的使用，在 OC 中，我们一般会采用继承 `NSProxy`中的方式实现一个弱代理来解决常见的循环引用问题，比如常用的`YYKit`中的 [YYWeakProxy](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYWeakProxy.h)，但是在 Swift 中这种方式是不行的，需要继承`NSObject`来进行实现，这个具体后面会有介绍。

Timer 的运行需要依赖于 Runloop，如果 Timer 所处线程没有开启 Runloop，Timer 也是无法正常启动的，普通构造函数与类方法创建的 Timer 两者在这部分会有差异，`init`创建的 Timer 不会自动加入到 Runloop 中，需要再手动进行添加，而`scheduledTimer`形式会自动加入到当前线程对应的`Runloop`中。

> `scheduledTimer`形式自动加入的是`Runloop`的`default`模式，如果在滚动状态下仍然需要保持计时，则需要再手动加入到`eventTracking`或者`common`模式下，如果在子线程中，因为其 Runloop 没有默认启动，所以还是需要自己手动加入。
>
> 这样就涉及到一个比较典型的面试题：Timer 在滚动视图滚动时不进行响应。答案已经在上面了。

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

如果是一次性调用的 `Timer`(即`repeats`参数设置为`false`)，会在调用完毕之后自动 `invalidate` 掉自身，当然一次性调用这种使用场景也是比较少见，对此场景，我们可能更多的是使用`GCD`的`asyncAfter`方法。
但是如果是多次重复调用的 `Timer`，就需要我们就需要我们自己在某个特定的时刻来 `invalidate` 掉 `Timer`，这个时刻一般是根据我们业务的实际情况去处理，常见的便是`deinit`时刻。

**同时需要注意一定要在触发`Timer`的线程去进行`invalidate`，否则并不会终止。**

**引申**

当调用 NSObject 的 `performSelecter:afterDelay:` 方法，实际上其内部会创建一个 `Timer` 并添加到当前线程的 `RunLoop` 中。**所以如果当前线程没有 RunLoop，则这个方法会失效。**

`performSelector:onThread:`方法同理。

Timer 的定时并不是绝对精确，其取决于所在线程的空闲情况。当线程在进行大量计算时，这期间有可能会错过很多次 Timer 的循环周期，但是 Timer 并不会将前面错过的执行次数在后面都执行一遍，而是继续执行后面的循环，也就是在一个循环周期内只会执行一次循环。无论循环延迟的多离谱，循环间隔都不会发生变化，在进行完大数据处理之后，有可能会立即执行一次 Timer 循环，但是后面的循环间隔始终和第一次添加循环时的间隔相同。

Timer 会有一个`tolerance`属性 -- `时间宽容度`，指定该属性时意味着系统可以在原有时间附加该宽容度内的任意时刻触发 Timer。例如，如果你要 timer 1 秒后运行，并有 0.5 秒的时间宽容度，实际就可能是 1 秒，1.5 秒或 1.3 秒等等时刻执行。默认的时间宽容度是 0，但即使是 0，系统内部也会自动添加一个很小的宽容度。
这个属性是起到什么作用呢？按照开发者文档上的说法，设置该属性可以起到省电和优化系统响应性的作用。其可以允许系统协同运行多个 Timer，把多个 Timer 事件合并到一起，节省电池寿命。

从性能方面考虑，对于实时性要求不是特别高的`Timer`，我们都可以设置一下`tolerance`属性。并且我们应在保证需求前提下尽量少的设置定时器，比如可以定义全局定时器供各业务使用。
还有因为主线程需要处理UI相关的事情，所以我们将`Timer`注册到子线程对应的`Runloop`可从一定程度上保证其更加精确。

## CADisplayLink

`CADisplayLink`简单来说就是一个能让我们以和屏幕刷新率相同的频率将内容画到屏幕上的定时器，不过，与其说它是一个定时器，不如说它是一个观察者，其回调由事件触发而非计时器。简单描述就是设备屏幕每刷新一次，该对象绑定的方法就会调用一次。

`CADisplayLink`与`Timer`一样都需要`Runloop`的支持，并且虽然其没有被定义成`final`类型，但是开发者文档告知我们不应该继承`CADisplayLink`。

我们先来看下其属性及方法，注意查看注释。

```swift
open class CADisplayLink : NSObject {
  public /*not inherited*/ init(target: Any, selector sel: Selector)

  open func add(to runloop: RunLoop, forMode mode: RunLoop.Mode)

  /// 是否暂停
  /// 注意暂停不会将CADisplayLink销毁，只是不会再接收回调
  open var isPaused: Bool

  /// 只读值
  /// 当前帧时间戳
  open var timestamp: CFTimeInterval { get }

    /// 下一帧时间戳
  @available(iOS 10.0, *)
  open var targetTimestamp: CFTimeInterval { get }

  /// 以 maximumFramesPerSecond 条件下的双帧间的间隔时间
  open var duration: CFTimeInterval { get }

  /// 倾向帧刷新频率
  @available(iOS 10.0, *)
  open var preferredFramesPerSecond: Int

  /// 类似Timer，从Runloop的所有Mode中移除
  open func invalidate()
}
```

对其中几个属性补充一下：

- iOS 10 之后，虽然给出了`targetTimestamp`属性，用来标识下一帧预计的时间戳，开发者文档也描述可以通过`targetTimeStamp` - `timeStamp`得到两帧之间的差值，继而得到FPS，但是经过实际测试，这样得到的FPS一直是60HZ，根本不会波动，所以我们还是利用 YYKit 中的 [YYFPSLabel](https://github.com/ibireme/YYKit/blob/master/Demo/YYKitDemo/YYFPSLabel.m) 这种方式来获取真实的FPS，下方会给出Swift代码示例。
- `preferredFramesPerSecond`这个属性为 **首选** 帧速率，表示设备每秒回调的帧数。每个设备都会有一个屏幕最大刷新频率的物理属性，大部分 iPhone 都是 60Hz，iPad pro 是 120Hz，我们可以利用`UIScreen.main.maximumFramesPerSecond`获取到。`preferredFramesPerSecond`默认值为 0，此时会按照最大刷新频率进行回调，我们也可以自定义设置的，但需要注意设置的值需要为最大刷新频率的因子，如 20、30 等（当然也不能设置的超过`maximumFramesPerSecond`），如果设置的值不为因子，则系统内部会将不是最大帧速率的除数的首选帧速率四舍五入到最接近的因子，比如说 60HZ 的设备上设置其为 26 或者 35，则内部会自动调整为 30，这也是命名中有`preferred`的原因；
- `duration`表示以设备以`maximumFramesPerSecond`回调时两帧之间的差值，不受`preferredFramesPerSecond`的影响。那 `duration` 与 `targetTimeStamp` - `timeStamp` 之间有什么区别呢？比如 60HZ 的设备上，`preferredFramesPerSecond`设置为 30，那 `duration` 还是 `1 / 60`，而`targetTimeStamp` - `timeStamp`为 `1 / 30`。

从上面代码中我们也可以看出来，`CADisplayLink`属性与方法比较少，使用起来也比较简单。其相对 Timer 来说使用场合相对专一，适合做 UI 的不停重绘，比如自定义动画或者视频播放的渲染，还有我们平时最常见的就是获取`FPS`，下面给出示例。

```swift
final class FPSUtils {
    public static let shared = FPSUtils()

    private init() {}

    private var link: CADisplayLink?

    /// 记录1s内回调次数
    private var countPerSecond: Int = 0
    /// 上传回调时间戳
    private var lastTimestamp: TimeInterval = 0

    @objc
    private func tick(_ link: CADisplayLink) {
        guard lastTimestamp != 0 else {
            lastTimestamp = link.timestamp
            return
        }
        countPerSecond += 1

        let timePassed = link.timestamp - lastTimestamp
        guard timePassed >= 1 else {
            return
        }

        /// 获取FPS
        let fps = TimeInterval(countPerSecond) / timePassed
        Log.d(fps)

        lastTimestamp = link.timestamp
        countPerSecond = 0
    }
}

// MARK: - 公开方法

extension FPSUtils {
    public func start() {
        stop()
        link = CADisplayLink(target: self, selector: #selector(tick(_:)))
        /// Mode 为 common， 避免滚动时不计算
        link?.add(to: RunLoop.main, forMode: .common)
    }

    public func stop() {
        link?.invalidate()
        link = nil
    }
}
```

`Timer` 与 `CADisplayLink` 背后都跟 `Runloop` 息息相关，后续会对 `Runloop` 进行单独介绍，这里就不单独展开了，那`Timer` 与 `CADisplayLink`之间有什么区别呢？

- 设置周期方式不同：一个通过`preferredFramesPerSecond`进行间接设置，一个直接通过`timeInterval`参数设置，后者更直接一些；
- 灵敏度不同：`CADisplayLink`受限于`maximumFramesPerSecond`的限制，不可以超过，而 Timer 的受限则是`Runloop`的周期，其灵敏度相对`CADisplayLink`大的多；
- 适用场景不同：`CADisplayLink`直接跟渲染挂钩，更适合用在 UI 方便，比如平滑动画或者上述提到的 FPS，而`Timer`使用范围则更大一些。
...

## DispatchSourceTimer(GCD)

`mach_absolute_time`

## 最后

要更加努力呀！

Let's be CoderStar!

参考资料

- [Timer使用指南](http://www.cocoachina.com/articles/23890)
