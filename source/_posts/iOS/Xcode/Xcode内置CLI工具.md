---
title: Xcode 内置 CLI 工具
category:
  - iOS
  - Xcode
tags:
  - Xcode
date: 2022-01-06 16:57:27
---

## 前言

Hi Coder，我是 CoderStar！

今天我们来聊一聊`Command Lines Tool`。

`Command Lines Tool`本质是一个命令行工具包，内部有很多有用的工具，如`Apple LLVM compiler`、`Make`等等。并且并不是只有开发 Apple 应用程序才需要用到这些工具包，当我们使用`Homebrew`在安装一些`python`库或者`js`库时，都会提示需要`Command Line Tools`。

> 后文会对`Command Line Tools`直接缩写成 CLI。

我们在开发者官网[Command Line Tools](https://developer.apple.com/download/all/?q=command)对其单独下载，当然每个版本的Xcode安装包内也会包含这套工具包。

其实下列有一部分工具属于 LLVM 序列，比如`dwarfdump`、`ar`，启动本质为`llvm-dwarfdump`、`llvm-ar`，都属于 LLVM 工具链中的一部分。

## 前置工具

在我来介绍这套工具包其他工具之前，我先来介绍两个工具，我称为前置工具，因为有了这两个工具，我们才能更好的使用其他的工具。

### xcode-select

这个工具可以帮助我们安装下载 CLI，比手动下载更便捷。并且还能解决另外问题，就是如果我们装有多个 Xcode，我们在使用 CLI 相关工具时，系统就会不知道该去使用哪个版本或者哪个位置的 CLI，使用这个工具可以帮助我们设置及切换当前默认使用的 CLI。

介绍该工具常见的命令：

* `xcode-select --install`: 安装 CLI，会安装到`/Library/Developer/CommandLineTools/`
* `xcode-select -p`: 显示当前指定的工具包所在 Xcode 路径
* `xcode-select -s <path>`: 切换默认工具包所在 Xcode 路径
* `xcode-select -r`: 重置工具包所在 Xcode 路径

> `xcode-select`提供了一个环境变量，让你能临时使用其他环境来执行`xcode command`，`env DEVELOPER_DIR="/Applications/Xcode-beta.app" /usr/bin/xcodebuild`

> xcode-select 选择路径不是直接选择的 CLI 路径，而是选择所在 Xcode 的路径，继而使用该 Xcode 对应的 CLI，默认情况会选择到该 Xcode 包内包含的 CLI，但是如果我们通过 Xcode Preferences 调整过该 Xcode 对应的 CLI，就会使用调整后的 CLI。

这个工具应该是 Mac 自带的工具，位于`/usr/bin/xcode-select`，并不是跟随 CLI 工具包一块下载下来的。

### xcrun

回想我们过去在使用一些 CLI 命令的时候，会直接在终端上执行`xcodebuild ...`这样的方式，没有指定具体的 CLI 路径，并且我们执行`which xcodebuild`得到的结果是`/usr/bin/xcodebuild`。那这个命令是怎么执行到我们通过`xcode-select`设置的默认 CLI 路径下呢？那就得提到我们马上要介绍的这个工具了 -- `xcrun`。

我们就以`xcodebuild`举例，我们通过`which xcodebuild`得到的结果是`/usr/bin/xcodebuild`，也就是说我们在执行`xcodebuild`的时候实际上在执行`usr/bin/xcodebuild`，那再让我们看看`/usr/bin/xcodebuild` 下的指令是怎么配合`xcode-select`找到 `/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild`的？

我们先通过`otool -tV /usr/bin/xcodebuild`查看其`textsection`，得到：

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

我们可以发现该命令调用`_xcselect_invoke_xcrun`函数。

然后我们通过`nm /usr/bin/xcodebuild`查看其`name list`

```txt
                 U __NSGetProgname
0000000100008018 d __dyld_private
0000000100000000 T __mh_execute_header
0000000100003f63 t _main
0000000100008010 s _shim_marker
                 U _xcselect_invoke_xcrun
                 U dyld_stub_binder
```

通过`_xcselect_invoke_xcrun`前面的`U`标识我们可以知道该函数是一个外部符号，是另外一个动态库去处理的。

后面我们通过[Swift-Swiftc](https://dongaxis.github.io/2016/04/28/Swift-Swiftc/)可以知道更详细流程。这里只说结论

```txt
libxcselect.dylib
👇🏻
_xcselect_invoke_xcrun
👇🏻
libxcrun.dylib
👇🏻
xcrun_main
```

我们后面最后实际上会调用到`xcrun_main`函数，其内部就会根据`xcode-select`等设置的情况选择合适的 CLI 路径，具体执行的顺序可见[Developer Binaries on OS X, xcode-select and xcrun](https://macops.ca/developer-binaries-on-os-x-xcode-select-and-xcrun/)。

xcrun（`Xcode Command Line Tool Runner`） 是 Xcode 基本的命令行工具，使用它来调用其他 CLI 工具，这时候你应该就知道为啥需要它来调用其他 CLI 工具了。

我们执行`xcrun`命令时实际上也是走的`/usr/bin/xcrun`，其内部也是上面一套流程，准确而言，在这套 CLI 工具包中位于`/usr/bin`路径下的命令都是上面那个流程，也就是说下面几个命令是等价的：

- xcodebuild
- /usr/bin/xcodebuild
- xcrun xcodebuild
- Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild

当然这套工具包有些命令不在`/usr/bin`路径下，我们就需要在命令前加上`xcrun`了，如`swift-demangle`，如果我们直接使用`swift-demangle`就会出现命令找不到的错误，使用`xcrun swift-demangle`这种方式即可。

相关命令：

* xcrun --find clang // 找到指定工具路径
* xcrun --sdk iphoneos --find pngcrush
* xcrun --sdk macosx --show-sdk-path

通过`xcode-select`安装的 CLI 路径位于：`/Library/Developer/CommandLineTools/`。
Xcode 内嵌的 CLI 路径位于：`/Applications/Xcode.app/Contents/Developer/usr/bin`

还有一点需要注意的是，xcrun 并不是只会寻找`xcode-select`设置下的路径，还会寻找 Xcode 另外的一些路径来执行命令，包括

* Developer/usr
* Developer/Platforms
* Developer/ToolChain

例子如下：

xcodebuild：`/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild`
swift-demangle：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swift-demangle`

[深入浅出 Xcode Command Lines Tool - 初探](https://juejin.cn/post/6844904052271087624)
[深入浅出 Xcode 命令列 - libxcselect.dylib](https://juejin.cn/post/6844904052271087623)
[深入浅出 Xcode 命令列 - xcrun](https://juejin.cn/post/6844904052275298317)

关于这两个工具有开源的实现[xcode-tools](https://github.com/b-man/xcode-tools)。

## 符号表相关

### dwarfdump

作用：解析目标文件，存档和`.dSYM` 包中的 `DWARF` 节，并以人类可读的形式打印其内容。
使用场景：Crash 符号化。
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/dwarfdump`

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

更多命令可见[llvm-dwarfdump](https://llvm.liuxfe.com/docs/man/llvm-dwarfdump)

### symbolicatecrash

作用：是一个`perl`脚本，里面整合了逐步解析的操作（可以将命令拷贝出来，直接进行调用）。
路径：`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`

### atos

作用：Crash 符号化
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/atos`

```shell
# 0x0000000100298000为 load address； 0x000000010029e694为 symbol address
# 最后一个i表示显示内联函数
atos -arch arm64  -o iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest -l 0x0000000100298000 0x000000010029e694 -i
```

## 构建相关

### xcodebuild

作用：我们可以使用其对 Xcode 工程进行清理，分析，构建，测试，存档。
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild`

可以通过`man xcodebuild`查看手册。

```shell
# 清理
xcodebuild clean -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE}

# 构建
xcodebuild archive -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH}

## 存档
xcodebuild -exportArchive -archivePath $ARCHIVE_PATH -exportPath ${IPA_PATH} -exportOptionsPlist ${EXPORTOPTIONSPLIST_PATH}
```

- `xctool`：`xctool` 是 `facebook` 推出的用于替换 `xcodebuild` 的更易于测试 ios 和 mac 应用程序的命令行工具，特别适用于 ios app 的持续集成；
- `xcbuild`：`xcbuild` 是一个兼容 `Xcode` 的编译工具，它能使编译更快快速，更友好的编译过程日志，可以运行在多个平台（主要指 OS X 和 Linux）；

### altool

作用：使用其验证 ipa 以及上传 ipa 到 store；
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

作用：
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swiftc`

### clang

作用：
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang`


### sourcekit-lsp

作用：
路径：`/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/sourcekit-lsp`

### 工具相关

### actool

作用：对 项目中 Assets 的文件进行压缩、处理，生成`.car`文件。
路径：`/Applications/Xcode.app/Contents/Developer/usr/bin/actool`；

`actool` 并非一个脚本，而是一个编译完成的二进制文件，所以`compile asset catalog`的过程是一个黑盒。

### swift-demangle

在 Swift 中因为命名空间的原因，需要对类名进行`mangle`，如果需要显示正确名称，自然也需要`demangle`。其实两个方法实现大家可以通过以下链接查看，

`mangle`：[copySwiftV1MangledName函数](https://github.com/opensource-apple/objc4/blob/cd5e62a5597ea7a31dccef089317abb3a661c154/runtime/objc-runtime-new.mm#L859)，

`demangle`：[copySwiftV1DemangledName](https://github.com/opensource-apple/objc4/blob/cd5e62a5597ea7a31dccef089317abb3a661c154/runtime/objc-runtime-new.mm#L813)

当然 Xcode 也为我们特意准备了一个 CLI 工具 --`swift-demangle`来方便我们。

```shell
命令：xcrun swift-demangle  _TtC7iOSTest27PickImageDemoViewController
结果：_TtC7iOSTest27PickImageDemoViewController ---> iOSTest.PickImageDemoViewController

命令：xcrun swift-demangle --compact _TtC7iOSTest27PickImageDemoViewController
结果：iOSTest.PickImageDemoViewController
```

## 反编译

### ar

### nm

命令列出目标文件的符号。

### otool

otool(object file displaying tool) : 针对目标文件的展示工具，用来发现应用中使用到了哪些系统库，调用了其中哪些方法，使用了库中哪些对象及属性。

```shell
-f print the fat headers
-a print the archive header
-h print the mach header
-l print the load commands
-L print shared libraries used
-D print shared library id name
-t print the text section (disassemble with -v)
-p <routine name>  start dissassemble from routine name
-s <segname> <sectname> print contents of section
-d print the data section
-o print the Objective-C segment
-r print the relocation entries
-S print the table of contents of a library
-T print the table of contents of a dynamic shared library
-M print the module table of a dynamic shared library
-R print the reference table of a dynamic shared library
-I print the indirect symbol table
-H print the two-level hints table
-G print the data in code table
-v print verbosely (symbolically) when possible
-V print disassembled operands symbolically
-c print argument strings of a core file
-X print no leading addresses or headers
-m don't use archive(member) syntax
-B force Thumb disassembly (ARM objects only)
-q use llvm's disassembler (the default)
-Q use otool(1)'s disassembler
-mcpu=arg use `arg' as the cpu for disassembly
-j print opcode bytes
-P print the info plist section as strings
-C print linker optimization hints
--version print the version of /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/otool
```

我们

`otool -l iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest | grep __TEXT -C 5`

执行命令后，结果如下，可以看到 dSYM 中代码段起始地址为 `0x0000000100000000`，一般情况下都为这个值。

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

使用链接器 ld 创建库

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

### class-dump

### dumpdecrypted

## 最后

要更加努力呀！

Let's be CoderStar!

- [iOS开发中常用命令工具（xcode-select、lipo、xcrun等](https://www.jianshu.com/p/a4731527ca73)
- [Xcode 相关终端工具使用](https://www.hanleylee.com/usage-of-xcode-terminal-tools.html)
- [Building from the Command Line with Xcode FAQ](https://developer.apple.com/library/archive/technotes/tn2339/_index.html)
