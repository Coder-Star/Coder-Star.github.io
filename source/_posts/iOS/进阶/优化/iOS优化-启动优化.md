---
title: iOS 优化 - 启动优化
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2021-02-05 22:03:03
---

## 前言

该篇文章是 iOS 优化系列的第二篇，主要介绍一下启动优化，即如何减少应用的启动时间。

苹果建议的pre-main启动时间不要超过 400ms，也就是从点击图标到启动图消失这段时间不能超过 400ms，并且启动时间超过 20s 将会被系统直接杀死。

## APP 启动过程

我们需要先了解 APP 的启动过程，才能对过程中的一些步骤进行优化，减少时间。

### 解析 `Info.plist`
   - 加载相关信息，例如如闪屏；
   - 沙箱建立、权限检查；
### `Mach-O` 加载（`main` 函数执行前，也被成为 **`pre-main`** 阶段）
   - dylib loading
     - 加载所有依赖的 Mach-O 文件（递归调用 Mach-O 加载的方法）
     - 加载动态链接库加载器 dyld(dynamic loader)
     - 加载动态链接库
   - rebase（偏移修正）/binding（符号绑定）
     - rebase(偏移修正)：修复内部指针。ASLR+ 偏移值 = 运行时确定的内存地址, 造成这种现象的原因是因为系统运用了虚拟内存。
     - binding(符号绑定)： 绑定就是给符号赋值的过程。例如 NSLog 方法，在编译时期生成的 mach-o 文件中，会创建一个符号！NSLog（目前指向一个随机的地址），然后在运行时（从磁盘加载到内存中，是一个镜像文件），会将真正的地址给符号（即在内存中将地址与符号进行绑定，是 dyld 做的，也称为动态库符号绑定）。
   - Objc setup
     - 初始化 Objective-C Runtime（包括 ObjC 相关 Class 的注册、category 注册、selector 唯一性检查等），category 注册会将 category 定义的方法插入到主类中。
   - initializer
     - 调用 ObjC 的 +load 函数
     - 执行声明为 attribute((constructor)) 的 C/C++ 函数
     - 创建 C++ 静态全局变量
### 调用 `main` 函数执行。（main -> applicationDidFinishLaunching）
  该阶段是指 main 函数执行之后到 AppDelegate 类中的 applicationDidFinishLaunching:withOptions: 方法执行结束前这段时间。
  这是 APP 启动优化的重点

### applicationDidFinishLaunching -> 首页渲染完成
   - 尽量使用纯代码编写，减少 xib/storyboard 的使用，首页布局不要过于复杂
   - 在 viewDidLoad 以及 viewWillAppear 方法中少做逻辑，或者采用异步的方式去做


## 量化手段

一般而言

## pre-main 阶段优化

- 减少动态库（使用静态库）的个数如果太多就使用合并（最多支持 6 个非系统动态库合成一个）的方式控制；这样可以节约 dylib loading 的时间。比如可以将 XXTableView, XXHUD, XXLabel 这些分散的库合并成一个 XXUIKit 来提高加载速度。
  > linkage的符号internal更多了，external更少了。dyld bind的时间必然能变少。比如两个动态库都依赖了objc_msgsend，要bind两次；但是一个库的话只需要一次，我是这么理解的。
- 尽量不使用 C++ 虚函数；这样可以节约 rebase/binding 的时间
- 清理项目中未用到的类、类别、方法等，这样可以节约 Objc setup 的时间
- 将 load 方法里面执行的逻辑延迟执行，如放入到首屏渲染后或者 +initialize 执行；控制 C++ 全局变量的数量；这样可以节约 initializer 的时间
- 二进制重排（主要是节省加载 Mach-O 文件的时间）
- 多使用 swift structs，利用 swift 静态分发的特性。

## main 阶段优化

- 启动阶段的网络请求，是否都放到异步请求
- 一些耗时的操作是否可以放到后面去执行，或异步执行等

## 二进制重排

核心步骤

- 利用 clang 插桩获得启动时期需要加载的所有 函数 / 方法 , block , swift 方法以及 c++ 构造方法的符号，已经有大神写好的现成的包 `https://github.com/yulingtianxia/AppOrderFiles`
- 通过 order file 机制实现二进制重排

编译器在生成二进制代码的时候，会先编译 OC 的代码，然后在编译 Swift 的代码，在此顺序前提下，会按照编译文件顺序、方法在文件中的顺序生成。

核心原理是启动时候减少 Page Fault（缺页中断）发生的次数。

获取启动加载所有函数的符号

- hook objc_MsgSend ( 只能拿到 oc 以及 swift @objc dynamic 后的方法 , 并且由于可变参数个数 , 需要用汇编来获取参数 .)
- 静态扫描 macho 特定段和节里面所存储的符号以及函数数据 . (静态扫描 , 主要用来获取 load 方法 , c++ 构造 (有关 c++ 构造 , 参考 从头梳理 dyld 加载流程 这篇文章有详细讲述和演示 ) .
- clang 插桩 ( 完美版本 , 完全拿到 swift , oc , c , block 全部函数 ) .

clang 插桩

1. 在 Build Settings 中添加编译选项 Other C Flags 增加`-fsanitize-coverage=func,trace-pc-guard`;
2. 如果是 OC Swift 混编，
   则在 Other Swift Flags 增加
   - `-sanitize-coverage=func`
   - `-sanitize=undefined`

获取启动过程中的执行方法：[AppOrderFiles](https://github.com/yulingtianxia/AppOrderFiles)


- [58 同城 App 性能治理实践-iOS 启动时间优化](https://mp.weixin.qq.com/s/wkK2UBvuUZW3Pf0Yd_3XTA)
- [iOS 优化篇 - 启动优化之Clang插桩实现二进制重排](https://juejin.cn/post/6844904130406793224)
- [脉脉iOS如何启动秒开](https://zhuanlan.zhihu.com/p/396550853)