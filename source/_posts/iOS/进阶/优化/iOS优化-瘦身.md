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

App Store OTA 下载大小限制：

虽然苹果历年都会调整 App 下载大小，由之前的 100M 到后来的 150M 再到现在的 200M。如今，App 下载大小超出 200 MB 时 ，会出现两种情况：
- iOS 13 以下的用户，无法通过蜂窝数据下载 App；
- iOS 13 及以上的用户，需要手动设置才可以使用蜂窝网络下载 App。

Apple __TEXT 段大小限制：

* iOS 7 之前，二进制文件中所有的 __TEXT 段总和不得超过 80 MB；
* iOS 7.X 至 iOS 8.X ，二进制文件中，每个特定架构中的 __TEXT 段不得超过 60 MB；
* iOS 9.0 之后，二进制文件中所有的 __TEXT 段总和不得超过 500 MB。

> 顺便给大家说下苹果将下载大小限制由 100M 调整到 150M 的原因是什么，主要原因就是 Uber 当年用 Swift 重构开发 APP，随着业务的增长，后期发现实在无法再将 APP 尺寸降到 100M 以下，只能联系苹果让其将下载大小提升到 150M，同时苹果的 Swift 团队还帮助添加了一些编译器选项 (-Osize)。

该文主要研究的是如何降低 APP 的下载大小的。在介绍我们作为开发者的优化方向之前，我们先看一下苹果自身对于 APP 下载大小的优化有哪些吧，我们要充分利用 Apple 自身的优化机制。

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

将 ipa 安装包后缀名改为 zip，将其解压，进入。app 文件后，就可以很直观的看到安装包的组成部分。一般主要会包括以下几个部分：
- Mach-o 可执行文件
- Frameworks 文件夹：内部会放置动态库，Swift runtime 相关 dylib 文件（如果打入的话）
- Assets.car
- xxx.lproj：原生的翻译资源
- Plugins：App Extension
- Bundle 或其他资源文件

其实核心组成部分便是**资源文件**与**Mach-O 可执行文件**两部分，这两个部分便是我们的主要瘦身方向。在瘦身过程中，应该尽量使用获得 ROI 最高的优化手段，付出更少的精力，得到更多的收益。

## 资源文件瘦身

资源文件优化方向比较多，相对优化 Mach-O 可执行文件来讲，风险也比较小。

### 去除无用的图片资源

业务的迭代开发，出现无用的图片资源是比较正常的，我们可以借助工具找出哪些图片资源没有被使用过。推荐下面两款工具。

