---
title: iOS 优化 -UI
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2020-11-11 11:37:17
---

## UI 渲染优化

渲染优化目的就是减少 GPU 的运行压力

## 图片

### 原理

### 过程

- 从磁盘拷贝数据到内核缓冲区域；
- 从内核缓存区域复制数据到用户内存；
- UIImage 赋值给 UIImageView 的 image 时，图像数据会被解码，变成位图数，内存大小为 width height 4bytes （4bytes: 每个像素点的大小)；
- CTTransaction 捕捉到 UIImageView layer 树的变化；
- 主线程 runloop 提交 CTTransaction，开始图像渲染。（如果数据没有字节对齐，Core Animation 会再拷贝一份数据，进行字节对齐，GPU 处理位图数据，进行渲染）；

### 加载方式

- imageNamed：在图片第一次渲染到屏幕的时候触发解码，缓存解码之后的数据，缓存在全局内存中，不会随着 UIImage 的释放而释放。适合加载一些经常显示、比较小的图片，如 QQ 列表缩略图等；
- imageWithContentsOfFile: 或 imageWithData: 与 imageNamed 不同的是会随着 UIImage 的释放而释放。适合加载不常显示而且比较大的图片。

### 优化

- 减少加载 UIImage 内存的大小, 根据 imageview 实际 size 来加载，可以减少内存占用。解码后生成 CGImage 缩略图，再转化为 UIImage, 然后传给 UIImgaeView 渲染展示。
- 解码的操作在主线程，比较耗费 CPU 的资源。可以把耗时的解码操作放入子线程，解码完成之后再回调到主线程刷新，例如 SDWebImage。还有更加极限的优化是在子线程解码之后，将解码之后的图片存在磁盘之中，例如 FastImageCache。

### 尽量将图片大小设置的和 UIImageView 的大小一致

当对一个 UIImageView 设置一个较大的 image 时，需要在主线程完成重采样的过程，耗时耗内存，所以尽量在设置图片时设置的和 UIImageView 的大小相近。

### 图片异步解码

### 避免图层混合

当两个 View 叠加时，其中一个 View 含有透明度，那么叠加部分就会出现图层混合，GPU 需要消耗资源去计算最终显示的颜色。计算公式如下

```
R(C) = (1-alpha)*R(B) + alpha*R(A)
G(C) = (1-alpha)*G(B) + alpha*G(A)
B(C) = (1-alpha)*B(B) + alpha*B(A)
```

产生图层混合的场景

- 开启了图片 alpha 通道；
- View 的 alpha 值小于 1；

解决办法就是让产生图层混合的场景不存在。

对于不透明的 View，设置 opaque 为 YES，这样在绘制该 View 时，就不需要考虑被 View 覆盖的其他内容（尽量设置 Cell 的 view 为 opaque，避免 GPU 对 Cell 下面的内容也进行绘制）, 给 view 都设置一个背景色，避免使用默认的透明；如果想要透明的效果，可以将背景色设置与父 View 的背景色一致。

### 减少 subviews 的个数和层级

子控件的层级越深，渲染到屏幕上所需要的计算量就越大；如多用 drawRect 绘制元素，替代用 view 显示

### 避免离屏渲染

## 整体优化

- 不要使用太复杂的 XIB/Storyboard

## 其他

- GPU 能处理的最大纹理尺寸是 4096x4096，一旦超过这个尺寸，就会占用 CPU 资源进行处理，所以纹理尽量不要超过这个尺寸
- 尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示
