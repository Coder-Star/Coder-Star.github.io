---
title: Swift泛型&协议&不透明类型&类型擦除
category:
  - Swift
tags:
  - Swift
date: 2022-06-15 21:39:54
---

## 前言

Hi Coder，我是 CoderStar！

- Protocols：
  协议本身具有隐藏实现细节以及运行时实例化的特性，故编译器、使用方无法知道其背后对应的真实类型；
  但，作为库的开发者 (代码是他写的)，明确知道 Protocol 背后可能对应的所有真实类型。
- Opaque Types：
  同 Protocols，库的开发者肯定是知道的；
  由于 Opaque Types 限制只能对应一种真实类型，并在编译期需明确，故编译器是知道的；
  对于使用方来说，他们看到的还是隐藏了细节的 Protocol。
- Generics：
  泛型是将类型决定权让给使用方的，故库的开发者是不知道真实类型的，而使用方知道；
  泛型属于编译期行为，故编译器能明确知道泛型对于的真实类型。
- Type Erasure：
  类型擦除属于使用方行为，用于规避编译错误等，故只有使用方知道。


不透明类型可以被称为 "反向泛型"。

## 最后

要更加努力呀！

Let's be CoderStar!


- [Swift Protocol 背后的故事(上)](http://zxfcumtcs.github.io/2022/02/01/SwiftProtocol1/)
- [opaque-return-types-and-type-erasure](https://www.raywenderlich.com/24942207-opaque-return-types-and-type-erasure)