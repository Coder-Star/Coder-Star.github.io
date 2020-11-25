---
title: Swift编译加速相关
category:
  - Swift
tags:
  - Swift
  - 编译优化
date: 2020-11-25 20:37:26
---

## Xcode配置
Build Settings > Other Swift Flags, 配置如下
```
// 100这个参数表示毫秒，表示当编译时间超过这个时间时，会显示警告
-Xfrontend -warn-long-function-bodies=100
-Xfrontend -warn-long-expression-type-checking=100
```

## 代码优化

### 使用 + 拼接可选字符串会极其耗时 
改写成 "\(string1)\(string2)"的形式

### 可选值使用??赋默认值再嵌套其他运算会极其耗时
使用guard let 解包

### 将长计算式代码拆分 最后组合计算
### 与或非和>=,<=,==逻辑运算嵌套Optional会比较耗时
使用guard let + 拆分 进行优化
### 手动增加类型推断会降低编译时间