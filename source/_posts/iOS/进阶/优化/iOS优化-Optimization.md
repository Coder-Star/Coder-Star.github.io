---
title: iOS 优化 -Optimization
category:
  - 父分类
  - 子分类
tags:
  - 标签 1
  - 标签 2
date: 2021-11-26 22:40:18
---

## 前言

Hi Coder，我是 CoderStar！

## WMO

Swift 编译器专属优化。

在 Xcode 9.3（Swift 4.1 ） 之前对应`Build Setting`中的`SWIFT_OPTIMIZATION_LEVEL`为`Owholemodule`，在之后，把'编译模式'从'优化级别'里剥离了出来。也就是`SWIFT_OPTIMIZATION_LEVEL`中没有了`Owholemodule`，而是直接多出一个`Build Setting`项，为`SWIFT_COMPILATION_MODE`，选项为`incremental`与`wholemodule`；同时

[whole-module-optimizations](https://www.swift.org/blog/whole-module-optimizations/)

与此同时，`SWIFT_OPTIMIZATION_LEVEL`还多出一个`-Osize`选项。

[Code Size Optimization Mode in Swift 4.1](https://www.swift.org/blog/osize/)

Swift 会尽可能的优化派发方式，一些函数表派发方法会优化成直接派发。编译器可以通过 `whole module optimization` 进行一些全局优化。比如：
- 对某些没有标记 final 的类通过计算，如果能在编译期确定执行的方法，则使用直接派发。比如一个函数没有 override，Swift 就可能会使用直接派发的方式。
- 泛型特化；
  其

  ```swift
  func min<T:Comparable>(x:T, y:T) -> T {
    return y < x ? y : x
  }

  // 泛型特化后
  func min<Int>(x:Int, y:Int) -> Int {
    return y < x ? y :x
  }
  ```

- 移除没有被使用的方法；
- 内联优化；

## SIL 优化

- 泛型特化 (Generic Specialization)：需要 WMO 的支持
- 去虚拟化 (VTable Devirtualization)
- 性能内联
- 引用计数优化
- 内存提升 / 优化
- 高级域特定优化：在基本 Swift 容器（如 Array 或 String）上实现高级优化。[HighLevelSILOptimizations](https://github.com/apple/swift/blob/main/docs/HighLevelSILOptimizations.rst)

## LTO

LTO 能带来的优化有：
 - **将一些函数內联化**：不用进行调用函数前的压栈、调用函数后的出栈操作，提高运行效率与栈空间利用率；
 - **去除了一些无用代码**：如果一段代码分布在多个文件中，但是从来没有被使用，普通的 -O3 优化方法不能发现跨中间代码文件的多余代码，因此是一个局部优化。但是 `Link-Time Optimization` 技术可以在 link 时发现跨中间代码文件的多余代码；
 - **对程序有全局的优化作用**：这是一个相对广泛的概念。举个例子来说，如果一个 if 方法的某个分支永不可能执行，那么在最后生成的二进制文件中就不应该有这个分支的代码。

## 最后

要更加努力呀！

Let's be CoderStar!
