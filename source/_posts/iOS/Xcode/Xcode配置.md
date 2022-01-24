---
title: Xcode 配置
date: 2020-11-04 16:49:59
category:
  - iOS
  - Xcode
tags: [iOS, Xcode]
---

## `Environment Variables`

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables

- `OS_ACTIVITY_MODE`值置为`Disable`：禁止控制台打印 NSLog
  **注意此种方式不禁止 swift 的 print()**
- 增加`DYLD_PRINT_STATISTICS`，值为`1`
如果获取更详细的信息，可以使用 DYLD_PRINT_STATISTICS_DETAILS：控制台输出 APP main() 函数执行之前的耗时
- 增加`DYLD_PRINT_ENV`，值为`1`：控制台输入动态链接库相关信息
- `DYLD_PRINT_OPTS`：值为 1

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

Build Setting 里的 Other Linker Flags 就是制定`ld`命令的参数。

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

- `$(PROJECT_DIR)` 代表的是整个项目，一般是`.xcodeproj`文件所在目录
- `${SRCROOT}` 包含定义 Target 的 Project 的路径，一般情况下和`$(PROJECT_DIR)`是等价的
- `${PROJECT_FILE_PATH}`表示 project 的当前路径，相当于`$(PROJECT_DIR)/$(PROJECT_NAME).xcodeproj`，见下图：
- `${PODS_ROOT}` 代表的是 pod 目录，是 cocoapods 通过 UserDefine 自定义的

- `$(inherited)` 继承上一级（Project）或依赖项的配置。通过 CocoaPods 集成的项目，$(inherited) 将会包含 Pods_xxxx.xcconfig 中的配置

在设置路径时还有下面两个选项，表示是否递归寻找子目录

- non-recursive：非递归
- recursive：递归

## Xcode 环境参数

- 关闭 Xcode，打开终端输入`defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES`，然后项目 Build 的时候可以在 Xcode 顶部看到项目编译时间

- `defaults write com.apple.Xcode PBXNumberOfParallelBuildSubtasks 8`  提高 XCode 编译时使用的线程数
