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

## category & extension

category：分类

- 给类添加新的方法
- 不能给类添加成员变量
- 通过@property 定义的变量，只能生成对应的 getter 和 setter 的方法声明，但是不能实现 getter 和 setter 方法，同时也不能生成带下划线的成员属性
- 是运行期决定的

分类同名方法会覆盖主类的方法。
多个分类中的同名方法会只执行一个,即后编译的分类里面的方法会覆盖所有前面的同名方法。

extension：扩展

- 可以给类添加成员变量及方法，但是是私有的
- 添加的属性和方法是类的一部分，在编译期就决定的。在编译器和头文件的@interface 和实现文件里的@implement 一起形成了一个完整的类
- 伴随着类的产生而产生，也随着类的消失而消失
- 必须有类的源码才可以给类添加 extension，所以对于系统一些类，如 nsstring，就无法添加类扩展

## block

block 本质上也是一个 OC 对象，它内部也有一个 isa 指针，封装了函数调用以及函数调用环境的 OC 对象；分为三种类型

## load 方法的加载顺序

父类的+load 方法执行在前，子类的+load 方法在后
所有类的+load 方法执行在前，所有分类的+load 方法执行在后
同级别类及分类的执行顺序与编译顺序有关系。

load 函数是 main 函数启动前装载文件直接通过函数地址调用，而不是使用消息转发的方式，只被调用一次。

initialize：当类或子类第一次收到消息时被调用只调用一次，调用方式是通过 runtime 的 objc_msgSend 的方式调用的，此时所有的类都已经装载完毕子类和父类同时实现 initialize，父类的先被调用，然后调用子类的。本类与 category 同时实现 initialize，category 会覆盖本类的方法，只调用 category 的。

## OC 中的 block

OC 中的 block 本质是一个 OC 对象，内部也会有一个isa指针。有三种类型。

- 全局 block：**NSGlobalBlock** ，存储在静态区
- 堆 block：**NSMallocBlock** ，存储在堆区
- 栈 block：**NSStackBlock** ，存储在栈去

* 全局 block：在 ARC 及 MRC 下，只要不使用外部变量，就是全局 block；copy 之后还是全局 block；
* 栈 block：在 MCR 下使用了外部变量，就是栈 block，arc 下没有栈 block，因为编译器会自动将栈上的block复制到堆block。
* 堆 block：在 ARC 下，只要使用了外部变量，就是堆 block，在 MRC 下将栈 block 进行 copy 操作，就会得到堆 block。堆 block 进行 copy 之后引用计数+1。


```

```