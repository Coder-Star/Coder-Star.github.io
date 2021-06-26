---
title: XCode及iOS等配置
date: 2020-11-04 16:49:59
category:
  - iOS
  - Xcode
tags: [iOS, Xcode]
---

## Xcode

### 控制台输出 Core Data 执行过程及结果

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Arguments Passed On Launch，在其中添加 `-com.apple.CoreData.SQLDebug 3`

### 禁止控制台打印 NSLog

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`OS_ACTIVITY_MODE`，值为`Disable`  
**注意此种方式不禁止 swift 的 print()**

### 控制台输出 APP main()函数执行之前的耗时

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`DYLD_PRINT_STATISTICS`，值为`1`
如果获取更详细的信息，可以使用 DYLD_PRINT_STATISTICS_DETAILS

### 控制台输入动态链接库相关信息

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`DYLD_PRINT_ENV`，值为`1`

### Swift 代码编译过长，显示警告信息

Build Settings > Other Swift Flags, 配置如下

```
// 100这个参数表示毫秒，表示当编译时间超过这个时间时，会显示警告
-Xfrontend -warn-long-function-bodies=100
-Xfrontend -warn-long-expression-type-checking=100
```

Xcode 代码片段路径
`~/Library/Developer/Xcode/UserData/CodeSnippets`

文件模板路径
`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/File\ Templates/iOS/Source`

## Build Settings

- Other Linker Flags：控制对OC库文件的链接
  - -ObjC 告诉链接器把库中定义的 Objective-C 类和 Category 都加载进来，如果库中只有 category 没有类，category 也不会加进来
  - -all_load 强制链接器把目标文件都加载进来，即使没有 objc 代码
  - -force_load 跟 all_load 作用类似，但是需要指定要进行全部加载的库文件的路径


- Preprocessor Macros：OC环境的宏定义，一般Debug模式下会配置DEBUG=1，这样才可以使用 #if DEBUG的写法
- Active Compilation Conditions： Swift环境下的宏定义，一般Debug模式下会配置DEBUG，这样才可以使用 #if DEBUG的写法

**通用**

- `${SRCROOT}` 项目根目录下
- `${PODS_ROOT}` 代表的是 pod 目录

- `$(PROJECT_DIR)` 代表的是整个项目
- `$(inherited)` 继承上一级或依赖项的配置。通过 CocoaPods 集成的项目，$(inherited)将会包含 Pods.xcodeproj 中的配置

