---
title: iOS中的库
category:
  - iOS
  - 架构设计
tags:
  - iOS
  - 库
date: 2021-03-21 20:46:46
---

## 概念

**静态库：** 静态库即静态链接库（Windows 下的 .lib，Linux 和 Mac 下的 **.a/.framework**）。之所以叫做静态，是因为静态库在编译的时候会被直接拷贝一份，复制到目标程序里，这段代码在目标程序里就不会再改变了；

静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大

**动态库：** 即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 **.dylib/.tbd/.framework**）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。

动态库的优点是，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作共享库）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行（Linux 下喜闻乐见的 lib not found 错误）。

.a 文件是一个纯二进制文件，是.o 文件的合集,不能直接拿来使用，需要配合头文件、资源文件一起使用。将静态库打包的时候，只能打包代码资源，图片、本地 json 文件和 xib 等资源文件无法打包进去，使用.a 静态库的时候需要三个组成部分：.a 文件+需要暴露的头文件+资源文件；

framework 实际上是一种打包方式，是将库的二进制文件、头文件、资源文件等打包在一起。可以直接拿来使用。

静态库在打包时会直接打包进 app 的二进制文件中，而动态库会放入沙盒中，Frameworks 文件夹下

iOS 8 之前是不允许自己创建动态 Framework，只有系统提供的动态 framework，如 UIKit.Framework，Foundation.Framework 等，iOS 8 后出现了 extension，为了 app 及 extension 之间可以共享代码，允许了动态 framework，但是这种也和系统库的有区别。系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中（App 和 Extension 的 Bundle 是共享的），因此苹果又把这种 Framework 称为 Embedded Framework。

Swift 不可以创建.a 库，如果需要创建静态库，是将 framework 里面的 Mach-O Type 改为 Static Library。
共享可执行文件 iOS 有沙箱机制，不能跨 App 间共享共态库，但 Apple 开放了 App Extension，可以在 App 和 Extension 间共间动态库（这也许是 Apple 开放动态链接库的唯一原因了）。

> Tips: Xcode12 打出的模拟器版 Release 模式下的 framework 会包括 x86_64、arm64 两种架构，但是模拟器的 arm64 与真机的 arm64 并不等同，所以需要对此架构进行擦除或者在打包时进行排除，排除方式为将 Build Setting 的 Excluded Architectures 下的 Any iOS Simulator SDK 中加上 arm64，注意这里一定要加上 Any iOS Simulator SDK 限制，否则也会把真机 build 时的 arm64 架构去掉。

## 静态库、动态库之间的组合

## Module 机制

如果要支持 Module，必须提供一个 module.modulemap 文件，用来声明模块与头文件之间的映射关系

module.modulemap 文件格式

```
framework module moduleName {
	umbrella header "moduleName-umbrella.h" // moduleName-umbrella.h中放置的就是要公开的头文件

	export *
	module * { export * }

  // 子Module
  explicit module SubModuleName {
    header "SubModuleName.h"
    export *
  }

	link framework "Foundation" // 目前这个地方只能link SDK内置的框架
}
```

swift 中使用 `import moduleName`
oc 中使用 `@import LDPMChart;`

## 打包（利用 cocoapods-packager 插件）

### framework

1. 安装 cocoapods-packager `gem "cocoapods-packager"`
2. 打包 `pod package XXX.podspec`

命令

```ruby
pod package XXX.podspec --force // --force表示强制覆盖之前存在的文件
pod package XXX.podspec --library // --library表示打包成a文件，如果不加就是打包成framework文件
```

```ruby
--force
# 强制覆盖之前已经生成过的二进制库

--embedded
# 生成静态.framework

--library
# 生成静态.a

--dynamic
# 生成动态.framework

--bundle-identifier
# 动态.framework是需要签名的，所以只有生成动态库的时候需要这个BundleId

--exclude-deps
# 不包含依赖的pod库的符号表/依赖的pod库不打包进去。生成动态库的时候不能使用这个命令，动态库一定需要包含依赖的符号表。

--configuration
# 表示生成的库是debug还是release，默认是release 。
例如：--configuration=Debug,ONLY_ACTIVE_ARCH=NO

--no-mangle
# 表示 Do not mangle symbols of depedendant Pods，当你的项目依赖包含静态库时，
不加上这句，就会打包失败：
[!] podspec has binary-only depedencies,mangling not possible.

--subspecs
# 如果你的pod库有subspec，那么加上这个命名表示只给某个或几个subspec生成二进制库，
# --subspecs=subspec1,subspec2。生成的库的名字就是你podspec的名字，
# 如果你想生成的库的名字跟subspec的名字一样，那么就需要修改podspec的名字。
# 这个脚本就是批量生成subspec的二进制库，每一个subspec的库名就是podspecName+subspecName。

--spec-sources
# 一些依赖的source，如果你有依赖是来自于私有库的，
# 那就需要加上那个私有库的source，默认是cocoapods的Specs仓库。
# --spec-sources=private,https://github.com/CocoaPods/Specs.git。
```

问题

- Swift 的 Pod 依赖 OC 的 Pod，如果 OC 开始没有模块化，打包不成功。
