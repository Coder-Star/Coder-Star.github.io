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

其实关于这块，网上的资料已经很多了，也已经有了很多成熟的优化方案，本文主要梳理了一下我所知的优化方案并结合我实际的使用给大家总结一下。

## App 启动过程

在优化之前，我们需要对 App 的完整启动过程有个了解，这样我们才能知道启动耗时分布的阶段、哪一个阶段可以被优化以及优化哪一个阶段 ROI 最高。

APP 的启动过程大部分情况都会被分成两部分，即`pre-main`以及`main`，其实还可以分的更细一点，分为三步：

- `pre-main`：main() 函数之前，即操作系统加载 App 可执行文件到内存，然后执行一系列的加载 & 链接等工作，最后执行至 App 的 `main()` 函数；
- `main`：`main()`函数之后，即从`main()`开始，到`appDelegate`的`didFinishLaunchingWithOptions`方法执行完毕；
- 首屏渲染：首批构建完成可浏览 / 可操作页面；

![启动流程](../../../../img/iOS/进阶/优化/启动优化/流程.jpeg)

### `pre-main`

在这个阶段，基本所有的工作都是由操作系统完成的，开发者能够插手的地方不多，所以如果想要优化这段时间，就必须先了解一下，操作系统在`main()`函数之前做了什么。`main()`函数之前操作系统所做的工作就是把可执行文件（Mach-O 格式）加载到内存空间，然后加载动态链接库 `dyld`，再执行一系列动态链接操作和初始化操作的过程（加载、绑定、及初始化方法）。

程序的加载是从`exec()`函数开始，exec() 是一个系统调用。操作系统首先为进程分配一段内存空间。

### dyld

dyld（the dynamic link editor）是苹果的动态链接器。是一个专门用来加载动态链接库的库，它是开源的。在 xnu 内核为程序启动做好准备后，执行由内核态切换到用户态，由 dyld 完成后面的加载工作，dyld 的主要是初始化运行环境，开启缓存策略，加载程序依赖的动态库 (其中也包含我们的可执行文件)，并对这些库进行链接（主要是 rebaseing 和 binding），最后调用每个依赖库的初始化方法，在这一步，runtime 被初始化。

dyld 会首先读取 mach-o 文件的 Header 和 load commands。接着就知道了这个可执行文件依赖的动态库。例如加载动态库 A 到内存，接着检查 A 所依赖的动态库，就这样的递归加载，直到所有的动态库加载完毕。通常一个 App 所依赖的动态库在 100-400 个左右，其中大多数都是系统的动态库，它们会被缓存到 dyld shared cache，这样读取的效率会很高。

* 设置运行环境。
  这一步主要是设置运行参数、环境变量等。比如获取 Xcode 设置的`Environment Variables`。
* 加载共享缓存。
  加载系统级别的动态库，比如`UIKit`等，位于`/System/Library/Caches/com.apple.dyld/dyld_shared_cache_armv64`
* 实例化主程序。
  这一步将主程序的 Mach-O 加载进内存，并实例化一个 ImageLoader。内核加载的主程序。
* 加载插入的动态库。
  这一步是加载环境变量`DYLD_INSERT_LIBRARIES`中配置的动态库。dyld 负责。

* 链接主程序。
  这一步调用 link() 函数将实例化后的主程序进行动态修正，让二进制变为可正常执行的状态。对应下面`rebase（偏移修正）/binding（符号绑定）`阶段
* 链接插入的动态库。
* 执行弱符号绑定

* 执行初始化方法。
  dyld 会优先初始化动态库，然后初始化主程序。

* 查找入口点并返回。
  执行 main 函数

### 扩展

dyld2 和 dyld3 的加载方式略有不同。dyld2 是纯粹的 in-process，也就是在程序进程内执行的，也就意味着只有当应用程序被启动的时候，dyld2 才能开始执行任务。dyld3 则是部分 out-of-process，部分 in-process。

dyld2 的过程是：加载 dyld 到 App 进程，加载动态库（包括所依赖的所有动态库），Rebase，Bind，初始化 Objective C Runtime 和其它的初始化代码。

dyld3 的 out-of-process 会做如下事情：分析 Mach-O Headers，分析依赖的动态库，查找需要 Rebase & Bind 之类的符号，把上述结果写入缓存。这样，在应用启动的时候，就可以直接从缓存中读取数据，加快加载速度。

### 过程

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
     - 执行声明为 __attribute__((constructor)) 的 C/C++ 函数
     - 创建 C++ 静态全局变量

> 为什么需要 Rebase 和 Bind？

> ASLR（Address Space Layout Randomization），地址空间布局随机化。在 ASLR 技术出现之前，程序都是在固定的地址加载的，这样 hacker 可以知道程序里面某个函数的具体地址，植入某些恶意代码，修改函数的地址等，带来了很多的危险性。ASLR 就是为了解决这个的，程序每次启动后地址都会随机变化，这样程序里所有的代码地址都需要需要重新对进行计算修复才能正常访问。

> Rebase：Rebasing 过程就是从__LINKEDIT 取出函数指针，根据偏移量修改函数指针，存入__DATA 中，Rebase 解决了**内部的符号引用**问题。

> Binding：当引用动态库其他的函数或者变量时，当前 mach-o 文件会指向其他 dylib。这时候就需要 Binding 操作，dyld 会根据符号表去找到相应函数和变量地址，Binding 解决了**修正外部指针指向**的问题。

### 调用 `main` 函数执行。（main -> applicationDidFinishLaunching）

  该阶段是指 main 函数执行之后到 AppDelegate 类中的 applicationDidFinishLaunching:withOptions: 方法执行结束前这段时间。
  这是 APP 启动优化的重点。

  主要是一些启动项。

