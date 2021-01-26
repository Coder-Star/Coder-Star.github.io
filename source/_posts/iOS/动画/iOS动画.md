---
title: iOS动画
category:
  - iOS
  - 动画
tags:
  - iOS
  - 动画
date: 2020-11-12 10:59:35
---

## CGAffineTransform

```swift
let view = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))

// 缩放出显示出的大小为50*50
view.transform = CGAffineTransform.identity.scaledBy(x: 0.5, y: 0.5)
// 缩放出显示出的大小为200*200，而不是 50*50 的2倍
// 原因使用 CGAffineTransform.identity 这种方式进行缩放是基于一个常量进行缩放的，这个常量为view真实的frame
view.transform = CGAffineTransform.identity.scaledBy(x: 2, y: 2)
```
