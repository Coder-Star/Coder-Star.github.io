---
title: iOS持久化方式-UserDefaults
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 持久化
date: 2021-02-24 22:43:50
---

**关于 UserDefaults 中的 suitename**

在咱们平时使用 UserDefaults 时一般使用`UserDefaults.standard`这个方式这存储信息，通过这种方式实际上操作的是 APP 沙箱中 Library/Preferences 目录下的以 bundle id 命名的 plist 文件；如果一个 APP 使用了一些 SDK，这些 SDK 会或多或沙的使用该方式存储，这样就会带来一系列问题；

- 各个 SDK 需要保证设置数据 KEY 的唯一性，以防止存取冲突；
- plist 文件越来越大造成的读写效率问题；
- 无法便捷的清除由某一个 SDK 创建的 UserDefaults 数据；

针对上述问题，我们可以使用 UserDefaults 提供的另外一个方法，

```Swift
@available(iOS 7.0, *)
public init?(suiteName suitename: String?)
```

根据传入的 suitname 有四种情况

- 传入 nil；跟使用`UserDefaults.standard`效果相同
- 传入 bundle id；无效，返回 nil
- 传入 App Groups 配置中 Group ID；会操作 APP 的共享目录中创建的 plist 文件，方便跨 APP 或宿主 APP 与扩展应用之间共享数据；
- 传入其他值；操作的是沙箱的 Documents/Library/Preferences 目录下以 suitename 命名的 plist 文件

**UserDefaults 底层也是使用 plist 文件，那和普通的 plist 文件读取有什$$么区别呢？**

iOS 系统会自动帮你做 plist 文件的读入以及 Cache，根据情况会把你在内存中所做的操作同步到 plist 文件（UserDefaults 同步到内存是同步的，同步到 Database 是异步的， iOS 8 开始，会有一个常驻进程 cfprefsd 来负责同步），所以UserDefaults的`synchronize`函数废弃也是有道理的，因为其保证不了调用之后会将值立即存储到plist文件中。本质上，我们是可以对 UserDefaults 的最终产物 plist 文件进行操作的，但这是有风险的，最好不要这么操作。
