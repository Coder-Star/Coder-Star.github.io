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

### 控制台输出 APP 启动加载各阶段所花费时间

进入 Product > Scheme > Edit Scheme... > Run > Arguments > Environment Variables，在其中增加`DYLD_PRINT_STATISTICS`，值为`1`


### Swift代码编译过长，显示警告信息
Build Settings > Other Swift Flags, 配置如下
```
// 100这个参数表示毫秒，表示当编译时间超过这个时间时，会显示警告
-Xfrontend -warn-long-function-bodies=100
-Xfrontend -warn-long-expression-type-checking=100
```

