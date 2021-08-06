---
title: UIView & CALayer
category:
  - iOS
  - UI
tags:
  - iOS
  - UIView
date: 2021-08-02 21:57:04
---

## 前言

Hi Coder，我是 CoderStar！

今天我们来聊一聊 UIView 与 CALayer 的相关知识以及它们之间的关系。

## UIView 与 CALayer

CALayer 继承自 NSObject, 负责图像渲染；
UIView 继承自 UIResponder, 主要负责事件响应；

UIView本身是不具备显示功能的，
 拥有一个 layer 属性用来持有一个 CALayer 成员，我们平时操作的 UIView 的绝大部分绘图属性内部其实都是操作其拥有的 layer 属性，比如 frame、hidden、mask 等。

```swift
open class UIView : UIResponder
  open var layer: CALayer { get }
  open class var layerClass: AnyClass { get }
}
```

CALayer 通常是做为 UIView 类的属性进行使用，但也可以单独使用。实际上 CALayer 是属于 QuartzCore 框架，而这个框架是一个跨平台的绘制框架。这里的跨平台指的是在 iOS 和 macOS 系统上均能使用，也就是说 CALayer 能在 iOS 和 macOS 上面绘制内容（这也说明了为什么 CALayer 的很多属性都不是 UIKit 框架下面的东西，比如它的 backgroundColor 是 CGColorRef，因为 macOS 中没有 UIKit）。但是这两个平台接收用户交互的方式完全不一样：iOS 是通过触摸事件（touch event）而 macOS 则是监听鼠标和键盘事件。

UIView 负责处理用户交互，负责绘制内容的则是它持有的那个 CALayer，我们访问和设置 UIView 的这些负责显示的属性实际上访问和设置的都是这个 CALayer 对应的属性，UIView 只是将这些操作封装起来了而已。

Layers 能够有子 Layers，就像视图可以有子视图一样。
Layers 能够动画化。当你想改变一个 Layers 的属性时后，你可以使用 CAAnimation 去为这些改变增加动画效果
Layers 相对 View 来说是轻量的

- [Understanding UIKit Rendering](https://developer.apple.com/videos/play/wwdc2011/121/)

[YYAsyncLayer](https://github.com/ibireme/YYAsyncLayer)

## frame、bounds、center、transfrom、

frame 与 bounds 都是一个 CGReact 结构，有 origin 以及 size 组成，即原点与尺寸组成。
而 center 则是一个 CGPoint 结构，只是单纯的一个点。

其中 frame 是相对父 View 的坐标系而言的。
bounds 是相对自身的坐标系而言的。
center 也是相对父 View 的坐标系而言的。

UIView 中，frame 其实是不存储的，而是动态计算的，改变 center，改变 bounds 大小，或者改变 transfrom 都可能会导致 frame 的改变。 也就是说 frame 是计算属性。

实际上 center 才是决定 UIView 位置的决定因素，但是不会改变其大小。

> iOS 和 macOS 两个系统的参考坐标系不一致，对于 iOS 来说原点默认在视图的左上角位置，而对于 macOS 来说原点默认是在视图的左下角位置。

```swift
var frame: CGReact: {
  get {
    return CGRect(
        x: center.x - bounds.size.width / 2,
        y: center.y - bounds.size.height / 2,
        width: bounds.size.width,
        height: bounds.size.height
      )
  }
  set {
    bounds.origin = 0,0
    bounds.size = frame.size

    center.x = frame.origin.x + frame.size.width / 2
    center.y = frame.origin.y + frame.size.height / 2
  }
}
```

transform 不会影响到 view 的 bounds 和 center，但是会影响到 frame

bounds.origin 一定是 (0, 0) 吗?
并不是。只是因为 bounds 的默认值为 ((0, 0), frame.size)，而且通常情况下没有修改 bounds.origin 的必要。bounds 作为对 view 自身坐标系的描述，修改 bounds.origin 会影响其 subview 的位置。

frame 不管对于位置还是大小，改变的都是自己本身。

bounds 改变位置时，改变的是子视图的位置，自身没有影响；其实就是改变了本身的坐标系原点，默认本身坐标系的原点是左上角； bounds 的大小改变时，当前视图的中心点不会发生改变，当前视图的大小发生改变，看起来效果就想缩放一样；

## 最后

新的一周要更加努力呀！

Let's be CoderStar!
