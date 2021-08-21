---
title: Instrument 相关
date: 2020-11-03 17:47:28
categories:
  - iOS
  - 相关
tags: [Instrument, iOS]
---

- Leaks（泄漏）：一般的查看内存使用情况，检查泄漏的内存，并提供了所有活动的分配和泄漏模块的类对象分配统计信息以及内存地址历史记录；
- Time Profiler（时间探查）：执行对系统的 CPU 上运行的进程低负载时间为基础采样。
- Allocations（内存分配）：跟踪过程的匿名虚拟内存和堆的对象提供类名和可选保留 / 释放历史；
- Activity Monitor（活动监视器）：显示器处理的 CPU、内存和网络使用情况统计；
- Blank（空模板）：创建一个空的模板，可以从 Library 库中添加其他模板；
- Automation（自动化）：这个模板执行它模拟用户界面交互为 IOS 机应用从 instrument 启动的脚本；
- Core Data：监测读取、缓存未命中、保存等操作，能直观显示是否保存次数远超实际需要。
- Cocoa Layout：观察约束变化，找出布局代码的问题所在。
- Network：跟踪 TCP / IP 和 UDP / IP 连接。
- Automations：创建和编辑测试脚本来自动化 iOS 应用的用户界面测试。

常用的为

- Leaks 检查内存, 看是否有内存泄露
- Time Profiler 分析代码的执行时间，找出导致程序变慢的原因
- Allocations 用来检查内存分配, 写算法的那批人也用这个来检查
- Zombies 检测僵尸对象
