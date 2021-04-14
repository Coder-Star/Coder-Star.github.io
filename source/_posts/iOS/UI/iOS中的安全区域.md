---
title: iOS中的安全区域
category:
  - iOS
  - UI
tags:
  - iOS
  - 安全区域
date: 2021-04-14 18:31:54
---

刘海屏之前一些位置的高度基本上是固定的，如状态栏，导航栏、标签栏等。相关高度如下：

- 状态栏（statusBar）：20pt
- 导航栏（navigationBar）：40pt
- 标签栏（tabBar）：49pt

但是随着刘海屏的出现，顶部及底部都多出一块区域，并且这块区域相对而言不是那么固定，即状态栏、标签栏高度会发生改变，导航栏高度不变，苹果也推荐利用更好的方式去布局。即安全区域（safeArea）。对于一个 VC 的 root view,safeArea 指的是未被状态栏、一些可见的 bars（navigationBar、tabBar、toolBar）、和开发者通过 additionalSafeAreaInsets 属性设置的值遮盖的区域。刘海屏部分导致状态栏变成了 44pt，底部 tabBar 要在自身高度的基础上再为 home indicator 留出 34pt 边距。

安全区域概念 API 限制在 iOS 11 及之上，两个关键 API 为 UIView 新增的两个属性：

- safeAreaInsets
- safeAreaLayoutGuide

其中 safeAreaInsets 适用获取到具体尺寸，也就是 frame 布局方式，如果适用 AutoLayout 方式布局，可以使用 safeAreaLayoutGuide 方式。

Tips：VC 的生命周期为`viewDidLoad ->viewWillAppear->viewSafeAreaInsetsDidChange->viewWillLayoutSubviews->viewDidLayoutSubviews->viewDidAppear`，在`viewSafeAreaInsetsDidChange` 方法时界面的 safeAreaInsets 值会被计算出来，我们可以在这个方法中去调整位置，所以我们不可以在`viewDidLoad`方法中利用 safeAreaInsets 去进行布局。但是我们可以在该生命周期中利用 safeAreaLayoutGuide 设置约束。

在 iOS 11 以前，我们进行约束布局通常使用 topLayoutGuide 及 bottomLayoutGuide，但是 iOS 11 之后被 safeAreaLayoutGuide 取代，下面的例子说明一下使用方法。

```swift
baseView = UIView()
view.addSubview(baseView)
baseView.snp.makeConstraints { make in
    if #available(iOS 11.0, *) {
      make.top.equalTo(view.safeAreaLayoutGuide.snp.top)
      make.bottom.equalTo(view.safeAreaLayoutGuide.snp.bottom)
    } else {
      make.top.equalTo(topLayoutGuide.snp.bottom)
      make.bottom.equalTo(bottomLayoutGuide.snp.top)
    }
    make.left.right.equalToSuperview()
 }
```

伴随着安全区域概念的出现，UIViewController 的 automaticallyAdjustsScrollViewInsets 属性也被 UIScrollView 的 contentInsetAdjustmentBehavior 属性所替代。contentInsetAdjustmentBehavior 属性的四种类型可以自己去查看一下注释。
