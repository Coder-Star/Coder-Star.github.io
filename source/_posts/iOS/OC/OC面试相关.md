---
title: OC面试相关
category:
  - iOS
  - OC
tags:
  - iOS
  - OC
date: 2021-02-02 20:25:06
---

## [self class] 和 [super class]区别

两者打印出来的内容相同.

`[self class]`就是发送消息 `objc_msgSend`，消息接收者 `self`，方法编号 `class`
`[super class]`本质就是 `objc_msgSendSuper`，消息的接收者还是 `self`，方法编号 `class`。只是调用 `objc_msgSendSuper` 的时候会直接跳过 `self` 查找，直接在从 `super` 出现的在的方法所在的类的父类开始查找进行查找。

class 方法的作用就是返回 receiver 的类别。

## category & extension 区别

category：分类

- 给类添加新的方法
- 不能给类添加成员变量
- 通过@property 定义的变量，只能生成对应的 getter 和 setter 的方法声明，但是不能实现 getter 和 setter 方法，同时也不能生成带下划线的成员属性
- 是运行期决定的

extension：扩展

- 可以给类添加成员变量及方法，但是是私有的
- 添加的属性和方法是类的一部分，在编译期就决定的。在编译器和头文件的@interface 和实现文件里的@implement 一起形成了一个完整的类
- 伴随着类的产生而产生，也随着类的消失而消失
- 必须有类的源码才可以给类添加 extension，所以对于系统一些类，如 nsstring，就无法添加类扩展

## autoreleasepool 相关

autoreleasepool 底层数据结构为一个双向链表。

autoreleasepool 核心作用就是降低内存使用峰值。当你使用类似 for 循环这样的逻辑需要产生大量的中间变量时，Autorelease Pool 无意是最佳的一种解决方案；

> 主要就是一个类:AutoreleasePoolPage。两个函数: objc_autoreleasePoolPush()、objc_autoreleasePoolPop()。运作方式: autoreleasepool 由若干个 autoreleasePoolPage 类以双向链表的形式组合而成, 当程序运行到@autoreleasepool{时, objc_autoreleasePoolPush()将被调用, runtime 会向当前的 AutoreleasePoolPage 中添加一个 nil 对象作为哨兵,在{}中创建的对象会被依次记录到 AutoreleasePoolPage 的栈顶指针,
当运行完@autoreleasepool{}时, objc_autoreleasePoolPop(哨兵)将被调用, runtime 就会向 AutoreleasePoolPage 中记录的对象发送 release 消息直到哨兵的位置, 即完成了一次完整的运作。

**典型应用场景**

- 文件的读写处理
- 数据的批量处理, 包括图像裁剪生成, 音视频编解码
- 线程中 block 的回调处理

main函数中的autoreleasepool目前根据实际情况好像没什么用。
