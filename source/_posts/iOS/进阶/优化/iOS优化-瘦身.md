---
title: iOS 优化 - 瘦身
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2021-01-19 08:50:18
---

## 前言

iOS 优化将是一个系列文章，其中会包括包体积优化（瘦身）、启动时间优化、UI 优化等等。那么这个系列的开篇就从瘦身开始吧。

APP 的大小是分为 APP 下载大小和安装大小两个概念的。
- 下载大小是指 App 压缩包（也就是 .ipa 文件）所占的空间，用户在下载 App 时，下载的是压缩包，这样做可以节省流量；
- 当压缩包下载完成后，就会自动解压，解压过程也就是通常所说的安装过程；安装大小就是指压缩包解压后所占用的空间。

其实用户在商店看到的大小是安装大小。如果想看全可以在 App Store Connect 后台查看。

虽然苹果历年都会调整 App 下载大小，由之前的 100M 到后来的 150M 再到现在的 200M。如今，App 下载大小超出 200 MB 时 ，会出现两种情况：
- iOS 13 以下的用户，无法通过蜂窝数据下载 App；
- iOS 13 及以上的用户，需要手动设置才可以使用蜂窝网络下载 App。

可执行文件大小限制：

* iOS 7 之前，二进制文件中所有的 __TEXT 段总和不得超过 80 MB；
* iOS 7.X 至 iOS 8.X ，二进制文件中，每个特定架构中的 __TEXT 段不得超过 60 MB；
* iOS 9.0 之后，二进制文件中所有的 __TEXT 段总和不得超过 500 MB。

> 顺便给大家说下苹果将下载大小限制由 100M 调整到 150M 的原因是什么，主要原因就是 Uber 当年用 Swift 重构开发 APP，随着业务的增长，后期发现实在无法再将 APP 尺寸降到 100M 以下，只能联系苹果让其将下载大小提升到 150M，同时苹果的 Swift 团队还帮助添加了一些编译器选项 (-Osize)。

该文主要研究的是如何降低 APP 的下载大小的。在介绍我们作为开发者的优化方向之前，我们先看一下苹果自身对于 APP 下载大小的优化有哪些吧。

## App Thinning（苹果自身优化）

App Thinning 是指 iOS9 以后引入的一项优化，Apple 会尽可能，自动降低分发到具体用户时，所需要下载的 App 大小。其主要包含以下三项功能。

### Slicing（应用分割）

当向 App Store Connect 上传 .ipa 后，App Store Connect 构建过程中，会自动分割该 App，会专门针对不同的设备来选择只适用于当前设备的内容（主要是架构和资源）以供设备下载。其差异性主要是体现在架构（32 位还是 64 位）和资源（@1x、@2x 还是 @3x）。

其中架构方面开发者不需要去控制，但是对于资源来说要求图片在 **Asset Catalog** 管理，如果放在 Bundle，则不会被优化。

### Bitcode（中间码）

Bitcode 是一个编译好的程序的中间表示形式（IR）。上传到 App Store Connect 中的包含 Bitcode 的 App 将会在 App store 中进行链接和编译。苹果会对包含 Bitcode 的二进制 app 进行二次优化，而不需要提交一个新的 app 版本到 app store 中。属于 Apple 内部的优化，但需要注意；

- 全部都要支持。我们所依赖的静态库、动态库、Cocoapods 管理的第三方库，都需要开启 Bitcode。否则打包会编译失败。具体错误会在 Xcode 中指出。
- crash 定位。开启 Bitcode 后最终生成的可执行文件是 Apple 自动生成的，同时会产生新的符号表文件，所以我们无法使用自己包生成的 DYSM 符号化文件来进行符号化，而是使用使用 Apple 生成的 DYSM 符号化文件。
- Flutter 不支持 Bitcode，如果项目是包含 Flutter 框架的，就无法使用这种方式。
- BitCode 在 iOS 开发中是可选的，在 watchOS 开发中是必须要选择的， Mac OS 是不支持 BitCode 的。

开启方式：`Build Settings -> Enable Bitcode -> 设置为 YES`。

如果想对Bitcode了解更深入一些，可以看下我之前的一篇博文--[iOS编译简析](../iOS编译简析)。

### On-Demand Resources（随需应变资源）

on-Demand Resource 即一部分图片可以被放置在苹果的服务器上，不随着 App 的下载而下载，直到用户真正进入到某个页面时才下载这些资源文件。

应用场景：相机应用的贴纸或者滤镜、关卡游戏等。

开启方式：`Build Settings -> Enable On Demand Resources -> 设置为 YES`

设置资源的 Tag 类型，种类包括：
* Initial install tags：资源和 App 同时下载。在 App Store 中，App 的大小计算已经包含了这部分资源。当没有 NSBundleResourceRequest 对象访问它们时，它们将会从设备上清除。
* Prefetch tag order： 在 App 安装后开始下载，按照预加载列表中的顺序依次下载。
* Dowloaded only on demand： 只有在 App 中发出请求时才会下载。

## 瘦身方向

将 ipa 安装包后缀名改为 zip，将其解压，进入。app 文件后，就可以很直观的看到安装包的组成部分，其中主要的组成部分便是资源文件与 可执行文件 Mach-O 文件两部分。其中资源文件又包括 Assets.car 文件、plist 文件、json 文件等等。这两个部分便是我们的主要瘦身方向。

## 资源瘦身

### 去除无用的图片资源

业务的迭代开发，出现无用的图片资源是比较正常的，我们可以借助工具找出哪些图片资源没有被使用过。推荐下面两款工具。

