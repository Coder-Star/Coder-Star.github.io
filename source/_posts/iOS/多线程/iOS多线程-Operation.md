---
title: Operation
category:
  - iOS
  - 多线程
tags:
  - iOS
  - 多线程
  - Operation
date: 2021-01-26 23:29:20
---

Operation 是苹果基于 GCD 封装的，面向对象使用；可控性比 GCD 要强，


相关的类有 Operation 和 OperationQueue。其中 Operation 是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的

子类：BlockOperation。创建 Operation 子类的对象，把对象添加到 OperationQueue 队列里执行。



相对GCD的优点：

- 可以对操作之前进行依赖；
- 设置最大任务数；