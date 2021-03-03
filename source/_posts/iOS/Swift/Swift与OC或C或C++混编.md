---
title: Swift与OC或C或C++混编
date: 2020-11-06 11:30:06
category:
  - Swift
tags: [iOS, OC, Swift, C, C++]
---

## Swift调用其他语言

* Swift调用OC或者Swift调用C的方式都是通过桥接文件的形式。
* Swift调用C++是无法直接调用的，需要依赖于C或者OC语言中转一下

当依赖OC语言来中转调用C++，则OC的.m文件后缀需要改为.mm文件，显式告诉编译器文件内部会包含C++代码。并且需要注意是.mm文件中不允许引入Swift生成的"Product Name-Swift.h"文件，如果引入，该.h文件会报错。


**Swift-C 数据类型对应**
![Swift-C数据类型对应](../../../img/iOS/Swift/Swift-C数据类型对应.png)