- [LSUnusedResources](https://github.com/tinymind/LSUnusedResources)：可视化客户端工具。
- [FengNiao](https://github.com/onevcat/FengNiao.git)：命令行工具，可嵌入到Run Script中使用。

因为这类工具的原理都是在相关文件（.m、.swift 等等）中检测是否有图片名称的字符，所以存在以下问题。
问题点：
- 如果代码中使用的图标名称是拼接而成的，就会误以为相关图片是废弃图片；
- 如果 Assets.xcassets 文件中直接修改了图片的名字，也会认为相关图片可能是废弃图片；

### @1x 图片去除

 @1x 的图片已经没有必要继续放入 Assets 中去了，可以删除所有 @1x 的图片资源。

 > @1x 图是 3Gs 用的，4 都开始使用 @2x 图了，@6p 开始使用 3x 图

### 资源压缩

以图片资源举例，我们可以使用工具对其进行压缩，推荐两款工具如下：
- 网页工具：[TinyPng](https://tinypng.com/)
- GUI 客户端工具：ImageOptim（https://github.com/ImageOptim/ImageOptim）

### 图片资源使用 Webp 格式

谷歌开源的格式，Webp压缩率比较高，同时支持有损和无损两种压缩模式，可以带来更小的图片体积，而且肉眼看不出差异。根据Google的测试，无损压缩后的WebP比PNG文件少了26％的体积，有损压缩后的WebP图片相比于等效质量指标的JPEG图片减少了25％~34%的体积。

但是 Webp 格式 在 CPU 消耗和解码时间上会比 PNG 格式 高两倍。所以，我们需要根据项目的实际情况在性能和体积上做取舍。

推荐二种转 Webp 格式的方法

- [iSpart](http://isparta.github.io/)：腾讯出品，GUI工具；
- [webp工具](https://developers.google.com/speed/webp/docs/using): 在Mac下，可以使用homebrew安装webp工具--`brew install webp`；

SDWebImage目前支持加载Webp格式的图片--`pod 'SDWebImage/WebP'`。

### 将部分资源移动到远程

除了上文提到的使用On-Demand Resources方式将部分资源放在苹果服务器之外，我们也可以将一些本地资源转移到服务器上去，如一些banner广告图、主题资源等等。这样不仅降低了安装包大小，也将这些资源动态化了。

### 图标优化

- 使用tint color精简图标
- 使用图标字体替换部分图标

## 代码瘦身

### 清除无用代码

### 

[LinkMap](https://github.com/huanxsd/LinkMap)




## 编译器优化

Xcode 支持编译器层面的一些优化优化选项，可以让我们介于更快的编译速度和更小的二进制大小并且更快的执行速度之间自由选择想要进行的优化粒度。

去除无用架构，如可以去除 armv6、armv7、armv7s。

- 模拟器 32 位处理器测试需要 i386 架构
- 模拟器 64 位处理器测试需要 x86_64 架构
- 真机 32 位处理器需要 armv7, 或者 armv7s 架构
- 真机 64 位处理器需要 arm64 架构

armv6

- iPhone
- iPhone2
- iPhone3G
- 第一代和第二代 iPod Touch

armv7

- iPhone4
- iPhone4S
- iPad1-iPad3，3、4 代 iPod Touch
- iPad mini

armv7s

- iPhone5
- iPhone5C
- iPad4

arm64

- iPhone 5S
- ...

其他全部是 arm64 架构，理论上只保留 arm64 架构其实就够用了。可以在`Build Settin`-`Excluded Architectures`项设置排除的架构

**OC**

OC 关于编译内联优化的参数位于`Build Settings` -> `Apple Clang - Code Generation` ->`Optimization Level`，可选参数如下

- None[-O0]: 编译器不会优化代码，意味着更快的编译速度和更多的调试信息，默认在 Debug 模式下开启。
- Fast[-O,O1]: 编译器会优化代码性能并且最小限度影响编译时间，此选项在编译时会占用更多的内存。
- Faster[-O2]：编译器会开启不依赖空间 / 时间折衷所有优化选项。在此，编译器不会展开循环或者函数内联。此选项会增加编译时间并且提高代码执行效率。
- Fastest[-O3]：编译器会开启所有的优化选项来提升代码执行效率。此模式编译器会执行函数内联使得生成的可执行文件会变得更大。一般不推荐使用此模式。
- Fastest Smallest[-Os]：编译器会开启除了会明显增加包大小以外的所有优化选项。默认在 Release 模式下开启。
- Fastest, Aggressive Optimization[-Ofast]：启动 -O3 中的所有优化，可能会开启一些违反语言标准的一些优化选项。一般不推荐使用此模式。

**Swift**

Swift 关于编译内联优化的参数位于`Build Settings` -> `Swift Compiler - Code Generation` -> `Optimization Level`，可选参数如下

- No optimization[-Onone]：不进行优化，能保证较快的编译速度。默认在 Debug 模式开启。
- Optimize for Speed[-O]：编译器将会对代码的执行效率进行优化，一定程度上会增加包大小。默认在 Release 模式下开启。
- Optimize for Size[-Osize]：编译器会尽可能减少包的大小并且最小限度影响代码的执行效率。

## 编码人素质

1. 代码复用，禁止无脑拷贝代码，共用代码下沉为底层组件；
2. 重复功能的框架使用一套。

相关链接

- [我在 Uber 亲历的最严重的工程灾难](https://www.infoq.cn/article/asjhHAmupqtcx5oGrb4b)
- [iOS 安装包瘦身实践](https://xiaozhuanlan.com/topic/6147250839)
- [今日头条 iOS 安装包瘦身-1](https://zhuanlan.zhihu.com/p/358002160)
- [今日头条 iOS 安装包瘦身-2](https://mp.weixin.qq.com/s/PufNDzf9VfrEZdJbNcpwNQ)
- [探究WebP一些事儿](https://jelly.jd.com/article/6006b1035b6c6a01506c87a9)