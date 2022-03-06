---
title: SPM 相关
category:
  - Swift
  - SPM
tags:
  - SPM
date: 2022-01-16 17:50:55
---

## 前言

Hi Coder，我是 CoderStar！

## SPM 太卡顿

首先你得有一个翻墙工具，我用的是`V2RayX`，端口为`8001`。

![V2RayX端口](../../../../img/iOS/Swift/SPM/V2RayX端口.png)

我们使用下列命令将终端代理指向翻墙工具。

`export https_proxy=http://127.0.0.1:8001 http_proxy=http://127.0.0.1:8001  all_proxy=socks5://127.0.0.1:8001`

然后使用下列命令使 SPM 下载时走终端的 git，这时就会开始 reslove。
`xcodebuild -resolvePackageDependencies -scmProvider system`
> 执行过程中可能会出现找不到对应版本的情况，这时候可以先用 Xcode 打开项目，再执行这些命令。

## 创建

我们可以先创建一个文件夹，然后进入文件夹后，执行`swift package init`命令，这时，

## 格式

我们需要使用一个`Package.swift`这个文件去描述一个 Swift Package。

```swift



```

## 其他

目前一个 package 可以支持 OC、Swift

## 最后

要更加努力呀！

Let's be CoderStar!
