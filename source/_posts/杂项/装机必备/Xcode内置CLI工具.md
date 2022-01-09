---
title: Xcode 内置 CLI 工具
category:
  - 杂项
tags:
  - Xcode
date: 2022-01-06 16:57:27
---

## 前言

Hi Coder，我是 CoderStar！

分为两类，一种是 Mac 下自带的命令行工具，另一种是 Xcode 附带的。

其中 Mac 自带的工具所在路径为`/usr/bin/`；
Xcode 附带的在`/Applications/Xcode.app/Contents/Developer/usr/bin`；

## Mac

### otool

## Xcode

### xcode-select

### actool

### atos

### xcodebuild

可以通过`man xcodebuild`查看手册。

我们可以使用其进行清理，分析，构建，测试，存档。

- xctool：xctool 是 facebook 推出的用于替换 xcodebuild 的更易于测试 ios 和 mac 应用程序的命令行工具，特别适用于 ios app 的持续集成。
- xcbuild：xcbuild 是一个兼容 Xcode 的编译工具，它能使编译更快快速，更友好的编译过程日志，可以运行在多个平台（主要指 OS X 和 Linux）

### xcurn

`xcurn`是用来调用其他的工具，比如`xcodebuild`、`swift-demangle`，当然有些工具是可以直接调用的，比如`xcodebuild`，但是有些工具必须借助`xcurn`，比如`swift-demangle`。

将swift函数名`demangle`，
```shell
xcrun swift-demangle  _TtC7iOSTest27PickImageDemoViewController
_TtC7iOSTest27PickImageDemoViewController ---> iOSTest.PickImageDemoViewController

xcrun swift-demangle --compact _TtC7iOSTest27PickImageDemoViewController
iOSTest.PickImageDemoViewController
```

### symbolicatecrash

### lipo

### libtool

### file

### dwarfdump

### class-dump

### dumpdecrypted

## 最后

要更加努力呀！

Let's be CoderStar!

- [iOS开发中常用命令工具（xcode-select、lipo、xcrun等](https://www.jianshu.com/p/a4731527ca73)
- [Xcode 相关终端工具使用](https://www.hanleylee.com/usage-of-xcode-terminal-tools.html)
- [Building from the Command Line with Xcode FAQ](https://developer.apple.com/library/archive/technotes/tn2339/_index.html)