---
title: Swift 模块
category:
  - Swift
tags:
  - Swift
date: 2022-05-27 23:54:44
---

## 前言

Hi Coder，我是 CoderStar！

## .swiftmodule



一般XXX.swiftmodule是一个目录，下面是相关的模块信息，会包括

- xxx.swiftmodule文件：二进制文件，就是对swift框架内部的方法声明， 包含序列化过后的 AST（抽象语法树 Abstract Syntax Tree）以及 SIL（Swift 中间语言 Swift Itermediate Language），Swift REPL工具可以对其进行解析。
- xxx.swiftinterface文件
- xxx.swiftdoc：二进制文件，保存对源代码的注释内容

- Project/xxx.swiftsourceinfo：二进制文件，是作为Swift源码的补充信息存在的，是对swiftdoc文件的补充，它的作用是用于定位Swift代码的行和列信息，在分发时候应该剔除。

[Proposal: emitting source information file during compilation](https://forums.swift.org/t/proposal-emitting-source-information-file-during-compilation/28794)


有组件 A 依赖组件 B，组件 B 依赖组件 C 在 Objective-C 中，B 对外暴露的头文件中引用了 C 的公开头文件，我们叫组件 B 传递依赖 C，结果就是编译组件 A 时必须同时能找到组件 B 和组件 C 的头文件，否则编译失败。

然而 Swift 并没有公开头文件一说，只要组件 B import C，导致 swiftmodule 中也明确标记了 import C，当组件 A import B 时，也同时 import C ，如果组件 A 找不到组件 C 的 module，那组件 A 将编译失败。

## 最后

要更加努力呀！

Let's be CoderStar!
