---
title: iOS面试题积累
date: 2019-10-16 14:29:09
categories: [iOS]
tags: [iOS]
---




## UIImageView 加载图片相关

**过程**

- 从磁盘拷贝数据到内核缓冲区域；
- 从内核缓存区域复制数据到用户内存；
- UIImage 赋值给 UIImageView 的 image 时，图像数据会被解码，变成位图数，内存大小为 width height 4bytes （4bytes:每个像素点的大小)；
- CTTransaction 捕捉到 UIImageView layer 树的变化；
- 主线程 runloop 提交 CTTransaction，开始图像渲染。（如果数据没有字节对齐，Core Animation 会再拷贝一份数据，进行字节对齐，GPU 处理位图数据，进行渲染）；

**加载方式**

- imageNamed：在图片第一次渲染到屏幕的时候触发解码，缓存解码之后的数据，缓存在全局内存中，不会随着 UIImage 的释放而释放。适合加载一些经常显示、比较小的图片，如 QQ 列表缩略图等；
- imageWithContentsOfFile: 或 imageWithData: 与 imageNamed 不同的是会随着 UIImage 的释放而释放。适合加载不常显示而且比较大的图片。

**优化**

- 减少加载 UIImage 内存的大小,根据 imageview 实际 size 来加载，可以减少内存占用。解码后生成 CGImage 缩略图，再转化为 UIImage,然后传给 UIImgaeView 渲染展示。
- 解码的操作在主线程，比较耗费 CPU 的资源。可以把耗时的解码操作放入子线程，解码完成之后再回调到主线程刷新，例如 SDWebImage。还有更加极限的优化是在子线程解码之后，将解码之后的图片存在磁盘之中，例如 FastImageCache。


## 循环引用

循环引用就是两个及以上的对象出现了引用环，导致对象都无法得到释放，典型场景一般包括 timer，block 以及 delegate；

**block、delegate**

一般使用 Weak、unowned 修饰可以解决问题，unowned 要比 weak 少一些性能消耗。
block 产生循环引用的原因是闭包表达式对用到的外层对象产生额外的强引用。

```
[weak self]  =>   __weak typeof(self) weakSelf;
[unowned self] =>  __unsafe_unretained typeof(self) weakSelf;
```

## iOS 自身的设计模式

- 代理模式：tableview 的 delegate、datasource
- 观察者模式：KVO，Notification
- 单例模式：UserDefault，UIApplication，FileManager，NotificationCenter，URLCache，HTTPCookieStorage
- 装饰模式：OC的category
