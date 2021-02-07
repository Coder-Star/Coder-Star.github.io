---
title: iOS启动优化
category:
  - iOS
  - 启动优化
tags:
  - iOS
  - 启动优化
date: 2021-02-05 22:03:03
---

APP 启动分为三个阶段

- main 函数执行前
  main()函数执行之前要做的事情，会包括
  - 加载可执行文件（优化方式：删减无用静态常量，删除无用代码，尽量不使用 C++虚函数）
  - 加载动态链接库（可以进行优化，最多支持 6 个非系统动态库合成一个）
  - 初始化 Objective-C Runtime
  - 执行其他初始化代码（load 方法），优化方式是将 load 方法里面执行的逻辑放入到首屏渲染后执行，放入+initialize 中

- main 函数执行后  
  main()函数执行之后到 AppDelegate 类中的 applicationDidFinishLaunching:withOptions:方法执行结束前这段时间。这是 APP 启动优化的重点
  - 尽量使用纯代码编写，减少 xib 的使用
  - 启动阶段的网络请求，是否都放到异步请求
  - 一些耗时的操作是否可以放到后面去执行，或异步执行等

- 首屏渲染完成后