### applicationDidFinishLaunching -> 首页渲染完成

   - 尽量使用纯代码编写，减少 xib/storyboard 的使用，首页布局不要过于复杂
   - 在 viewDidLoad 以及 viewWillAppear 方法中少做逻辑，或者采用异步的方式去做

- [dyld详解](https://www.dllhook.com/post/238.html)

## 指标及量化手段

苹果建议的 `pre-main` 启动时间不要超过 `400ms`，也就是从点击图标到启动图消失这段时间不能超过 `400ms`，并且启动时间超过 `20s` 将会被系统直接杀死。

### Environment Variables

进入 `Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables`，增加`DYLD_PRINT_STATISTICS`，设置值为`1`，如果获取更详细的信息，可以使用 `DYLD_PRINT_STATISTICS_DETAILS`。

加入`DYLD_PRINT_STATISTICS`后，显示信息如下：

![DYLD_PRINT_STATISTICS](../../../../img/iOS/进阶/优化/启动优化/DYLD_PRINT_STATISTICS.png)

加入`DYLD_PRINT_STATISTICS_DETAILS`后

![DYLD_PRINT_STATISTICS_DETAILS](../../../../img/iOS/进阶/优化/启动优化/DYLD_PRINT_STATISTICS_DETAILS.png)

在使用这种方式时，需要注意两个地方
- 这种方式在我测试过程中发现并不支持真机，只支持模拟器。（当然可能是我打开姿势不对，有了解的小伙伴可以私信告知我）
- 在 Debug 环境下拿到的数据会有`debugger pause time`的影响，为排除该因素影响，我们可以将`scheme`中的`debug executable`进行关闭。

### App Launch

### 埋点

#### 进程启动时间

这种方式据小伙伴说会有一定的偏差。

```objective-c
#import <sys/sysctl.h>
#import <mach/mach.h>

+ (BOOL)processInfoForPID:(int)pid procInfo:(struct kinfo_proc*)procInfo
{
    int cmd[4] = {CTL_KERN, KERN_PROC, KERN_PROC_PID, pid};
    size_t size = sizeof(*procInfo);
    return sysctl(cmd, sizeof(cmd)/sizeof(*cmd), procInfo, &size, NULL, 0) == 0;
}

+ (NSTimeInterval)processStartTime
{
    struct kinfo_proc kProcInfo;
    if ([self processInfoForPID:[[NSProcessInfo processInfo] processIdentifier] procInfo:&kProcInfo]) {
        return kProcInfo.kp_proc.p_un.__p_starttime.tv_sec * 1000.0 + kProcInfo.kp_proc.p_un.__p_starttime.tv_usec / 1000.0;
    } else {
        NSAssert(NO, @"无法取得进程的信息");
        return 0;
    }
}
```

```swift
extension ProcessInfo {
    public var uptime: TimeInterval {
        return Date().timeIntervalSince(startTime)
    }

    public var startTime: Date {
        return processStartTime(for: processIdentifier)
    }

    public func processStartTime(for pid: Int32) -> Date {
        var mib = [CTL_KERN, KERN_PROC, KERN_PROC_PID, pid]
        var proc = kinfo_proc()
        var size = MemoryLayout.size(ofValue: proc)
        mib.withUnsafeMutableBufferPointer { p in
            _ = sysctl(p.baseAddress, 4, &proc, &size, nil, 0)
        }

        let time = proc.kp_proc.p_starttime
        let seconds = Double(time.tv_sec) + Double(time.tv_usec) / Double(NSEC_PER_SEC)

        return Date(timeIntervalSince1970: seconds)
    }
}
```

#### pre-main 结束时间

推荐使用__attribute__((constructor)) 构建器函数的被调用时间点作为 pre-main() 阶段结束时间点：__t2 能最大程度实现解耦。为什么不用最后一个 load 方法执行时间作为 pre-main() 阶段的结束时间点？因为在超大型工程中我们没办法确定哪个名称 load 方法是最后一个被执行 load 方法。。。

```objective-c
void static __attribute__((constructor)) before_main() {
    // 获取时间
}
```

## pre-main 阶段优化

- 动态库转静态库
- 减少动态库（使用静态库）的个数如果太多就使用合并（最多支持 6 个非系统动态库合成一个）的方式控制；这样可以节约 dylib loading 的时间。比如可以将 XXTableView, XXHUD, XXLabel 这些分散的库合并成一个 XXUIKit 来提高加载速度。
  > linkage 的符号 internal 更多了，external 更少了。dyld bind 的时间必然能变少。比如两个动态库都依赖了 objc_msgsend，要 bind 两次；但是一个库的话只需要一次，我是这么理解的。
- 动态库懒加载
- 尽量不使用 C++ 虚函数；这样可以节约 rebase/binding 的时间
- 清理项目中未用到的类、类别、方法等，这样可以节约 Objc setup 的时间。这个过程会影响很多方面，代码减少会降低 fixup 的次数，如果减少的是分类，也会降低`Objc setup`的时间
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
- [iOS 应用的启动流程和优化详解](https://juejin.cn/post/6951591401528229895)
- [抖音品质建设 - iOS启动优化《原理篇》](https://mp.weixin.qq.com/s/3-Sbqe9gxdV6eI1f435BDg)
- [Optimizing App Launch](https://developer.apple.com/videos/play/wwdc2019/423/)
- [美团外卖iOS App冷启动治理](https://tech.meituan.com/2018/12/06/waimai-ios-optimizing-startup.html)
