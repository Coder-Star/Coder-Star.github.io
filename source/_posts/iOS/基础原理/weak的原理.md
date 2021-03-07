---
title: weak的原理
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - weak
date: 2021-03-02 21:31:08
---

weak 是弱引用，用 weak 描述修饰或者所引用的对象，引用计数不会加以，并且在引用的对象释放时候自动被设置为 nil。利用这种特性，可以很方便的解决循环引用，也避免了野指针访问导致崩溃的情况。

- Swift 中的 weak 相当于 OC 里面的 weak；
- Swift 中的 unowned 相当于 OC 里的 \_\_unsafe_unretained / assign

OC 的 weak 实现和 Swift 还有些区别；

## OC 中 Weak 原理

Runtime 维护了一个 weak 表，用于存储指向某个对象的所有 weak 指针。weak 表其实是一个 hash 表，Key 是所指对象的地址，value 是 weak 指针的地址数组，是数组的原因是一个对象可能被多个弱引用指针指向。

1. 初始化时：runtime 会调用 objc_initWeak 函数，初始化一个新的 weak 指针指向对象的地址。
2. 添加引用时：objc_initWeak 函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3. 释放时，调用 clearDeallocating 函数。clearDeallocating 函数首先根据对象地址获取所有 weak 指针地址的数组，然后遍历这个数组把其中的数据设为 nil，最后把这个 entry 从 weak 表中删除，最后清理对象的记录。

weak 对象自动被置为 nil 的流程

1. 对象的引用计数为 0 时，执行 dealloc
2. 在 dealloc 中，调用了\_objc_rootDealloc 函数
3. 在\_objc_rootDealloc 中，调用了 object_dispose 函数,该函数内部作用如下：
   1. 为 C++ 的实例变量们（iVars）调用 destructors
   2. 为 ARC 状态下的 实例变量们（iVars） 调用 -release
   3. 解除所有使用 runtime Associate 方法关联的对象
   4. 解除所有 \_\_weak 引用
   5. 调用 free()
4. 调用 objc_destructInstance
5. 最后调用 objc_clear_deallocating
   1. 从 weak 表中获取废弃对象的地址为键值的记录
   2. weak 修饰符变量的地址，赋值为 nil(weak_clear_no_lock 方法)
   3. 将 weak 表中该记录删除
   4. 从引用计数表中删除废弃对象的地址为键值的记录

## Swift 中 Weak 原理
