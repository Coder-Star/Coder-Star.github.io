---
title: iOS事件分发及响应链
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 事件分发
date: 2020-12-22 20:50:01
---

## iOS 事件分发及响应链机制

### 寻找最合适的 View

```swift
/// 寻找顺序
touch(UIEvent)->UIApplication事件队列->UIWindow->UIView->UIView的子view->...->view
```

在寻找最合适 View 过程中会用到 UIView 的下列两个方法；（注意一下，在寻找最合适 View 过程中 UIViewController 并没有参与进来)

```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
}

override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
}

```

当一个视图 View 收到 hitTest 消息时，先会检查自己是否可以响应事件，如果 View 的 userInteractionEnabled = NO，enabled = NO（UIControl），或者 alpha <= 0.01， hidden = YES 等情况的时候，直接返回 nil，然后调用自己的 poinInside 方法；如果返回 false 表示点击区域不在自己视图范围内，直接返回 nil。  
返回 nil 表示此 View 已经不是合适 View 了，如果不返回 nil 会遍历自己的子视图，所有子视图的遍历顺序是从最顶层视图一直到到最底层视图，即从 subviews 数组的末尾向前遍历，即后加入的子 view 会先遍历，子视图就会调用自己的 hitTest 方法；逐级进行进去，找到最小的那个 UIview。

tips  
在测试过程中，发现 hitTest 方法会执行两遍，point 值一致，根据 stackoverflow 上面的描述，苹果回复意思就是说 hitTest 是一个没有副作用的纯函数，进行多次调用也不会对外产生影响，因此系统可以多次调用调整 Point。

### 事件分发

在找到最合适的 View 后会进行事件分发。
UIApplication sendEvent: → UIWindow sendEvent: → 最合适的 view 开始响应

### 事件响应

事件响应的方式可以分别三种：UIResponder、UIGestureRecognizer、UIControl

#### UIResponder

子类包括：UIViewController、UIView、UIApplication

UIResponder 类中包含以下几个方法，用来响应事件，采用响应链进行传递。

```
– touchesBegan:withEvent:
– touchesMoved:withEvent:
– touchesEnded:withEvent:
– touchesCancelled:withEvent:
```

响应链是在事件分发寻找 View 中产生的响应链，最合适的 View 便是第一响应者，如果第一响应者不响应事件，便把这事件交由下一个响应者进行处理；

```swift
/// 根据事件类型调用对应方法，以touchBegan为例：
最合适的view touchesBegan: withEvent: → 所在ViewController touchesBegan: withEvent:→ parentViewViewController touchesBegan: withEvent: → ... → UIWindow touchesBegan: withEvent: → UIAplication touchesBegan: withEvent: → AppDelegate touchesBegan: withEvent: → 结束
/// 如果某个View或ViewController未调用super touchesBegan: withEvent:则响应结束
```

![响应链.webp](../../../img/响应链.png)

#### UIGestureRecognizer

手势识别器同样有 touch 的四个函数，但是手势识别器本身并不继承自 UIResponder，本身并不在响应链里，只有手势识别器对应的 view 在响应链中的时候手势识别器才会监听 touch 事件，并根据自己的 touch 函数识别手势，然后触发相应的回调函数。  
本质来说，hit-test view 触摸事件的回调跟手势识别器是两个独立的过程，互不干涉，手势识别器先开始接收 touch 事件。  
一般来说手势识别器的回调函数会比 hit-test view 的触摸事件的晚一些，因为手势识别器只有在手势识别出来之后才会触发回调函数（默认情况下只有一个手势识别器能够响应）.但是手势识别器接收 touch 事件的时机比 hit-test view 早。  
但是手势识别中定义了三个属性，能够影响 hit-test view 触摸事件的调用过程，这三个属性如下所示：

```swift
// 当值为YES时（默认值），表示手势识别成功后触摸事件取消掉，即识别成功后hitTest-View会调用touchesCancelled函数。
// 当值为NO时，触摸事件会正常起作用，会正常收到touchesEnded消息。
cancelsTouchesInView

// 当值为NO时（默认值），触摸事件和手势识别的过程同时进行，当然先会发送触摸事件，然后当手势识别成功时，触摸事件会被取消掉，即识别成功后hitTest-View会调用touchesCancelled函数。
// 当值为YES时，手势识别器先接收touch事件进行手势识别，识别过程中hit-test view的触摸事件会先被UIWindow hold住，当手势识别成功时hit-test view的触摸事件不会调用，当手势识别失败时才开始调用touchesBegan函数。
delaysTouchesBegan

// 当值为YES时（默认值），当手势识别失败时会延迟（约0.15ms）调用touchesEnded函数。
// 当值为NO时，当手势识别失败时会立即调用touchesEnded函数。
delaysTouchesEnded
```

### 3、UIControl

UIControl 会有自己的四个 Tracking 系列方法对应 touch 的四个方法，事实上，UIControl 的 Tracking 系列方法是在 touch 系列方法内部调用的。比如 beginTrackingWithTouch 是在 touchesBegan 方法内部调用的， 因此它虽然也是 UIResponder，但 touches 系列方法的默认实现和 UIResponder 本类还是有区别的。

UIControl 同时添加 action 以及绑定手势时，action 会响应，手势不响应；这貌似违背了手势优先级更高的原则，其实主要原因是 UIControl 是 UIResponder 中一个比较特殊的存在；

- 其在阻断响应链，UIControl 在事件处理的封装内并不会默认向 nextResponder 传递事件，而是自己拦截了这个事件进行处理
- 比父视图手势识别器的优先级高，UIControl 则在内部重写了 UIGestureRecognizer 的代理方法 gestureRecognizerShouldBegin:，使其不能够进行手势识别的状态转换，从而达到了「比父视图的 UIGestureRecognizer 优先级更高」的效果

如果想手势更优先响应，可应该手势的 delayTouchesBegan 设置为 true。
