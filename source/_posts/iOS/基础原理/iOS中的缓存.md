---
title: iOS 中的缓存
category:
  - 基础原理
tags:
  - 基础原理
  - 缓存
date: 2021-10-13 09:41:35
---

## 前言

Hi Coder，我是 CoderStar！

## NSCache

NSCache 删除缓存中的对象的场景

* NSCache 缓存对象自身被释放
* 手动调用 removeObjectForKey: 方法
* 手动调用 removeAllObjects
* 缓存中对象的个数大于 countLimit，或，缓存中对象的总 cost 值大于 totalCostLimit
* 程序进入后台后
* 收到系统的内存警告
* 同一个 key 更新对象时会将之前的对象删除

> 在研究这个的时候意外发现需要注意的一个点，就是Apple开源的[swift-corelibs-foundation](https://github.com/apple/swift-corelibs-foundation)实现与操作系统内置的`Foundation`实现并不是完全一样。
> 比如在`swift-corelibs-foundation`里面，其删除缓存的机制为先删除`cost`值比较小的。
> 而系统内置的`Foundation`，实际上用的是 LRU 算法。

## 最后

要更加努力呀！

Let's be CoderStar!
