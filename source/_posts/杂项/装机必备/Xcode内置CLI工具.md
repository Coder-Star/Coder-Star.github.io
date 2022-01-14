---
title: Xcode 内置 CLI 工具
category:
  - 杂项
tags:
  - Xcode
date: 2022-01-06 16:57:27
---

## 前言

Hi Coder，我是 CoderStar！

今天我们来聊一聊我们平时可能会用到的一些 CLI 工具或者脚本。主要分为两类，一种是 Mac 下自带的命令行工具，另一种是 Xcode 附带的。

## xcrun

`xcode command-line tool runner`，xcrun 是 Xcode 基本的命令行工具。使用它可以调用其他工具。工具所在路径为

* `/usr/bin/`
* `/Applications/Xcode.app/Contents/Developer/usr/bin`

使用 `xcrun xcodebuild` 这种方式会在上述多个路径去查找指定工具，当机器有多个 Xcode 环境，会按照通过`xcode-select`命令设置的默认 Xcode 环境去寻找工具。

比如`xcodebuild`、`swift-demangle`，当然有些工具是可以直接调用的，比如`xcodebuild`，但是有些工具必须借助`xcurn`，比如`swift-demangle`。

将 swift 函数名`demangle`，

```shell
xcrun swift-demangle  _TtC7iOSTest27PickImageDemoViewController
_TtC7iOSTest27PickImageDemoViewController ---> iOSTest.PickImageDemoViewController

xcrun swift-demangle --compact _TtC7iOSTest27PickImageDemoViewController
iOSTest.PickImageDemoViewController
```

xcrun --find clang
xcrun --sdk iphoneos --find pngcrush
xcrun --sdk macosx --show-sdk-path

xcrun altool --validate-app -f ${IPA_PATH}/${SCHEME_NAME}.ipa -t ios --apiKey xxx --apiIssuer xxx --verbose

xcrun altool --upload-app -f ${IPA_PATH}/${SCHEME_NAME}.ipa -t ios --apiKey xxx --apiIssuer xxx --verbose

[Developer Binaries on OS X, xcode-select and xcrun](https://macops.ca/developer-binaries-on-os-x-xcode-select-and-xcrun/)
[深入浅出 Xcode Command Lines Tool - 初探](https://juejin.cn/post/6844904052271087624)
[深入浅出 Xcode 命令列 - libxcselect.dylib](https://juejin.cn/post/6844904052271087623)
[深入浅出 Xcode 命令列 - xcrun](https://juejin.cn/post/6844904052275298317)

## Mac

### otool

`otool -l iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest | grep __TEXT -C 5`

执行命令后，结果如下，可以看到 dSYM 中代码段起始地址为 `0x0000000100000000`，一般情况下都为这个值。

### xcode-select

显示当前选择的 Xcode。

* xcode-select --install: 安装 command line tools，会按照到`/Library/Developer/CommandLineTools/`
* xcode-select -p: 显示当前指定的 xcode 路径
* xcode-select -s <path>: 指定 xcode 路径
* xcode-select -r: 重置 xcode 路径

## Xcode

### actool

对 Assets 的文件进行压缩、处理

### atos

符号化解析

```shell
atos -arch arm64  -o iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest -l 0x0000000100298000 0x000000010029e694 -i
```

### xcodebuild

可以通过`man xcodebuild`查看手册。

`xcodebuild clean -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE} || exit`

`xcodebuild clean -project ${PROJECT_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE} || exit`

`xcodebuild archive -project ${PROJECT_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH}`

`xcodebuild archive -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH} -quiet || exit`

`xcodebuild -exportArchive -archivePath $ARCHIVE_PATH -exportPath ${IPA_PATH} -exportOptionsPlist ${EXPORTOPTIONSPLIST_PATH} -quiet || exit`

我们可以使用其进行清理，分析，构建，测试，存档。

- xctool：xctool 是 facebook 推出的用于替换 xcodebuild 的更易于测试 ios 和 mac 应用程序的命令行工具，特别适用于 ios app 的持续集成。
- xcbuild：xcbuild 是一个兼容 Xcode 的编译工具，它能使编译更快快速，更友好的编译过程日志，可以运行在多个平台（主要指 OS X 和 Linux）

### symbolicatecrash

符号化 crash 文件

### lipo

lipo 源于 mac 系统要制作兼容 powerpc 平台和 intel 平台的程序，lipo 是一个在 Mac OS X 中处理通用程序（Universal Binaries）的工具。

**lipo -info**

查看查看静态库支持的 CPU 架构
lipo -info frameworkName.framework/frameworkName
lipo -info frameworkName.a

**合并静态库**

lipo -create 静态库存放路径 1 静态库存放路径 2 ... -output 整合后存放的路径

lipo -create frameworkName-armv7.a frameworkName-armv7s.a frameworkName-i386.a -output frameworkName.a

lipo -create frameworkNameOne.framework/frameworkNameOne frameworkNameTwo.framework/frameworkNameTwo -output frameworkName.framework

**静态库拆分**

lipo 静态库源文件路径 -thin CPU 架构名称 -output 拆分后文件存放路径

lipo libname.a -thin armv7 -output libname-armv7.a

**擦除指定架构**

lipo XXX.a -remove arm64 -output XXX.a

### libtool

### file

我们可以使用`file`命令来区分动态库与静态库。

如`file Flutter`得到，我们可以很容易看到`dynamically`关键字，其为一个动态库

```text
Flutter: Mach-O 64-bit dynamically linked shared library arm64
```

如 `file CSPickerView`，得到结果如下：[CSPickerView](https://github.com/Coder-Star/CSPickerView)为一个静态库

```text
CSPickerView: Mach-O universal binary with 5 architectures: [i386:current ar archive] [arm_v7] [arm_v7s] [x86_64] [arm64]
CSPickerView (for architecture i386):	current ar archive
CSPickerView (for architecture armv7):	current ar archive
CSPickerView (for architecture armv7s):	current ar archive
CSPickerView (for architecture x86_64):	current ar archive
CSPickerView (for architecture arm64):	current ar archive
```

### dwarfdump

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

# 查找指定地址的相关信息
dwarfdump --arch arm64 --lookup 0x100006694 iOSTest.app.dSYM

# 校验DWARF的有效性
dwarfdump --verify iOSTest.app.dSYM
```

找到指定位置的函数符号

```shell
dwarfdump --arch arm64 --lookup 0x100006694 iOSTest.app.dSYM

或者

dwarfdump iOSTest.app.dSYM --lookup 0x100006694
```

### class-dump

### dumpdecrypted

## Xcode 环境参数

- 关闭 Xcode，打开终端输入`defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES`，然后项目 Build 的时候可以在 Xcode 顶部看到项目编译时间

- `defaults write com.apple.Xcode PBXNumberOfParallelBuildSubtasks 8`  提高 XCode 编译时使用的线程数

## 最后

要更加努力呀！

Let's be CoderStar!

- [iOS开发中常用命令工具（xcode-select、lipo、xcrun等](https://www.jianshu.com/p/a4731527ca73)
- [Xcode 相关终端工具使用](https://www.hanleylee.com/usage-of-xcode-terminal-tools.html)
- [Building from the Command Line with Xcode FAQ](https://developer.apple.com/library/archive/technotes/tn2339/_index.html)
