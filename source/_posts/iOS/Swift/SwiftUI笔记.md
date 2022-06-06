---
title: SwiftUI笔记
category:
  - Swift
tags:
  - Swift
date: 2022-06-03 20:16:55
---

## 前言

Hi Coder，我是 CoderStar！

## 笔记

pop页面
```swift
@Environment(\.presentationMode) var presentationMode

self.presentationMode.wrappedValue.dismiss()
```

NavigationLink 必须在 NavigationView 下。


List item取消高亮样式。
```swift
给ForEach添加.listRowBackground(Color.clear)
```

## 最后

要更加努力呀！

Let's be CoderStar!