---
title: iOS 中的库
category:
  - iOS
  - 架构设计
tags:
  - iOS
  - 库
date: 2021-03-21 20:46:46
---

## 概念

### 静态库

静态库即静态链接库（Windows 下的 .lib，Linux 和 Mac 下的 **.a/.framework**）。之所以叫做静态，是因为静态库在编译的时候会被直接拷贝一份，复制到目标程序里，这段代码在目标程序里就不会再改变了；

静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大

### 动态库

即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 **.dylib/.tbd/.framework**）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。

动态库的优点是，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作共享库）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行（Linux 下喜闻乐见的 lib not found 错误）。

.a 文件是一个纯二进制文件，是.o 文件的合集, 不能直接拿来使用，需要配合头文件、资源文件一起使用。将静态库打包的时候，只能打包代码资源，图片、本地 json 文件和 xib 等资源文件无法打包进去，使用.a 静态库的时候需要三个组成部分：.a 文件 + 需要暴露的头文件 + 资源文件；

framework 实际上是一种打包方式，是将库的二进制文件、头文件、资源文件等打包在一起。可以直接拿来使用。

静态库在打包时会直接打包进 app 的二进制文件中，而动态库会放入沙盒中，Frameworks 文件夹下

iOS 8 之前是不允许自己创建动态 Framework，只有系统提供的动态 framework，如 UIKit.Framework，Foundation.Framework 等，iOS 8 后出现了 extension，为了 app 及 extension 之间可以共享代码，允许了动态 framework，但是这种也和系统库的有区别。系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中（App 和 Extension 的 Bundle 是共享的），因此苹果又把这种 Framework 称为 Embedded Framework。

Swift 不可以创建.a 库，如果需要创建静态库，是将 framework 里面的 `Mach-O Type` 改为 `Static Library`。
共享可执行文件 iOS 有沙箱机制，不能跨 App 间共享共态库，但 Apple 开放了 App Extension，可以在 App 和 Extension 间共间动态库（这也许是 Apple 开放动态链接库的唯一原因了）。

> Tips: Xcode12 打出的模拟器版 Release 模式下的 framework 会包括 x86_64、arm64 两种架构，但是模拟器的 arm64 与真机的 arm64 并不等同，所以需要对此架构进行擦除或者在打包时进行排除，排除方式为将 Build Setting 的 Excluded Architectures 下的 Any iOS Simulator SDK 中加上 arm64，注意这里一定要加上 Any iOS Simulator SDK 限制，否则也会把真机 build 时的 arm64 架构去掉。

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

## 静态库、动态库之间的组合

## Module 机制

被使用方：`DEFINES_MODULE` 设置为`YES`。
使用方：`Enable Modules`设置为`YES`。

如果要支持 Module，必须提供一个 module.modulemap 文件，用来声明模块与头文件之间的映射关系。这个声明语言叫做 **模块映射语言**

好处：

* 语义上完整描述了一个框架的作用;
* 提高编译时可扩展性，只编译或 include 一次。避免头文件多次引用，只解析一次头文件甚至不需要解析（类似预编译头文件）;
* 减少碎片化，每个 module 只处理一次，环境的变化不会导致不一致;
* 对工具友好，工具（语言编译器）可以获取更多关于 module 的信息，比如链接库，比如语言是 C++ 还是 C 等等;

Module 编译之后的缓存是 `pcm`格式的文件。其为二进制格式。

module.modulemap 文件格式

```c
/// 如果是framwork会是  framework module moduleName
module moduleName {
  // 伞骨文件，通过这种方式避免将所有对外的.h文件都在该文件书写，转到一个统一文件的中去。
  // 如常见的UIKit本身也是有很多.h文件组合而成的
  // #import <UIKit/UIKit.h> -> #import <UIKit/UIViewController.h> #import <UIKit/UILabel.h> ...
  umbrella header "moduleName-umbrella.h"

  
  // 如果是个目录，也会递归查询所有子目录
  // 注意framework module 中无法使用 umbrella的递归导出文件夹内容
  umbrella "folder"

  // * 表示通配符，表示将 umbrella.h 中的所有头文件全部导出
  export *

  // 导出子module
  // module * ：目录下所有的头文件都当作一个子module
  module * { export * }

  // 子Module
  // 显式声明一个子module的名称
  explicit module SubModuleName {
    header "SubModuleName.h"
    export *
  }

  /// C和OC要分开编译, C++可以和C一起编译， C++也可以和OC一起编译。
  /// 当OC 和C 一起需要添加requires objc
  requires objc

  // 目前这个地方只能link SDK内置的框架
  link framework "Foundation"
}
```

swift 中使用 `import moduleName`
oc 中使用 `@import moduleName;`

默认开启 module 之后，在引入头文件时使用 include""、 import<>、@import ；这三种写法，最终都会被转化成 @import。

[module-map](https://clang.llvm.org/docs/Modules.html#module-map-language)

> 建议 modulemap 内声明一个umbrella header，便于快速引用对应的头文件，但必须将所有公开的头文件填充到 umbrella header 文件内。否则将得到一个警告。

```text
<module-includes>
Umbrella header for module 'XXX' does not include header 'absolute path to a public header'
```

问题

- Swift 的 Pod 依赖 OC 的 Pod，如果 OC 开始没有模块化，打包不成功。

动态库使用场景：
- 解决苹果对__TEXT 段大小限制问题；
- 懒加载
- 宿主程序与扩展程序使用同一个库

## 静态链接器ld

[深入 iOS 静态链接器（一）— ld64](https://mp.weixin.qq.com/s/tSj6JVEg7plJQm7aDHLyMw)
