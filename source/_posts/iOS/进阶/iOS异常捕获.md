---
title: iOS异常捕获
category:
  - iOS
  - 进阶
tags:
  - iOS
  - 异常捕获
date: 2021-02-06 18:49:28
---

iOS 中异常捕获的方式主要包含三种

- Mach 异常：EXC_CRASH
- UNIX 信号：SIGABRT
- NSException 异常：应用层，通过 NSUncaughtExceptionHandler 捕获

## Mach 异常

```

```

## UNIX 信号

```swift
extension CrashHandler {

    private static func openSignalHandler() {
        // 大部分异常就是SIGTRAP，OC中的NSException异常对应的也是这个信号。
        signal(SIGTRAP, CrashHandler.signalHandler)
        signal(SIGABRT, CrashHandler.signalHandler)
        signal(SIGSEGV, CrashHandler.signalHandler)
        signal(SIGBUS, CrashHandler.signalHandler)
        signal(SIGILL, CrashHandler.signalHandler)
    }

    private static func closeSignalHandler() {
        signal(SIGINT, SIG_DFL)
        signal(SIGSEGV, SIG_DFL)
        signal(SIGTRAP, SIG_DFL)
        signal(SIGABRT, SIG_DFL)
        signal(SIGILL, SIG_DFL)
    }

    private static let signalHandler: @convention(c) (Int32) -> Void = { signal in
    }
}
```

## NSException 异常

这种方式捕获异常只在 OC 环境下使用，一般情况下捕获 NSException 异常后同时也会捕获一个对应的 signal 异常。

```oc
extension CrashHandler {
    private static func openExceptionHandler() {
        NSSetUncaughtExceptionHandler(CrashHandler.exceptionHandler)
    }

    private static func closeExceptionHandler() {
        NSSetUncaughtExceptionHandler(nil)
    }

    private static let exceptionHandler: @convention(c) (NSException) -> Void = { exception in

    }

}




```
