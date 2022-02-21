---
title: Swift 关键字底层解析
category:
  - Swift
tags:
  - Swift
date: 2022-01-19 21:06:14
---

## 前言

Hi Coder，我是 CoderStar！

利用 SIL 文件，我们很容易分析出 Swift 的一些底层原理，

## Lazy

本质被 lazy 修饰的变量实际在底层是一个可选值，在内存的表现是 0x0，在第一次访问的过程中，访问的是 getter 方法，通过 enum 值的分支进行一个赋值的操作。

## static

底层通过`swift_once`保证线程安全，内部调用的就是 GCD 底层的`dispatch_once_f`。

## inout

虽然 & 符号可能会让你想起 C 和 Objective-c 中的取址操作符，或者是 C++ 中的引用传递操作符，但在 Swift 中，其作用是不一样的。就像对待普通的参数一样，Swift 还是会复制传入的 inout 参数，但当函数返回时，会用这些参数的值覆盖原来的值。也就是说，即使在函数中对一个 inout 参数做多次修改，但对调用者来说只会注意到一次修改的发生，也就是在用新的值覆盖原有值的时候。同理，即使函数完全没有对 inout 参数做任何的修改，调用者也还是会注意到一次修改 (willSet 和 didSet 这两个观察者方法都会被调用)。

## mutating

底层实际上使用了 inout 方式

## 其他

[A collection of well tested Swift Property Wrappers](https://github.com/guillermomuntaner/Burritos)

## 最后

要更加努力呀！

Let's be CoderStar!
