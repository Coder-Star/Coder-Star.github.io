---
title: UIImage相关
category:
  - iOS
  - UI
tags:
  - iOS
  - UIImage
date: 2020-11-09 18:30:02
---

创建 UIImage 的形式有两种

```swift
// 简称 named 方法
public init?(named name: String)
// 简称 contentsOfFile 方法
public init?(contentsOfFile path: String)
```

## 图片缓存

- named 方法创建的 UIImage 会进行缓存，其创建 UIImage 的步骤为：根据图片文件名在缓存池中查找特定的 UIImage 对象，如存在，将这个对象返回。如果不存在，则从 bundle 中加载图片数据，创建对象并返回。如果相应的图片数据不存在，返回 nil。
  这种方式比较适合存放系统常用的，占用内存小的图片资源；

- contentsOfFile 方法不会进行缓存；这种方式适合不常使用，占用内存比较大的图片；

## Assets.xcassets

Assets.xcassets 在 app 打包后，以 Assets.car 文件的形式出现在 bundle 中。其作用在于：

- 自动识别@2x，@3x 图片，对内容相同但分辨率不同的图片统一管理
- 可以对图片进行剪裁和拉伸处理

Assets.xcassets 中的图片资源只可以使用 named 方法进行加载，通过 contentsOfFile 方法加载无法获取到具体的图片路径，无法加载成功。

## jpg 以及 png 图片

当不以 Assets.xcassets 的方式加载图片时，png 相对于其他格式有不同的地方

- png 图片名称只需是文件名即可，不需加上后缀格式
- 其他格式 必须加上对应的后缀格式，否则加载不成功
