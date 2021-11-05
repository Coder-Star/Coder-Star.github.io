---
title: iOS中的时间
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

## mach_absolute_time

```
let beginTime = mach_absolute_time()

// do something...

var baseInfo = mach_timebase_info_data_t(numer: 0, denom: 0)
if mach_timebase_info(&baseInfo) == KERN_SUCCESS {
    let finiTime = mach_absolute_time()

    let nano = (finiTime - beginTime) * UInt64(baseInfo.numer) / UInt64(baseInfo.denom)
}
```

### mach_continuous_time()

## Date
## CFAbsoluteTimeGetCurrent
## CACurrentMediaTime

## 最后

要更加努力呀！

Let's be CoderStar!

- [时间间隔](https://www.freesion.com/article/868922153/)