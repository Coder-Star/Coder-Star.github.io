---
title: iOS优化-瘦身
category:
  - iOS
  - 优化
tags:
  - iOS
  - 优化
date: 2021-01-19 08:50:18
---

iOS 优化系列目录如下：

- [iOS 优化-总览](../iOS优化-总览)
- [iOS 优化-UI](../iOS优化-UI)
- [iOS 优化-内存](../iOS优化-内存)
- [iOS 优化-启动优化](../iOS优化-启动优化)
- [iOS 优化-瘦身](../iOS优化-瘦身)

## App Thinning

### Slicing

App Thinning 会专门针对不同的设备来选择只适用于当前设备的内容以供下载。比如，iPhone 6 只会下载 2x 分辨率的图片资源，iPhone 6plus 则只会下载 3x 分辨率的图片资源。

### bitcode

Bitcode 是一个编译好的程序的中间表示形式。上传到 iTunes Connect 中的包含 Bitcode 的 app 将会在 App store 中进行链接和编译。苹果会对包含 Bitcode 的二进制 app 进行二次优化，而不需要提交一个新的 app 版本到 app store 中。属于 Apple 内部的优化，但需要注意；

- 全部都要支持。我们所依赖的静态库、动态库、Cocoapods 管理的第三方库，都需要开启 Bitcode。否则打包会编译失败。具体错误会在 Xcode 中指出。
- crash 定位。开启 Bitcode 后最终生成的可执行文件是 Apple 自动生成的，同时会产生新的符号表文件，所以我们无法使用自己包生成的 dYSM 符号化文件来进行符号化。

**Bitcode 开启 Build Settings -> Enable Bitcode -> 设置为 YES**

### On-Demand Resources

ODR（on-demand resources 随需应变资源)是 iOS 减少应用资源消耗的另外一种方法。比如多级游戏，用户需要的通常都是他们当前的级数以及下一级。ODR 意味着用户可以下载他们需要的几级游戏。随着你的级数不断增加，应用再下载其他级数，并将用户成功过关的级数删掉。 
当用户点击应用内容的时候，就会动态从 App Store 上进行下载，也就是说用户只会在需要的时候占用存储空间。这项功能有趣之处还在于当将这些内容在后台进行下载之后，当存储空间紧张的时候会自动进行删除。

配合 Xcode 中的 Resources Tag 使用，

## 删除没有使用的图片

### LSUnusedResources 工具

客户端工具，不能执行脚本 [下载地址](https://github.com/tinymind/LSUnusedResources)

### FengNiao

脚本工具， [下载地址](https://github.com/onevcat/FengNiao.git)

```
git clone https://github.com/onevcat/FengNiao.git
cd FengNiao
./install.sh
```

## @1x 图片去除

由于当前 App 不适配非 Retina 屏幕，所以@1x 的图片没必要继续放入项目当中，核对后删除所有@1x 的图片资源。（1x图是3Gs用的，4都开始使用2x图了，6p开始使用3x图）

## 图片、视频压缩

### 使用 WebP 格式

谷歌开源的格式，WebP 压缩率比较高，而且肉眼看不出差异，同时支持有损和无损两种压缩模式。

WebP 在 CPU 消耗和解码时间上会比 PNG 高两倍。所以，我们有时候还需要在性能和体积上做取舍。

如果图片大小超过了 100KB，你可以考虑使用 WebP；而小于 100KB 时，你可以使用网页工具 TinyPng（https://tinypng.com/）或者 GUI 工具 ImageOptim（https://github.com/ImageOptim/ImageOptim）进行图片压缩。这两个工具的压缩率没有 WebP 那么高，不会改变图片压缩方式，所以解析时对性能损耗也不会增加。

** 格式转换工具**

1. 腾讯开发的 iSpart http://isparta.github.io/
2. Google 提供的工具 https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html

## 代码瘦身

### 通过 Appcode 找出无用代码

使用 APPCode 提供的静态分析 Code->Inspect Code

### LinkMap 结合 Mach-O 找出无用代码

### 编码人素质

1. 代码复用，禁止拷贝代码，共用代码下沉为底层组件
2. 尽量将图片资源放入 Images.xcassets 中，包括 pod 库的图片。 Images.xcassets 中的图片加载后会有缓存，提升加载速度，并且在最终打包时会自动进行压缩，再根据最终运行设备进行 2x 和 3x 分发。
3. 对于一些非必要的大资源文件，例如字体库、换肤资源，可以在 App 启动后通过异步下载到本地，而不用直接放在 ipa 包内。

## 其他优化

### 编译器优化

Xcode 支持编译器层面的一些优化优化选项，可以让我们介于更快的编译速度和更小的二进制大小并且更快的执行速度之间自由选择想要进行的优化粒度。

去除无用架构，如可以去除armv6、armv7、armv7s。

armv6
- iPhone
- iPhone2
- iPhone3G
- 第一代和第二代iPod Touch

armv7
- iPhone4
- iPhone4S
- iPad1-iPad3，3、4代iPod Touch
- iPad mini

armv7s
- iPhone5
- iPhone5C
- iPad4
  
其他全部是arm64架构，理论上只保留arm64架构其实就够用了。

**OC**

OC 关于编译内联优化的参数位于`Build Settings` -> `Apple Clang - Code Generation` ->`Optimization Level`，可选参数如下

- None[-O0]: 编译器不会优化代码，意味着更快的编译速度和更多的调试信息，默认在 Debug 模式下开启。
- Fast[-O,O1]: 编译器会优化代码性能并且最小限度影响编译时间，此选项在编译时会占用更多的内存。
- Faster[-O2]：编译器会开启不依赖空间/时间折衷所有优化选项。在此，编译器不会展开循环或者函数内联。此选项会增加编译时间并且提高代码执行效率。
- Fastest[-O3]：编译器会开启所有的优化选项来提升代码执行效率。此模式编译器会执行函数内联使得生成的可执行文件会变得更大。一般不推荐使用此模式。
- Fastest Smallest[-Os]：编译器会开启除了会明显增加包大小以外的所有优化选项。默认在 Release 模式下开启。
- Fastest, Aggressive Optimization[-Ofast]：启动 -O3 中的所有优化，可能会开启一些违反语言标准的一些优化选项。一般不推荐使用此模式。

**Swift**

Swift 关于编译内联优化的参数位于`Build Settings` -> `Swift Compiler - Code Generation` -> `Optimization Level`，可选参数如下

- No optimization[-Onone]：不进行优化，能保证较快的编译速度。默认在 Debug 模式开启。
- Optimize for Speed[-O]：编译器将会对代码的执行效率进行优化，一定程度上会增加包大小。默认在 Release 模式下开启。
- Optimize for Size[-Osize]：编译器会尽可能减少包的大小并且最小限度影响代码的执行效率。
