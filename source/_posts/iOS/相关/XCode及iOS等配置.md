---
title: XCode 及 iOS 等配置
date: 2020-11-04 16:49:59
category:
  - iOS
  - 相关
tags: [iOS, Xcode]
---

## `Environment Variables`

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables

- `OS_ACTIVITY_MODE`值置为`Disable`：禁止控制台打印 NSLog
  **注意此种方式不禁止 swift 的 print()**
- 增加`DYLD_PRINT_STATISTICS`，值为`1`
如果获取更详细的信息，可以使用 DYLD_PRINT_STATISTICS_DETAILS：控制台输出 APP main() 函数执行之前的耗时
- 增加`DYLD_PRINT_ENV`，值为`1`：控制台输入动态链接库相关信息
- `DYLD_PRINT_OPTS`：值为1

## `Arguments Passed On Launch`

- 添加 `-com.apple.CoreData.SQLDebug 3`：控制台输出 Core Data 执行过程及结果

### Swift 代码编译时间相关

Build Settings > Other Swift Flags, 配置如下

```
// 100这个参数表示毫秒，表示当编译时间超过这个时间时，会显示警告
// 方法体时长超过 100 ms
-Xfrontend -warn-long-function-bodies=100
// 类型推断时长超过 100 ms
-Xfrontend -warn-long-expression-type-checking=100

// 输出每个函数的编译时长
-Xfrontend -debug-time-function-bodies
```

Xcode 代码片段路径
`~/Library/Developer/Xcode/UserData/CodeSnippets`

文件模板路径
`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/File\ Templates/iOS/Source`

## Build Settings

Build Setting里的Other Linker Flags就是制定`ld`命令的参数。

链接器（Linker）是一个程序，它可以将一个或多个由编译器或汇编器生成的目标文件外加库链接为一个可执行文件。

- Other Linker Flags：控制对 OC 库文件的链接
  - -ObjC 告诉链接器把库中定义的 Objective-C 类和 Category 都加载进来。如果库中只有 category 没有类，category 也不会加进来，但是不会将 C++、C等类加载进来；（为了修复这个问题，通常的解决办法是在这个.m 文件里写一个空的 Class 来占位）

  ```objective-c
  // Use dummy class for category in static library.
  #ifndef DUMMY_CLASS
  #define DUMMY_CLASS(name) \
      @interface DUMMY_CLASS_ ## name : NSObject @end \
      @implementation DUMMY_CLASS_ ## name @end
  #endif
  //使用示例:
  //UIColor+YYAdd.m
  #import "UIColor+YYAdd.h"
  DUMMY_CLASS(UIColor+YYAdd)

  @implementation UIColor(YYAdd)
  ...
  @end
  ```

  - -all_load 强制链接器把目标文件都加载进来，即使没有 objc 代码
  - -force_load 跟 all_load 作用类似，但是需要指定要进行全部加载的库文件的路径

- Preprocessor Macros(GCC_PREPROCESSOR_DEFINITIONS)：OC 环境的宏定义，一般 Debug 模式下会配置 DEBUG=1，这样才可以使用 #if DEBUG 的写法
- Other Swift Flags(OTHER_SWIFT_FLAGS)：Xcode8 之前 Swift 环境下的条件编译变量，加上`-D XXX`
- Active Compilation Conditions(SWIFT_ACTIVE_COMPILATION_CONDITIONS)：Xcode8 之后 Swift 环境下的条件编译变量，一般 Debug 模式下会配置 DEBUG，这样才可以使用 #if DEBUG 的写法，不需要加 D；

Swift 内置了几种平台和架构的组合，如下所示，当然我们也可通过上述方式进行自定义配置

```swift
// 操作系统: macOS\iOS\tvOS\watchOS\Linux\Android\Windows\FreeBSD
#if os(macOS) || os(iOS)

// CPU架构:i386\x86_64\arm\arm64
#elseif arch(x86_64) || arch(arm64)

// swift版本
#elseif swift(<5) && swift(>=3)

// 模拟器
#elseif targetEnvironment(simulator)

// 可以导入某模块
#elseif canImport(Foundation)

#elseif DEBUG

#endif

```

**通用**

- `${SRCROOT}` 项目根目录下
- `${PODS_ROOT}` 代表的是 pod 目录

- `$(PROJECT_DIR)` 代表的是整个项目
- `$(inherited)` 继承上一级或依赖项的配置。通过 CocoaPods 集成的项目，$(inherited) 将会包含 Pods.xcodeproj 中的配置

## workspace, project, target, build configuration, scheme

想 build 出一个 product，需要知道有哪些文件需要 build，build 的时候需要哪些构建参数。

target 指定了需要哪些文件，build configuration 指定了使用哪些构建参数。所以我们 build 的时候就需要一个特定的 target 和一个特定的 build configuration，这时候 scheme 就起作用了，scheme 可以理解为工程编译运行时的配置文件, 它可以指定 build 的时候用哪个 target 和 build configuration，project 里可以有多个 target 和多个 build configuration，同时 workspace 里可以有多个 project，这些 project 的编译输出文件同在一个编译输出目录下，即整个 workspace 维度的。

所以如果需要编译不同的文件，那么需要不同的 target；如果编译的文件都相同，只是配置文件不同，如 plist、entitlements 文件等，那么只需不同的 build configuration 即可。如果既要编译不同的文件，又要不同的配置文件及编译参数，那么需要 target 和 build configuration 混合使用。所有 build 设置相关的都可以在 build configuration 中单独设置。
