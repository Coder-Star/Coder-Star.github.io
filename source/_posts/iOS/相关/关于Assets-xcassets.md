---
title: 关于Assets.xcassets
category:
  - iOS
  - 相关
  - Assets.xcassets
tags:
  - iOS
  - Assets.xcassets
date: 2020-11-14 11:23:12
---

先放置一个Assets.xcassets中图片的操作选项

![Assets.xcassets](../../../img/iOS/相关/Assets.png)


## Render As (渲染模式)

包含三个选项，分别为

* Default  
  默认值，基于上下文决定具体的渲染模式. 如果是在navigation bars, tab bars, toolbars, 和segmented controls中则为Template, 其余上下文则为Original
* Original Image  
  按照原图渲染
* Template Image  
  模板渲染，系统会忽略图像的颜色信息，并根据图像中的alpha值创建图像模板，图像中alpha值 < 1.0的部分被视为完全透明，alpha值 = 1.0的部分为Image Tint的颜色，默认Default为蓝色。

## 放置矢量图

将Scales设置为Single Scale

苹果在Xcode6中便允许我们在项目中利用Assets.xcassets使用矢量图来代替传统的位图，但是实际上Xcode在编译时候会根据根据矢量图来自动生成对应的@1x、@2x和@3x格式的位图。这种方式并不能减小包的体积；

勾选Preserve Vector Data选项时，会在编译期间将矢量图文件拷贝到二进制文件中，以便我们可以在运行时任意缩放图片而不会失真