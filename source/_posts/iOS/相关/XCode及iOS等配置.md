---
title: XCode及iOS等配置
date: 2020-11-04 16:49:59
category:
  - iOS
  - Xcode
tags: [iOS, Xcode]
---

## Xcode

```
Product > Scheme > Edit Scheme... > Run > Arguments > Arguments Passed On Launch
Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables
```

### 控制台输出 Core Data 执行过程及结果

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Arguments Passed On Launch，在其中添加 `-com.apple.CoreData.SQLDebug 3`

### 禁止控制台打印 NSLog

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`OS_ACTIVITY_MODE`，值为`Disable`  
**注意此种方式不禁止 swift 的 print()**

### 控制台输出 APP main()函数执行之前的耗时

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`DYLD_PRINT_STATISTICS`，值为`1`
如果获取更详细的信息，可以使用DYLD_PRINT_STATISTICS_DETAILS


  
APP启动分为三个阶段
* main函数执行前
  main()函数执行之前要做的事情，会包括
  * 加载可执行文件（优化方式：删减无用静态常量，删除无用代码，尽量不使用C++虚函数）
  * 加载动态链接库（可以进行优化，最多支持6个非系统动态库合成一个）
  * 初始化Objective-C Runtime
  * 执行其他初始化代码（load方法），优化方式是将load方法里面执行的逻辑放入到首屏渲染后执行，放入+initialize中
* main函数执行后  
  main()函数执行之后到AppDelegate 类中的applicationDidFinishLaunching:withOptions:方法执行结束前这段时间。这是APP启动优化的重点
  * 尽量使用纯代码编写，减少xib的使用
  * 启动阶段的网络请求，是否都放到异步请求
  * 一些耗时的操作是否可以放到后面去执行，或异步执行等
* 首屏渲染完成后

### 控制台输入动态链接库相关信息
进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`DYLD_PRINT_ENV`，值为`1`

### Swift代码编译过长，显示警告信息
Build Settings > Other Swift Flags, 配置如下
```
// 100这个参数表示毫秒，表示当编译时间超过这个时间时，会显示警告
-Xfrontend -warn-long-function-bodies=100
-Xfrontend -warn-long-expression-type-checking=100
```


Xcode代码片段路径
`~/Library/Developer/Xcode/UserData/CodeSnippets`

文件模板路径
`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/Templates/File\ Templates/iOS/Source`
