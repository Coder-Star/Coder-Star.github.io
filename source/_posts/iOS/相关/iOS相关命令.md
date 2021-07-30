---
title: iOS相关命令
category:
  - iOS
  - 命令
tags:
  - iOS
  - 命令
date: 2021-02-03 20:45:28
---

## lipo 相关

lipo 源于 mac 系统要制作兼容 powerpc 平台和 intel 平台的程序，lipo 是一个在 Mac OS X 中处理通用程序（Universal Binaries）的工具。

**lipo -info**
查看查看静态库支持的 CPU 架构
lipo -info frameworkName.framework/frameworkName
lipo -info frameworkName.a

**合并静态库**
lipo -create 静态库存放路径 1 静态库存放路径 2 ... -output 整合后存放的路径

lipo -create frameworkName-armv7.a frameworkName-armv7s.a frameworkName-i386.a -output frameworkName.a

lipo -create frameworkNameOne.framework/frameworkNameOne frameworkNameTwo.framework/frameworkNameTwo -output frameworkName.framework

**静态库拆分**

lipo 静态库源文件路径 -thin CPU 架构名称 -output 拆分后文件存放路径

lipo  libname.a  -thin  armv7  -output  libname-armv7.a

**擦除指定架构**

lipo XXX.a -remove arm64 -output XXX.a


## Xcode

- 关闭Xcode，打开终端输入`defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES`，然后项目Build的时候可以在Xcode顶部看到项目编译时间

- `defaults write com.apple.Xcode PBXNumberOfParallelBuildSubtasks 8`  提高XCode编译时使用的线程数


## xcodebuild



## xcurn