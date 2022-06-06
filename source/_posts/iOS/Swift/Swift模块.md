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

## 最后

要更加努力呀！

Let's be CoderStar!
