---
title: UIView相关
category:
  - iOS
  - UIView
tags:
  - iOS
  - UIView
date: 2020-11-12 13:44:33
---

## UIView.ContentMode枚举类型

``` swift
public enum ContentMode : Int {
  case scaleToFill = 0 // 缩放使图片填满整个容器，会改变图片本身的比例；默认值
  case scaleAspectFit = 1 // 以长边为标准，按照比例进行调整，容易会有留白
  case scaleAspectFill = 2 // 以短边为标准，按照比例进行调整，图片会显示不全
  case redraw = 3 // 尺寸变化时强制重绘， redraw on bounds change (calls -setNeedsDisplay)
  case center = 4 
  case top = 5
  case bottom = 6
  case left = 7
  case right = 8
  case topLeft = 9
  case topRight = 10
  case bottomLeft = 11
  case bottomRight = 12
}
```

一般UIImageView设置该属性比较多，目的是调整其显示图片的样式