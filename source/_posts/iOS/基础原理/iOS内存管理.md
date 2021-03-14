---
title: iOS内存管理
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 内存管理
date: 2021-03-02 21:39:57
---

iOS 内存管理分为 MRC、ARC 两个方式，核心原理都是利用对象的引用计数来实现。MRC 是需要开发者自己手动去管理的对象的 retain 及 release，控制引用计数的变化。ARC 是编译时自动在程序对应的位置加上 retain 及 release。

当每个 runloo 完成一个循环后，都会检查对象的引用计数（retainCount），如果 retainCount 为 0，则释放该对象。

引用计数这种方式只面向 class 这类引用类型，不适用于值类型。

其中Swift固定使用ARC，而OC可以自己设置是否ARC，通过设置Xcode的`Build Settings`中的`将Objective-C Automatic Reference Counting`来控制OC是否开启ARC。

## 内存管理区

iOS内存管理区分为栈、堆、全局和静态变量存储区、常量存储区、代码区。这几个区内存地址由高到低。其中栈、堆在运行过程中会动态增长，其他在编译器确定。