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

## Extension & Category：分类

extension：扩展

- 可以给类添加成员变量及方法
- 添加的属性和方法是类的一部分，在编译期就决定的。在编译器和头文件的@interface 和实现文件里的@implement 一起形成了一个完整的类
- 伴随着类的产生而产生，也随着类的消失而消失
- 必须有类的源码才可以给类添加 extension，所以对于系统一些类，如 NSString，就无法添加类扩展

category：分类

- 给类添加新的方法
- 不能给类添加成员变量
- 通过@property 定义的变量，只能生成对应的 getter 和 setter 的方法声明，但是不能实现 getter 和 setter 方法，同时也不能生成带下划线的成员属性
- 是运行期决定的，这也是没法添加成员变量的原因，因为在运行期，对象的内存布局已经确定，所以不允许再添加成员变量去修改内存布局。

分类同名方法会优先主类的方法使用。
多个分类中的同名方法会只执行一个,即后编译的分类里面的方法会覆盖所有前面的同名方法。只调用 category 中方法的原因是：runtime 加载某个类的所有分类数据，将分类中的方法、属性、协议数据都合并到一个大数组中。而由于是倒序的方式遍历，所以后面参与编译的 Category 数据会在数组的前面。最后将合并后的分类数据插入到类原来数据的前面。

## [self class] 和 [super class]区别

两者打印出来的内容相同.

`[self class]`就是发送消息 `objc_msgSend`，消息接收者 `self`，方法编号 `class`
`[super class]`本质就是 `objc_msgSendSuper`，消息的接收者还是 `self`，方法编号 `class`。只是调用 `objc_msgSendSuper` 的时候会直接跳过 `self` 查找，直接在从 `super` 出现的在的方法所在的类的父类开始查找进行查找。

class 方法的作用就是返回 receiver 的类别。

## load 方法的加载顺序

父类的+load 方法执行在前，子类的+load 方法在后
所有类的+load 方法执行在前，所有分类的+load 方法执行在后
同级别类及分类的执行顺序与编译顺序有关系。

load 函数是 main 函数启动前装载文件直接通过函数地址调用，而不是使用消息转发的方式，只被调用一次。

initialize：当类或子类第一次收到消息时被调用只调用一次，调用方式是通过 runtime 的 objc_msgSend 的方式调用的，此时所有的类都已经装载完毕，子类和父类同时实现 initialize，父类的先被调用，然后调用子类的。本类与 category 同时实现 initialize，category 会覆盖本类的方法，只调用 category 的。

load 方法里面可以调用 category 中声明的方法，因为附加 category 到类的工作会先于+load 方法的执行。

## OC 中的 block

OC 中的 block 本质是一个 OC 对象，内部也会有一个 isa 指针。有三种类型。

- 全局 block：**NSGlobalBlock** ，存储在静态区
- 堆 block：**NSMallocBlock** ，存储在堆区
- 栈 block：**NSStackBlock** ，存储在栈区

* 全局 block：在 ARC 及 MRC 下，只要不使用 auto 变量，就是全局 block；copy 之后还是全局 block；
* 栈 block：在 MCR 下使用了 auto 变量，就是栈 block，arc 在有些情况下编译器会自动将栈上的 block 转换为堆 block。
* 堆 block：在 MRC 下将栈 block 进行 copy 操作，就会得到堆 block。堆 block 进行 copy 之后引用计数+1。

> auto 变量是它所在的区域执行完毕,自动销毁的变量。

ARC 下，栈 block 自动转为堆 block 的情况

- block 作为函数返回值的时候
- 将 block 赋值给\_\_strong 指针的时候
- block 作为 Cocoa API 中方法名含有 usingBlock 的方法参数时
- block 作为 GCD API 的方法参数时

\_\_block 作用：1、解决 block 内部想修改外部 auto 变量的问题；2、解决循环引用；

## copy / mutableCopy

- 不可变对象调用 copy，不会生成新的对象，因为没有必要。指针指向同一地址即可满足。
- 不可变对象调用 mutableCopy，生成新的对象，因为需要修改而原对象不支持修改。
- 可变对象调用 copy，生成新的对象，因为预期是获得一个不可变对象。
- 可变对象调用 mutableCopy，生成新对象，可变对象的修改不应影响到原对象，所以会生成新对象。
