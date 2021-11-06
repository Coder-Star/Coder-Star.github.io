---
title: iOS 中的时间
category:
  - iOS
  - 时间
tags:
  - iOS
  - 时间
date: 2021-11-05 07:45:27
---

## 前言

Hi Coder，我是 CoderStar！

Date 或 CFAbsoluteTimeGetCurrent() 返回网络时间同步的时钟时间。

mach_absolute_time() 和 CACurrentMediaTime() 是系统时间，是基于内建时钟的，能够更精确更原子化地测量，

并且不会因为外部时间变化而变化（例如时区变化、夏时制、秒突变等）, 但它和系统的 uptime 有关, 系统重启后 CACurrentMediaTime() 会被重置。

NSDate 属于Foundation框架

CFAbsoluteTimeGetCurrent（）属于CoreFoundation框架

CACurrentMediaTime（）属于QuartzCore框架

## Date

## CFAbsoluteTimeGetCurrent

相当于` Date().timeIntervalSinceReferenceDate`

### gettimeofday

## mach_absolute_time

```swift
let beginTime = mach_absolute_time()

// do something...

var baseInfo = mach_timebase_info_data_t(numer: 0, denom: 0)
if mach_timebase_info(&baseInfo) == KERN_SUCCESS {
    let finiTime = mach_absolute_time()

    let nano = (finiTime - beginTime) * UInt64(baseInfo.numer) / UInt64(baseInfo.denom)
}
```

返回一个基于系统启动后的时钟嘀嗒数，是一个 CPU/ 总线依赖函数。在 macOS 上可以确保它的行为，并且它包含系统时钟所拥有的全部时间区域，精度达到纳秒级。

**时钟嘀嗒数在每次手机重启后，都会重新开始计数，而且 iPhone 锁屏进入休眠之后，tick 也会暂停计数。**

一般应用场景是 作为耗时的测量工具。

不会受系统时间影响，只受设备重启和休眠行为影响。

我们知道 mach_absolute_time() 在系统锁屏的时候是处于暂停状态，如果我们想统计包含锁屏时间，那么该如何处理呢？

答案是 iOS 10 以后，官方为我们提供的`mach_continuous_time()`函数，供我们使用。

## CACurrentMediaTime

其实底层就是`mach_absolute_time()`，只不过将结果转换为了秒。

## 最后

要更加努力呀！

Let's be CoderStar!

- [时间间隔](https://www.freesion.com/article/868922153/)
