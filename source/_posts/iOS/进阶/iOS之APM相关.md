---
title: iOS 之 APM 相关
category:
  - iOS
tags:
  - iOS
  - APM
date: 2022-02-03 15:41:14
---

## 前言

Hi Coder，我是 CoderStar！

## 启动时间

[iOS优化-启动优化](../../进阶/优化/iOS优化-启动优化)

## Crash

[iOS 符号化浅析](../../进阶/iOS%20符号化浅析)

- [KSCrash](https://github.com/kstenerud/KSCrash) （建议阅读源码）
- [plcrashreporter](https://github.com/microsoft/plcrashreporter)
- [CrashKit](https://github.com/kaler/CrashKit)
- [breakpad](https://github.com/google/breakpad)

## 卡顿

[iOS 卡顿监测方案总结](https://mp.weixin.qq.com/s/Csg7_lN8QjCdngY-EB_H9g)
[ iOS 性能监控 SDK —— Wedjat（华狄特）开发过程的调研和整理](https://github.com/aozhimin/iOS-Monitor-Platform/blob/master/iOS-Monitor-Platform_Basic.md)

### CADisplayLink

CADisplayLink 仍然是基于 Runloop 来实现的，而 RunLoop 的运行取决于其所在的 mode 以及 CPU 的繁忙程度，当 CPU 忙于计算显示内容或者 GPU 工作太繁重时，就会导致显示出来的 FPS 与 Instrument 的不一致。所以说基于 CADisplayLink 实现的 FPS 无法完全检测出当前 Core Animation 的性能情况，它只能检测出当前 RunLoop 的帧率。

### 子线程 Ping：这种方案不准

### Runloop

### CPU 超过 80%

获取当前堆栈：[RCBacktrace](https://github.com/woshiccm/RCBacktrace)

## 其他

iOS主线程UI调用监测工具：[MDMMainThreadChecker](https://github.com/mademao/MDMMainThreadChecker)

## 最后

要更加努力呀！

Let's be CoderStar!
