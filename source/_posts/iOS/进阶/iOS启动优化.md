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

APP 启动过程

1. 解析 Info.plist
   - 加载相关信息，例如如闪屏
   - 沙箱建立、权限检查
2. Mach-O 加载
   - 如果是胖二进制文件，寻找合适当前 CPU 类别的部分
   - 加载所有依赖的 Mach-O 文件（递归调用 Mach-O 加载的方法）
   - 定位内部、外部指针引用，例如字符串、函数等
   - 加载类扩展（Category）中的方法
   - C++静态对象加载
   - 调用 ObjC 的 +load 函数
   - 执行声明为 attribute((constructor))的 C 函数
3. 程序执行
   * 调用main()
   * 调用UIApplicationMain()
   * 调用applicationWillFinishLaunching

**关键步骤**

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
