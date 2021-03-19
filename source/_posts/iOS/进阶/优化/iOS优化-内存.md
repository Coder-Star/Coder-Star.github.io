---
title: iOS优化-内存
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2021-03-18 09:44:17
---

### 检测循环引用

**MLeaksFinder**

基本原理是当 ViewController pop 或者 dimiss 后应该很快进行销毁，如果不销毁就是发生了循环引用，通过 hook 相关方法。在 hook 方法中使用一个弱引用引用 VC，然后延时一段时间再去使用该引用，如果此时引用为 nil，则没有循环引用，如果不为 nil，则有可能发生了循环引用。

**FBRetainCycleDetector**

当传入内存中的任意一个 OC 对象，FBRetainCycleDetector 会递归遍历该对象的所有强引用的对象，以检测以该对象为根结点的强引用树有没有循环引用。
