---
title: dyld迭代
category:
  - 进阶
tags:
  - dyld
date: 2022-01-09 14:16:43
---

## 前言

Hi Coder，我是 CoderStar！


## dyld 迭代

我们对`dyld`展开讲一下，其是开源的，我们可以[官网下载](https://opensource.apple.com/tarballs/dyld/)后阅读其源码。

一、dyld 1.0（1996-2004）

二、dyld 2（2004-2017）

三、dyld 3（2017- 至今）

dyld2 和 dyld3 的加载方式略有不同。dyld2 是纯粹的 in-process，也就是在程序进程内执行的，也就意味着只有当应用程序被启动的时候，dyld2 才能开始执行任务。dyld3 则是部分 out-of-process，部分 in-process。

dyld2 的过程是：加载 dyld 到 App 进程，加载动态库（包括所依赖的所有动态库），Rebase，Bind，初始化 Objective C Runtime 和其它的初始化代码。

dyld3 的 out-of-process 会做如下事情：分析 Mach-O Headers，分析依赖的动态库，查找需要 Rebase & Bind 之类的符号，把上述结果写入缓存。这样，在应用启动的时候，就可以直接从缓存中读取数据，加快加载速度。

* 分析 Mach-o Headers
* 分析依赖的动态库
* 查找需要 Rebase & Bind 之类的符号
* 把上述结果写入缓存

dyld2 是从 iOS 3.1 引入，一直持续到 iOS 12。dyld2 有个比较大的优化是 dyld shared cache[1]，什么是 shared cache 呢？

shared cache 就是把系统库 (UIKit 等) 合成一个大的文件，提高加载性能的缓存文件。
iOS 13 开始 Apple 对三方 App 启用了 dyld3，dyld3 的最重要的特性就是启动闭包，闭包里包含了启动所需要的缓存信息，从而提高启动速度。

## 最后

要更加努力呀！

Let's be CoderStar!