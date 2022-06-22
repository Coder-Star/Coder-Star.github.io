---
title: Swift 底层
category:
  - Swift
tags:
  - Swift
date: 2022-05-29 22:10:52
---

## 前言

Hi Coder，我是 CoderStar！

## protocol

protocol 底层默认存储方式为：Existential Container。
> 因为协议可以被很多结构（class、struct）所实现，所以需要一个额外的结构去承接。

- 前三个字节：Value buffer。用来存储 Inline 的值，如果 word 数大于 3，则采用指针的方式，在堆上分配对应需要大小的内存；
- 第四个字节：Value Witness Table(VWT)。每个类型都对应这样一个表，用来存储值的创建，释放，拷贝等操作函数。(管理 Existential Container 生命周期)
- 第五个字节：Protocol Witness Table(PWT)，用来存储协议的函数。

## 最后

要更加努力呀！

Let's be CoderStar!
