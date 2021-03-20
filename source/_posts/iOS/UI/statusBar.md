---
title: statusBar
category:
  - iOS
  - statusBar
tags:
  - iOS
  - statusBar
  - 状态栏
date: 2021-01-26 19:59:00
---

**info.list中的配置**
Status bar is initially hidden // 设置全局状态栏是否隐藏

View controller-based status bar appearance // 设置VC对其所在页面状态栏是否进行控制，当为YES时，则以VC设置为准，当为NO，则以上述配置为准，设置为YES后，拥有的相关设置如下：
```swift
override var prefersStatusBarHidden: Bool
override var preferredStatusBarStyle: UIStatusBarStyle
// UINavigationController使用
override var childForStatusBarStyle: UIViewController?
override var childForStatusBarHidden: UIViewController? 
setNeedsStatusBarAppearanceUpdate()  // 这个方法会通知系统去调用当前UIViewController的preferredStatusBarStyle方法
```
