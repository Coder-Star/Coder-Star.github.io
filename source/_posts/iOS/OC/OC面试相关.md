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


## block

block本质上也是一个OC对象，它内部也有一个isa指针，封装了函数调用以及函数调用环境的OC对象；