- [LSUnusedResources](https://github.com/tinymind/LSUnusedResources)：可视化客户端工具。
- [FengNiao](https://github.com/onevcat/FengNiao.git)：命令行工具，可嵌入到Run Script中使用。

因为这类工具的原理都是在相关文件（.m、.swift 等等）中利用正则表达式检测是否有图片名称的字符，所以存在以下问题。
问题点：
- 如果代码中使用的图标名称是拼接而成的，就会误以为相关图片是废弃图片；
- 如果 Assets.xcassets 文件中直接修改了图片的名字，也会认为相关图片可能是废弃图片；

### 资源压缩

>请注意：这里的资源不包括 Assets Catalog 管理的图片等资源，原因可见下文。

资源压缩包含两种方式：

- 直接通过一些压缩工具将资源进行压缩，格式保持不变，如一些图片资源、音视频等资源；
- 还有一些资源为文本资源，如 json 文件、html 文件等，无法使用上述的方式压缩，可以采用压缩成 zip 等压缩格式的方式，可分为三步：
  1. 压缩阶段：在 Build Phase 中添加脚本，构建期间对白名单内的文本文件做 zip 压缩；
  2. 解压阶段：在 App 启动阶段，在异步线程中进行解压操作，将解压产物存放到沙盒中；
  3. 读取阶段：在 App 运行时，hook 读取这些文件的方法，将读取路径从 Bundle 改为沙盒中的对应路径；

### Assets Catalog

#### @1x 图片去除

 @1x 的图片已经没有必要继续放入 Assets 中去了，可以删除所有 @1x 的图片资源。

 > @1x 图是 iPhone 3Gs 用的，iPhone 4 开始使用 @2x 图了，iPhone 6p 开始使用 3x 图

### 图片压缩

Assets.car 文件是工程中 Asset Catalog 的构建产物。Xcode 构建过程中，在 compile asset catalog 节点时， 构建 Asset Catalog 的工具 `actool` 会首先对 Asset Catalog 中的 png 图片进行解码，得到 Bitmap 数据，然后再运用 actool 的编码压缩算法进行编码压缩处理。

`xcrun assetutil --info Assets.car`。可使用该命令检查 Assets.car 中每张图片使用的编码压缩算法。

目前`actool`会使用的压缩算法包括 `lzfse`、`palette_img`、`deepmap2`、`deepmap_lzfse` 、`zip`；影响其使用何种算法的因素包括**iOS 系统版本**、**ASSETCATALOG_COMPILER_OPTIMIZATION 设置**（位于 Build Setting 中）等；

* iOS 11.x 版本：对应的压缩算法为 lzfse、zip；
* iOS 12.0.x - iOS 12.4.x: 对应的压缩算法为 deepmap_lzfse、palette_img；
* iOS 13.x: 对应的压缩算法为 deepmap2 ；

**按照压缩比来讲 lzfse < palette_img ~= deepmap_lzfse < deepmap2**

如果设置了 `ASSETCATALOG_COMPILER_OPTIMIZATION` 为`space` 那么低版本的使用 lzfse 压缩算法的图片会变成 zip 的算法可减少 iOS11.x 及以下的 iOS 设备图片的占用大小。其他 iOS 版本的压缩算法不受这个配置的影响。

- 无损压缩通过变换图片的编码压缩算法减少大小，但是不会改变 Bitmap 数据，对于 actool 来说，它接收的输入（Bitmap 数据）没有改变，**所以无损压缩无法优化 Assets.car 的大小**。

- 使用有损压缩方式并采用合适的压缩方法是可以减小 Assets.car 的大小。可以对图片采用`RGB with palette`（调色板算法）编码方式来达到图标压缩的效果，这种编码方式进行压缩特别适合图标中颜色相对接近的图标。但是需要注意如果图片中有半透明效果，这种压缩方式可能会导致半透明的地方出现噪点，所以压缩之后请注意仔细检查一下。

> RGB with palette 编码的得到的字节流首先维护了一个颜色数组。颜色数组每个成员用 RGBA 四个分量维护一个颜色。图像中的每个像素点则存储颜色数组的下标代表该点的颜色。颜色数组维护的颜色种类和数量由图片决定，同时可以人为的限制颜色数组维护颜色的种类的上限，默认为最大值 256 种，具体原理详见底部相关链接 --【Palette Images】；

使用下文提到的 ImageOptim-CLI 工具，我们可以改变图片的编码方式为 RGB with palette，命令如下：

`imageoptim -Q --no-imageoptim --imagealpha --number-of-colors 16 --quality 40-80 ./1.png`

> --number-of-colors  控制颜色数组维护颜色的数量；--quality  控制图片的质量变为原来的百分比；命令中的数值可以在显著减少包大小的同时维持肉眼看不到的质量变化。

以图片资源举例，我们可以使用工具对其进行压缩，推荐几款工具如下：
- 网页工具：[TinyPng](https://tinypng.com/) -- 有损压缩
- 客户端工具：[TinyPNG4Mac](https://github.com/kyleduo/TinyPNG4Mac/) TinyPng的客户端工具，无需联网使用浏览器。

- GUI 客户端工具：[ImageOptim](https://github.com/ImageOptim/ImageOptim) -- 支持无损压缩及有损压缩两种形式
- [ImageOptim-CLI](https://github.com/JamieMason/ImageOptim-CLI): mac 可使用`brew install imageoptim-cli`安装，其会根据你的指定，选择性调用 JPEGmini、ImageAlpha、ImageOptim 等工具，实现中间过程自动化。

总结一下：使用 Asset Catalog 管理图片不要对图片进行无损压缩，最终起不到压缩效果，如果想要压缩，可以采用上面所提到的有损压缩方式。

### 图片资源使用 Webp 格式

谷歌开源的格式，Webp 压缩率比较高，同时支持有损和无损两种压缩模式，可以带来更小的图片体积，而且肉眼看不出差异。根据 Google 的测试，无损压缩后的 WebP 比 PNG 文件少了 26％的体积，有损压缩后的 WebP 图片相比于等效质量指标的 JPEG 图片减少了 25％~34% 的体积。

但是 WebP 与 JPG 以及 PNG 相对比，在编解码的 CPU 消耗以及解码时间上会差一些，因为编码是用户上传图片时的一次性操作，并且编码过程是在服务器后台进行，对用户的影响不对，对用户影响的可能主要是解码过程。所以，我们需要根据项目的实际情况在性能和体积上做取舍。

> 如果从服务器带宽以及流量来看，因为图片的体积变小，所以会减小带宽，降低成本。

推荐二种转 Webp 格式的方法

- [iSpart](http://isparta.github.io/)：腾讯出品，GUI工具；
- [webp工具](https://developers.google.com/speed/webp/docs/using): 在Mac下，可以使用homebrew安装webp工具--`brew install webp`；

SDWebImage 目前支持加载 Webp 格式的图片 --`pod 'SDWebImage/WebP'`。

### 资源动态化

除了上文提到的使用 On-Demand Resources 方式将部分资源放在苹果服务器之外，我们也可以将一些本地资源转移到自己的服务器上去，如一些 banner 广告图、主题资源等等。这样不仅降低了安装包大小，也将这些资源动态化了。

### 图标优化

- 使用 tint color 精简单色图标；
- 使用图标字体替换部分图标；

## Mach-O 可执行文件瘦身

在对 Mach-O 文件进行瘦身优化时，我们可以通过分析 Link Map 文件来给我们一定的数据参考，帮助我们分析 Mach-O 文件的构成。

Link Map 是编译链接时可以生成的一个 txt 文件，它生成目的就是帮助程序员分析包大小。Link Map 记录了每个方法在当前的二进制架构下占据的空间。通过分析 Link Map，我们可以了解每个类甚至每个方法占据了多少安装包空间。

开启 build setting 中的`Write Link Map File`开关，Xcode 就会生成一份 Link Map 文件；其中生成的 Link Map 文件路径如下：
`~/Developer/Xcode/DerivedData/项目/Build/Intermediates.noindex/项目.build/Debug-iphonesimulator/项目.build/项目-LinkMap-normal-x86_64.txt`

如果直接阅读 Link Map 文件，效率会比较低，也不直观，我们可以使用一些工具帮助我们分析。

[LinkMap工具地址](https://github.com/huanxsd/LinkMap)

![LinkMap效果图](../../../../img/杂项/效率软件/LinkMap.png)

### 编译选项改进

Xcode 支持编译器层面的一些优化选项，通过修改`Build Setting` 的一些相关配置，可以让我们介于更快的编译速度和更小的二进制大小并且更快的执行速度之间自由选择想要进行的优化粒度。

这种方式的性价比极高，改动一项配置，就可能会带来收益，但是有具有一定的风险，需要谨慎。

#### 去除无用架构

可以在`Build Setting`-`Excluded Architectures`项设置排除的架构

先看一下几种架构的含义

- 模拟器 32 位处理器测试需要 `i386` 架构
- 模拟器 64 位处理器测试需要 `x86_64` 架构
- 真机 32 位处理器需要 `armv7`, 或者 `armv7s` 架构
- 真机 64 位处理器需要 `arm64` 架构

armv6 | armv7 |armv7s | arm64
---------|----------|--------- |---------
 iPhone<br>iPhone2<br>iPhone3G<br>第一代和第二代 iPod Touch | iPhone4<br>iPhone4S<br>iPad1-iPad3，3、4 代 iPod Touch<br>iPad mini | iPhone5<br>iPhone5C<br>iPad4 | iPhone 5S 等剩余全部机型

所以理论上只保留 arm64 架构其实就够用了。所以可以去除 `armv6`、`armv7`、`armv7s`三种架构。

#### 使用链接时优化 LTO（Link-Time Optimization）

> 可以在`Build Setting`-`Link-Time Optimization`项设置优化方式

其提供三种选项：
- `No` 不开启链接期优化；
- `Monolithic` 生成单个 LTO 文件，每次链接重新生成，无缓存高内存消耗，参数 LLVM_LTO=YES；
- `Incremental` 生成多个 LTO 文件，增量生成，低内存消耗，参数 LLVM_LTO=YES_THIN；

LTO 能带来的优化有：
 - **将一些函数內联化**：不用进行调用函数前的压栈、调用函数后的出栈操作，提高运行效率与栈空间利用率；
 - **去除了一些无用代码**：如果一段代码分布在多个文件中，但是从来没有被使用，普通的 -O3 优化方法不能发现跨中间代码文件的多余代码，因此是一个局部优化。但是 Link-Time Optimization 技术可以在 link 时发现跨中间代码文件的多余代码
 - **对程序有全局的优化作用**：这是一个相对广泛的概念。举个例子来说，如果一个 if 方法的某个分支永不可能执行，那么在最后生成的二进制文件中就不应该有这个分支的代码

LTO 会降低编译链接的速度，所以建议在打正式包时开启；
开启了 LTO 之后，Link Map 的可读性明显降低，多出了很多数字开头的类（LTO 的全局优化导致的），所以如果需要阅读 Link Map，可以先关闭 LTO；

>LTO 虽然是链接期优化，但是仍然需要编译期参与，加入了 LTO 的编译出来的 .a 本质是 LLVM 的 BitCode，如果使用未开启 LTO 构建出来的的 .a 直接是机器码了。直接链接是无法完成 LTO 优化的。
>
> 开启 LTO 之后跨编译单元的重复代码会被链接器单独生成以 .lto.o 为后缀的目标文件进行链接。尤其是对于 Objc Runtime 需要的一些结构， **比如方法签名的 literal string、protocol 的结构等有比较大的优化**。同时开启 Oz 和 LTO 可以让外联函数都只存在一份能够最大限度的优化安装包体积（是全局的优化作用，将已经外联的函数去重）。

> **如果项目中大量的使用了 Protocol 建议还是开启这个选项。**

#### 语言编译优化

**OC**

OC 关于编译内联优化的参数位于`Build Settings` -> `Apple Clang - Code Generation` ->`Optimization Level`，可选参数如下

- None[-O0]: 编译器不会优化代码，意味着更快的编译速度和更多的调试信息，默认在 **Debug** 模式下开启。
- Fast[-O,O1]: 编译器会优化代码性能并且最小限度影响编译时间，此选项在编译时会占用更多的内存。
- Faster[-O2]：编译器会开启不依赖空间 / 时间折衷所有优化选项。在此，编译器不会展开循环或者函数内联。此选项会增加编译时间并且提高代码执行效率。
- Fastest[-O3]：编译器会开启所有的优化选项来提升代码执行效率。此模式编译器会执行函数内联使得生成的可执行文件会变得更大。一般不推荐使用此模式。
- Fastest Smallest[-Os]：编译器会开启除了会明显增加包大小以外的所有优化选项。默认在 **Release** 模式下开启。
- Fastest, Aggressive Optimization[-Ofast]：启动 -O3 中的所有优化，可能会开启一些违反语言标准的一些优化选项。一般不推荐使用此模式。

**Swift**

Swift 关于编译内联优化的参数位于`Build Settings` -> `Swift Compiler - Code Generation` -> `Optimization Level`，可选参数如下

- No optimization[-Onone]：不进行优化，能保证较快的编译速度。默认在 **Debug** 模式开启。
- Optimize for Speed[-O]：编译器将会对代码的执行效率进行优化，一定程度上会增加包大小。默认在 **Release** 模式下开启。
- Optimize for Size[-Osize]：编译器会尽可能减少包的大小并且最小限度影响代码的执行效率。

Optimize for Size 的核心原理是对重复的连续机器指令外联成函数进行复用，和函数内联的原理正好相反。因此，将其开启，能减小二进制的大小，但同时理论上会带来执行效率的额外消耗，对性能（CPU）敏感的代码使用需要评估。

#### 死代码裁剪

> 可以在`Build Setting`-`DEAD_CODE_STRIP`项设置优

在构建完成之后如果是 C、C++ 等静态的语言的代码、一些常量定义，如果发现没有被使用到将会被标记为 Dead code。开启 DEAD_CODE_STRIP = YES 这些 Dead code 将不会被打包到安装包中。在 LinkMap 这些符号也会被标记为 <<dead>>；

### 清除无用代码

### 二进制段压缩

Mach-O 文件占据了 nstall Size 中很大一部分比例，但并不是文件中的每个段 / 节在程序启动的第一时间都要被用到。可以在构建过程中将 Mach-O 文件中的这部分段 / 节压缩，然后只要在这些段被使用到之前将其解压到内存中，就能达到了减少包大小的效果，同时也能保证程序正常运行。

### __TEXT 段迁移

在链接阶段使用 -rename_section 选项将 **TEXT,**text 迁移到 **BD_TEXT,__text，减少苹果对可执行文件的加密范围，提升可执行文件的压缩效率，从而减少 Download Size。

### 多个可执行文件中去除相同代码

这里的多个可执行文件一般是指 APP 宿主程序与 Extension 程序，如果说 APP 宿主程序与 Extension 程序都以静态库的形式依赖同一个库时，就会导致两个可执行文件中都包含相同的代码；个人觉得有两种解决方案：
- 考虑到 Extension 程序相对宿主程序来说功能较小，可尽量使用原生功能，不接入三方库；
- 如果想要接入同一份库，可将该库以动态库的方式引入，最终两个可执行文件会动态链接同一份库，避免了重复代码；

## 其他

### Pod

使用 resource_bundles 配合 xcassets 的方式来集成各个插件中的资源文件，因为 resource_bundle 中的资源在构建期能经过 Xcode 的优化，而 resource 中的资源不能。并且这种形式可以将每个 pod 的资源放在自己的 Bundle 中，更方便管理。

### 合理使用动态库

### 编码人素质

1. 代码复用，禁止无脑拷贝代码，共用代码下沉为底层组件；
2. 重复功能的框架使用一套。

相关链接

- [我在 Uber 亲历的最严重的工程灾难](https://www.infoq.cn/article/asjhHAmupqtcx5oGrb4b)
- [iOS 安装包瘦身实践](https://xiaozhuanlan.com/topic/6147250839)
- [今日头条 iOS 安装包瘦身-1](https://zhuanlan.zhihu.com/p/358002160)
- [今日头条 iOS 安装包瘦身-2](https://mp.weixin.qq.com/s/PufNDzf9VfrEZdJbNcpwNQ)
- [探究WebP一些事儿](https://jelly.jd.com/article/6006b1035b6c6a01506c87a9)
- [Palette Images](http://www.manifold.net/doc/mfd9/palette_images.htm)
