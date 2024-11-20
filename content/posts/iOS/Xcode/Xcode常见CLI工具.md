---
title: Xcode常见CLI工具
date: 2024-11-20T23:56:02
categories: Xcode
tags:
  - iOS
  - Xcode
share: true
description: ""
cover.image: ""
lastmod: 2024-11-20T21:37:38
---

## 前言

Hi Coder，我是 CoderStar！

在新的一年里，祝小伙伴们工作顺利，升职加薪。

在咱们日常开发中，或多或少都会用到 Xcode 内置的一些 `CLI` 工具，但是大部分小伙伴可能只是会用到一些具体的命令，今天我们就一起来聊一聊 Xcode 内置的常见 `Command Line Tool`。

> 介绍的可能不全，大家可以去文中出现的路径下查看更多的工具。

`Command Line Tool` 本质是一个命令行工具包，内部有很多有用的工具，如 `Apple LLVM compiler`、`Make` 等等。并且并不是只有开发 Apple 应用程序才需要用到这些工具包，当我们下载 `Homebrew` 时，在安装过程中也会下载 `Command Line Tool`。

> 下文会对 `Command Line Tool` 直接缩写成 CLI，XXX 一般情况是指对应路径地址。

我们在开发者官网 [Command Line Tool](https://developer.apple.com/download/all/?q=command) 对其单独下载，当然每个版本的 Xcode 安装包内也会包含这套工具包。

其实下列有一部分工具属于 LLVM 序列，比如 `dwarfdump`、`ar`，启动本质其实为 `llvm-dwarfdump`、`llvm-ar`，都属于 `LLVM` 工具链中的一部分。

## 前置工具

在我来介绍这套工具包其他工具之前，我先来介绍两个工具，我称它们为前置工具，因为有了这两个工具，我们才能更好的使用其他的工具。

### xcode-select

这个工具可以帮助我们下载及安装 CLI，比手动下载更便捷。并且还能解决另外问题，就是如果我们装有多个 Xcode，我们在使用 CLI 相关工具时，系统就会不知道该去使用哪个版本或者哪个位置的 CLI，使用这个工具可以帮助我们设置及切换当前默认使用的 CLI。

介绍该工具常见的命令：

- `xcode-select --install`: 安装 CLI，会安装到 `/Library/Developer/CommandLineTools/`
- `xcode-select -p`: 显示当前指定的工具包所在 Xcode 路径
- `xcode-select -s <path>`: 切换默认工具包所在 Xcode 路径
- `xcode-select -r`: 重置工具包所在 Xcode 路径

> `xcode-select` 提供了一个环境变量，让你能临时使用其他环境来执行 `xcode command`，`env DEVELOPER_DIR="/Applications/Xcode-beta.app" /usr/bin/xcodebuild`

> xcode-select 选择路径不是直接选择的 CLI 路径，而是选择所在 Xcode 的路径，继而使用该 Xcode 对应的 CLI，默认情况会选择到该 Xcode 包内包含的 CLI，但是如果我们通过 Xcode Preferences 调整过该 Xcode 对应的 CLI，就会使用调整后的 CLI。

这个工具应该是 Mac 自带的工具，位于 `/usr/bin/xcode-select`，并不是跟随 CLI 工具包一块下载下来的。

### xcrun

回想我们过去在使用一些 CLI 命令的时候，会直接在终端上执行 `xcodebuild ...` 这样的方式，没有指定具体的 CLI 路径，并且我们执行 `which xcodebuild` 得到的结果是 `/usr/bin/xcodebuild`。那这个命令是怎么执行到我们通过 `xcode-select` 设置的默认 CLI 路径下呢？那就得提到我们马上要介绍的这个工具了 -- `xcrun`。

我们就以 `xcodebuild` 举例，我们通过 `which xcodebuild` 得到的结果是 `/usr/bin/xcodebuild`，也就是说我们在执行 `xcodebuild` 的时候实际上在执行 `usr/bin/xcodebuild`，那再让我们看看 `/usr/bin/xcodebuild` 下的指令是怎么配合 `xcode-select` 找到 `/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild` 的？

我们先通过 `otool -tV /usr/bin/xcodebuild` 查看其 `textsection`，得到：

```txt
(__TEXT,__text) section
_main:
0000000100003f63	pushq	%rbp
0000000100003f64	movq	%rsp, %rbp
0000000100003f67	pushq	%r14
0000000100003f69	pushq	%rbx
0000000100003f6a	movq	%rsi, %r14
0000000100003f6d	movl	%edi, %ebx
0000000100003f6f	callq	0x100003f88                     ## symbol stub for: __NSGetProgname
0000000100003f74	movq	(%rax), %rdi
0000000100003f77	leal	-0x1(%rbx), %esi
0000000100003f7a	leaq	0x8(%r14), %rdx
0000000100003f7e	movl	$0x1, %ecx
0000000100003f83	callq	0x100003f8e                     ## symbol stub for: _xcselect_invoke_xcrun
```

我们可以发现该命令调用 `_xcselect_invoke_xcrun` 函数。

然后我们通过 `nm /usr/bin/xcodebuild` 查看其 `name list`

```txt
                 U __NSGetProgname
0000000100008018 d __dyld_private
0000000100000000 T __mh_execute_header
0000000100003f63 t _main
0000000100008010 s _shim_marker
                 U _xcselect_invoke_xcrun
                 U dyld_stub_binder
```

通过 `_xcselect_invoke_xcrun` 前面的 `U` 标识我们可以知道该函数是一个外部符号，是另外一个动态库去处理的。

后面我们通过 [Swift-Swiftc](https://dongaxis.github.io/2016/04/28/Swift-Swiftc/) 可以知道更详细流程，这里只说结论：

```txt
libxcselect.dylib
👇🏻
_xcselect_invoke_xcrun
👇🏻
libxcrun.dylib
👇🏻
xcrun_main
```

我们后面最后实际上会调用到 `xcrun_main` 函数，其内部就会根据 `xcode-select` 等设置的情况选择合适的 CLI 路径，具体执行的顺序可见 [Developer Binaries on OS X, xcode-select and xcrun](https://macops.ca/developer-binaries-on-os-x-xcode-select-and-xcrun/) 。

xcrun（`Xcode Command Line Tool Runner`） 是 Xcode 基本的命令行工具，使用它来调用其他 CLI 工具，这时候你应该就知道为啥需要它来调用其他 CLI 工具了。

我们执行 `xcrun` 命令时实际上也是走的 `/usr/bin/xcrun`，其内部也是上面一套流程，准确而言，在这套 CLI 工具包中位于 `/usr/bin` 路径下的命令都是上面那个流程，也就是说下面几个命令是等价的：

- xcodebuild
- /usr/bin/xcodebuild
- xcrun xcodebuild
- Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild

当然这套工具包有些命令不在 `/usr/bin` 路径下，我们就需要在命令前加上 `xcrun` 了，如 `swift-demangle`，如果我们直接使用 `swift-demangle` 就会出现命令找不到的错误，使用 `xcrun swift-demangle` 这种方式即可。

相关命令：

- xcrun --find clang // 找到指定工具路径
- xcrun --sdk iphoneos --find pngcrush
- xcrun --sdk macosx --show-sdk-path

通过 `xcode-select` 安装的 CLI 路径位于：`/Library/Developer/CommandLineTools/`。
Xcode 内嵌的 CLI 路径位于：`/Applications/Xcode.app/Contents/Developer/usr/bin`

还有一点需要注意的是，xcrun 并不是只会寻找 `xcode-select` 设置下的路径，还会寻找 Xcode 另外的一些路径来执行命令，包括

- Developer/usr
- Developer/Platforms
- Developer/ToolChain

例子如下：

xcodebuild：`/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild`
swift-demangle：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swift-demangle`

[深入浅出 Xcode Command Lines Tool - 初探](https://juejin.cn/post/6844904052271087624)
[深入浅出 Xcode 命令列 - libxcselect.dylib](https://juejin.cn/post/6844904052271087623)
[深入浅出 Xcode 命令列 - xcrun](https://juejin.cn/post/6844904052275298317)

关于这两个工具有开源的实现 [xcode-tools](https://github.com/b-man/xcode-tools)。

## 符号表相关

先简单介绍一下 `DWARF` 以及 `dSYM`。

`DWARF` 与 `dSYM` 的关系是，`DWARF` 是文件格式，而 `dSYM` 往往指一个单独的文件。在 Xcode 中如果不做特殊指定，`debug information` 是被保存在 `executable` 文件中。因为 `DWARF` 的存在我们才可以在 `debug` 时看到函数名称等信息，因为 `dSYM` 文件的存在，我们才可以符号化，解 Crash。

> 关于符号解析之前有过一篇文章 [iOS符号化解析](../../进阶/iOS%20符号化浅析)。

### dwarfdump

作用：解析目标文件，存档和 `.dSYM` 包中的 `DWARF` 节，并以人类可读的形式打印其内容；
使用场景：Crash 符号化；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/dwarfdump`；

```shell
# 使用示例
dwarfdump -h

# 查看 xx.app 文件的 UUID
dwarfdump --uuid xx.app/xx

# 查看 xx.app.dSYM 文件的 UUID
dwarfdump --uuid xx.app.dSYM

# 导出debug_info 的信息到文件 debug_line.txt 中
dwarfdump --debug-info xx.app.dSYM > debug_info.txt

#  出debug_line 的信息到文件 debug_line.txt 中
dwarfdump --debug-line xx.app.dSYM > debug_line.txt

# 校验DWARF的有效性
dwarfdump --verify iOSTest.app.dSYM

# 查找指定地址的相关信息
# 一般用在Crash解析时
dwarfdump --arch arm64 --lookup 0x100006694 iOSTest.app.dSYM

```

更多命令可见 [llvm-dwarfdump](https://llvm.liuxfe.com/docs/man/llvm-dwarfdump)。

### dsymutil

作用：可以使用 `dsymutil` 从 二进制中 中提取 `dSYM` 文件以及对 `dSYM` 文件进行一些操作；
使用场景：当 `dSYM` 文件丢失后，可以将其作为找回 `dSYM` 文件的一种方式；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/dsymutil`；

```shell

# 从二进制文件中还有`DSYM`信息的二进制包中抽取形成`.dysm`文件
dsymutil XXX

# 使用指定的符号映射更新现有的 dSYM
# 处理开启bitcode选项的dsym文件
dsymutil -symbol-map /Users/XXXXX/Library/Developer/Xcode/Archives/2019-09-27/YYYY.xcarchive/BCSymbolMaps 0f1e9458-9741-36fb-b47c-694546728ea1.dSYM
```

### symbolicatecrash

作用：是一个 `perl` 脚本，里面整合了逐步解析的操作（可以将命令拷贝出来，直接进行调用）；
场景：Crash 文件符号化；
路径：`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`；

```shell
# 需要先运行该命令，不然下面 symbolicatecrash命令会出现
# Error: "DEVELOPER_DIR" is not defined at ./symbolicatecrash line 69.
export DEVELOPER_DIR="/Applications/XCode.App/Contents/Developer"

# 运行命令前需要将崩溃日志、 dSYM 以及 symbolicatecrash 复制到同一个目录下
symbolicatecrash log.crash -d xxx.app.dSYM > symbol.log
```

iOS 15 之后，崩溃日志的格式为 JSON 形式，新的脚本地址为 `/Applications/Xcode.app/Contents/SharedFrameworks/CoreSymbolicationDT.framework/Versions/A/Resources/CrashSymbolicator.py`

这个 py 脚本为 python3 写的，所有也需要 python3 环境下去调用，命令为

```shell
python3 /Applications/Xcode.app/Contents/SharedFrameworks/CoreSymbolicationDT.framework/Versions/A/Resources/CrashSymbolicator.py 要解析的崩溃日志 -d dSYM所在路径 -o 解析后的日志地址
```

解析之后的结果也是 JSON 形式的。

当然我们也可以将其拷贝出来使用，但是需要在首行加上下列代码

```shell
import sys

sys.path.append("/Applications/Xcode.app/Contents/SharedFrameworks/CoreSymbolicationDT.framework/Versions/A/Resources/")
```

### atos

作用：Crash 符号化；
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/atos`；

```shell
# 0x0000000100298000为 load address； 0x000000010029e694为 symbol address
# 最后一个i表示显示内联函数
atos -arch arm64  -o iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest -l 0x0000000100298000 0x000000010029e694 -i
```

## 构建相关

### xcodebuild

作用：我们可以使用其对 Xcode 工程进行清理，分析，构建，测试，存档；
场景：CI 构建等；
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild`；

可以通过 `man xcodebuild` 查看手册。

> 其中 `man` 命令就是用来查看指定命令的使用手册。

```shell

# 不传递任何构建参数，默认为 Xcode.app 最近使用的 scheme 和 配置
xcodebuild

# 清理
xcodebuild clean -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE}

# 构建
xcodebuild archive -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH}

## 存档
xcodebuild -exportArchive -archivePath $ARCHIVE_PATH -exportPath ${IPA_PATH} -exportOptionsPlist ${EXPORTOPTIONSPLIST_PATH}
```

- `xctool`：`xctool` 是 `facebook` 推出的用于替换 `xcodebuild` 的更易于测试 iOS 和 mac 应用程序的命令行工具，特别适用于 iOS App 的持续集成；
- `xcbuild`：`xcbuild` 是一个兼容 `Xcode` 的编译工具，它能使编译更快快速，更友好的编译过程日志，可以运行在多个平台（主要指 OS X 和 Linux）；

### altool

作用：使用其验证 ipa 以及上传 ipa 到 Store；
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/altool`

```shell
# 验证
# version、build号是否正确等case
xcrun altool --validate-app -f xxx.ipa -t ios --apiKey xxx --apiIssuer xxx --verbose

# 上传
xcrun altool --upload-app -f xxx.ipa -t ios --apiKey xxx --apiIssuer xxx --verbose
```

## 工具链相关

### swiftc

作用：swift 语言的编译前端；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swiftc`；

> `swiftc` 只是一个替身，原身是 `swift-frontend`。

### clang

作用：oc 语言的编译前端；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang`；

### sourcekit-lsp

LSP（Language-Server-Protocol）开源的语言服务器协定。由红帽、微软和 Codenvy 联合推出，可以让不同的程序编辑器与集成开发环境（IDE）方便嵌入各种程序语言，允许开发人员在最喜爱的工具中使用各种语言来撰写程序，`SourceKit-LSP` 是 Apple 维护的用于 Swift 的 LSP；其的存在允许我们使用其他 IDE 开发 Swift，如 VSCode；

路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/sourcekit-lsp`

### 工具相关

### actool

作用：对 项目中 Assets 的文件进行压缩、处理，生成 `.car` 文件；
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/actool`；

`actool` 并非一个脚本，而是一个编译完成的二进制文件，所以 `compile asset catalog` 的过程是一个黑盒。

### swift-demangle

在 Swift 中因为命名空间的原因，需要对类名进行 `mangle`，如果需要显示正确名称，自然也需要 `demangle`。其实两个方法实现大家可以通过以下链接查看，

`mangle`：[copySwiftV1MangledName函数](https://github.com/opensource-apple/objc4/blob/cd5e62a5597ea7a31dccef089317abb3a661c154/runtime/objc-runtime-new.mm#L859)，

`demangle`：[copySwiftV1DemangledName](https://github.com/opensource-apple/objc4/blob/cd5e62a5597ea7a31dccef089317abb3a661c154/runtime/objc-runtime-new.mm#L813)

当然 Apple 本身也为我们特意准备了一个 CLI 工具 --`swift-demangle` 来方便我们。

```shell
命令：xcrun swift-demangle  _TtC7iOSTest27PickImageDemoViewController
结果：_TtC7iOSTest27PickImageDemoViewController ---> iOSTest.PickImageDemoViewController

命令：xcrun swift-demangle --compact _TtC7iOSTest27PickImageDemoViewController
结果：iOSTest.PickImageDemoViewController
```

我们还可以在 [SwiftDemangle.h](https://github.com/apple/swift/blob/main/include/swift/SwiftDemangle/SwiftDemangle.h) 看到 swift 底层该函数名称 -- `swift_demangle_getDemangledName`，项目中需要加入 `libswiftDemangle.tbd`。

### genstrings & ibtool

#### genstrings

作用：`genstrings` 工具从指定的 C 或者 Objective-C 源文件生成 `.strings` 文件；
命令：`genstrings -a /path/to/source/files/*.m`

#### ibtool

正如 `genstrings` 作用于源代码，而 `ibtool` 作用于 `XIB` 文件；
命令：`$ ibtool --generate-strings-file Localizable.strings en.lpoj/Interface.xib`

本地化是它的主要功能，ibtool 还拥有对 Interface Builder 文档有效的其他几个功能。

- --convert： 更改所有对类名的引用
- --upgrade： 将文档升级到最新版
- --enable-auto-layout：启用自动布局
- --update-frames：更新框架
- --update-constraints：更新约束

### xed

作用：这个命令可以简单地打开 Xcode，可以用在 CLI 程序内；
命令：`xed XXX.xcworkspace`

### agvtool

作用：读取以及写入 Xcode 工程 `Info.plist` 中的版本号等信息。

- `agvtool what-version`：返回 build 号；
- `agvtool next-versio`：build 号加上，写入 `Info.plist`；

### plutil

`plutil` 多用于处理 plist 文件的校验和修改，其还可以用用于校验多语言文件 `.strings` 的格式问题。

`plutil -lint path/Localizable.strings`

### 模拟器

- `open -a Simulator`： 打开一个模拟器（会是最近打开过的那一个）
- `xcrun simctl list`：列出安装的所有设备
- `xcrun simctl boot XXX`： XXX 为设备唯一标识（长这样：`413E2CA2-420B-4074-BD64-47333E167A20`），`xcrun simctl list` 结果里面会有。

## Mach-O 操作相关

其实关于 Mach-O 操作的大部分工具都是 LLVM 下面的，包括 `otool`、`objdump`、`nm`、`dwarfdump` 等等，其命令本质上都是一个替身，背后实际上都是 `llvm-XXX` 命令的原身。

### nm

作用：`nm` 命令是 linux 下自带的特定文件分析工具，一般用来检查分析二进制文件、库文件、可执行文件中的符号表，返回二进制文件中各段的信息，查看二进制目标文件的符号，主要就是函数名称以及全局变量；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/nm`；

```shell
# 得到XXX中的程序符号表
nm XXX

# 查看所有符号，会打印出符号来源哪个地方
nm -nm XXX

# 找到未定义的符号，也就是外部符号
nm -u XXX
```

前面我们曾经查看过 `xcodebuild` 的符号，输出如下。

```txt
                 U __NSGetProgname
0000000100008018 d __dyld_private
0000000100000000 T __mh_execute_header
0000000100003f63 t _main
0000000100008010 s _shim_marker
                 U _xcselect_invoke_xcrun
                 U dyld_stub_binder
```

上述中间的大写字母就是后面对应符号的类型，其中全部的类型包括：

- A 该符号的值在今后的链接中将不再改变；
- B 该符号放在 BSS 段中，通常是那些未初始化的全局变量；
- D 该符号放在普通的数据段中，通常是那些已经初始化的全局变量；
- T 该符号放在代码段中，通常是那些全局非静态函数；
- U 该符号未定义过，需要自其他对象文件中链接进来；
- W 未明确指定的弱链接符号；同链接的其他对象文件中有它的定义就用上，否则就用一个系统特别指定的默认值。

### otool & objdump

为什么要把这两个工具放到一起说呢？因为这两个工具之间有一定的关系。其实 `otool` 本质上就是 `objdump` 的一层 wrapper，底层其实都是使用 `objdump` 的实现。

比如 `otool -L XXX` 本质就等价 `objdump --macho -dylibs-used XXX`，更多详细的转换规则可见 [otool.html](https://wangwangok.gitbooks.io/mac-terminal-tool/content/otool.html)。

两者作用： 针对目标文件的展示工具，用来发现应用中使用到了哪些系统库，调用了其中哪些方法，使用了库中哪些对象及属性。

#### otool

路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/otool`

还有一个比这个更方便的工具，`jtool`

> MachOView 算是这个 CLI 工具的 GUI 工具了。

```shell
# 查看使用到哪些动态库，一般是涉及到 	/usr/lib/ 	/System/Library/Frameworks/  @rpath 这三个位置，如果没有自己的动态库，就没有后面的 @rpath
otool -L XXX

# 筛选是否链接了xxx库
otool -L XXX | grep "xxx"

# 查看汇编码
otool -tV XXX

# 输出Object-C类结构以及定义的方法
otool -ov XXX

# 查看头部内容
otool -h XXX

# 查看 load commands
otool -l XXX

# 查看该应用是否砸壳
# 看输出结果的cryptid参数，其中0：砸壳、1：未砸壳。
otool -l XXX | grep -B 2 crypt

# 查看代码段起始地址
otool -l iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest | grep __TEXT -C 5

# 查看重定位符号表
otool -r XXX
```

#### objdump

Linux 的文件也可以使用这个工具。

路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/objdump`

```shell
# 查看mach-header
objdump --macho -private-header XXX

# 查看text段
objdump --macho -d XXX

# 查看符号表
objdump --macho --syms XXX

# 查看导出符号表
objdump --macho --exports-trie XXX

# 查看间接符号表
objdump --macho --indirect-symbols XXX

# 查看重定位符号表
objdump --macho --reloc XXX
```

> 其实 `objdump` 的功能之一可以代替 `nm` 命令，其中 `objdump --macho --syms XXX` 也可以输出符号表。

### strings

作用：查看二进制文件中的字符串；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/strings`；

```shell
# 查看指定字符串
strings XXX | grep "xxx"
```

### lipo

`lipo` 源于 `mac` 系统要制作兼容 `powerpc` 平台和 intel 平台的程序，`lipo` 是一个在 `Mac OS X` 中处理通用程序（Universal Binaries）的工具。

```shell
### 查看查看静态库支持的 CPU 架构
lipo -info frameworkName.framework/frameworkName
lipo -info frameworkName.a

### 合并静态库
lipo -create 静态库存放路径 1 静态库存放路径 2 ... -output 整合后存放的路径

lipo -create frameworkName-armv7.a frameworkName-armv7s.a frameworkName-i386.a -output frameworkName.a
lipo -create frameworkNameOne.framework/frameworkNameOne frameworkNameTwo.framework/frameworkNameTwo -output frameworkName.framework

### 静态库拆分
lipo 静态库源文件路径 -thin CPU 架构名称 -output 拆分后文件存放路径
lipo libname.a -thin armv7 -output libname-armv7.a

### 擦除指定架构
lipo -remove XXX.a arm64 -output XXX.a
```

### ar

作用：建立、修改静态库；
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/libtool`；

```shell
ar -x XXX

-d 删除备存文件中的成员文件。
-m 变更成员文件在备存文件中的次序。
-p 显示备存文件中的成员文件内容。
-q 将问家附加在备存文件末端。
-r 将文件插入备存文件中。
-t 显示备存文件中所包含的文件。
-x 从库文件取出成员文件（.O文件），使用其之前需要将多架构文件拆成单文件架构；
```

### file

我们可以使用 `file` 命令来区分动态库与静态库。

如 `file Flutter` 得到，我们可以很容易看到 `dynamically` 关键字，其为一个动态库

```text
Flutter: Mach-O 64-bit dynamically linked shared library arm64
```

如 `file CSPickerView`，得到结果如下：[CSPickerView](https://github.com/Coder-Star/CSPickerView) 为一个静态库

```text
CSPickerView: Mach-O universal binary with 5 architectures: [i386:current ar archive] [arm_v7] [arm_v7s] [x86_64] [arm64]
CSPickerView (for architecture i386):	current ar archive
CSPickerView (for architecture armv7):	current ar archive
CSPickerView (for architecture armv7s):	current ar archive
CSPickerView (for architecture x86_64):	current ar archive
CSPickerView (for architecture arm64):	current ar archive
```

### class-dump

[下载地址](http://stevenygard.com/projects/class-dump/)

这是一个命令行实用程序，用于检查存储在 `Mach-O` 文件中的 `Objective-C` 运行时信息。它为类、类别和协议生成声明。这与使用 'otool -ov' 提供的信息相同，但呈现为普通的 Objective-C 声明，因此更加紧凑和可读。

> 如果安装了 `MonkeyDev`，内置了 `class-dump`，就不用再特意去安装了。

### dyldinfo

```shell
# 查看一个可执行文件需要 Fix-up 的内容
xcrun dyldinfo -rebase -bind XXX
```

## 最后

当然，CLI 命令还有很多，这里只是列举了一些常见的，对于其他的，大家可以直接通过开头提到的一些路径去查找。

要更加努力呀！

Let's be CoderStar!

- [iOS开发中常用命令工具（xcode-select、lipo、xcrun等](https://www.jianshu.com/p/a4731527ca73)
- [Xcode 相关终端工具使用](https://www.hanleylee.com/usage-of-xcode-terminal-tools.html)
- [Building from the Command Line with Xcode FAQ](https://developer.apple.com/library/archive/technotes/tn2339/_index.html)
- [Xcode 工具链](https://chaosky.tech/2017/01/04/Xcode-Toolchain/#%E5%85%B6%E4%BB%96%E5%B7%A5%E5%85%B7)
