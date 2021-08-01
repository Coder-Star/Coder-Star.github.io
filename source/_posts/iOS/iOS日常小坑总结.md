---
title: iOS 日常小坑总结
category:
  - iOS
tags:
  - iOS
  - 坑
date: 2021-05-27 14:51:08
---

- iOS 请求权限回调，第一次用户选择时回调会发生在非主线程，如果在回调中进行 UI 操作，请注意切换到主线程。

- Xib 设置设置颜色使用的色域是 Generic RGB，而 API 设置颜色以及吸色计等大部分都是使用的 sRGB，所以我们需要注意将 xib 的色域换成 sRGB，

- WkWebview 字体过小，注入一段 js `var meta = document.createElement('meta'); meta.setAttribute('name', 'viewport'); meta.setAttribute('content', 'width=device-width'); document.getElementsByTagName('head')[0].appendChild(meta);`

- UIButton 通过 textLabel 设置颜色、标题不生效，因为这两个属性跟按钮本身状态挂钩，UIButton 只会显示跟状态对应的标题和颜色。

- attributedText 设置后，text、lineBreakMode、textAlignment 等属性会被重置，虽然文档上说 font，textColor 也会被重置，但是根据我的测试，结果显示这两个属性不会被重置。

- UICollectionViewFlowLayout 设置`estimatedItemSize`后`sizeForItemAt`代理虽然会走，但是设置的值不会起效果

- UITextField 的 clearButtonMode 背后的删除按钮本身也是 UITextField rightView 的一种，如果手动设置了 rightView，clearButtonMode 的设置将不会生效。

- UITextField 直接设置 text，不会触发 editingChanged，如果需要实现触发目的，可以手动调用一下触发事件。

```swift
  field.text = nickName
  field.sendActions(for: .editingChanged)
```

- `selector-based` API 即 [addObserver(_:selector:name:object:)](https://developer.apple.com/documentation/foundation/notificationcenter/1415360-addobserver) 不再需要主动调用 `NotificationCenter.removeObserver(_:)` 方法（iOS 11.2 中测试）;
  `block-based` API 即 [addObserver(forName:object:queue:using:)](https://developer.apple.com/documentation/foundation/notificationcenter/1411723-addobserver) 还是需要主动调用 `NotificationCenter.removeObserver(_:)` 才能删除 callback Block（在 iOS 11.2 中测试）。

- UITableCell初始化时Width不管是什么机型都会是320，如果在该时间需要使用Width，需要手动设置一下Width；