---
title: Swift 版本更新记录
category:
  - iOS
  - Swift
tags:
  - Swift
  - 版本更新
date: 2021-03-06 10:27:20
---

[Swift 官方更新日志](https://github.com/apple/swift/blob/main/CHANGELOG.md)

## Swift5

### 一、非语法层面

#### 1、ABI 稳定。

iOS12.2 内置 Swift 核心库。当我们开发项目时，设置适配最低版本在 12.2 及其以上时，打包便不会再打入 swift 标准库，会很大程度上减小安装包的尺寸。当最低版本低于 12.2 版本后，就会打入 swift runtime，但是经过 app store 分发后，因为 App Thinning 机制，不同系统获取到的安装包尺寸会不同，即 12.2 及以上系统的尺寸会小一点，因为去掉了包中的 swift runtime。
因为内置了 Swift runtime 后，带来了以下好处：
- **减小包体积**
- **节省内存**
- **减小启动耗时**
- **不强制依赖编译器**：在 ABI 稳定之前，两个二进制组件必须是同一版本编译器编译的才能共存，在 ABI 稳定之后，不会再存在这个问题，两个不同版本编译器编译出来的二进制组件可以**共存**。
-

### 二、语法层面、

- 枚举遍历时，即使所有选项都覆盖了还是会提示警告让实现 @unknown default，这样主要还是为了 ABI 稳定做考虑，必须后后续系统枚举增加选项。

#### 1、原始字符串。

使用#"...."#来创建原始字符串，而不是采用转义字符的方式。如果字符串本身有#，则外部使用##来创建

## Swift5.1

### 一、非语法层面

### 1、模块稳定。

**模块稳定：** 定义了一个新的基于文本的模块接口文件 (`.swiftinterface文件`)，该文件说明了二进制框架的 API。Module 稳定使我们创建的 Swift framework 能够兼容未来的 Swift 版本。比如说你当前的 framework 是 Swift5.1 编译的，那当 Swift 6 编译器下也可以正常使用 framework 提供的 API。组件能在更高版本的编译器环境下被引用。

**Library Evolution：** 二进制组件在发生改变时，只要 API 向前兼容，引用它的宿主无须重新编译。

`BUILD_LIBRARY_FOR_DISTRIBUTION`，XCode 中这个 Build Setting 属性

- 编译出来的二进制文件更小，通常是 10%。
  Swift 和 ObjC 的桥接更快了
  - NSString 和 String 桥接有 15x 的提升
  - NSDictionary 和 Dictionary 有 1.6x 的提升
  - 大小比较小的 String 的优化
- 新的改进的诊断体系。
- SwiftUI。

### 二、语法层面

- 单行函数隐式返回，不需要 return；

- 不透明返回类型 some 关键字；
- 属性包装器 @propertyWrapper；

Tips：不透明返回类型、属性包装器就是为 SwiftUI 量身定做的。

## Swift5.2

### 一、非语法层面

- 调用类型为函数。
- 面向对象的函数式编程。
- 将密钥路径作为函数传递。

### 二、语法层面

## Swift5.3

### 一、非语法层面

### 二、语法层面

- 多模式 catch 语句
- 多尾随闭包
-

## Swift5.4

### 一、非语法层面

### 二、语法层面

- 函数多个可变参数。
- 属性包装器可以用局部变量。
- 改进了隐式成员语法。
- 支持同名函数，参数类型不同即可。

## Swift 5.5

- async、await 出现
- 自动执行 Double 和 CGFloat 值之间的转换

[Swift 5.5 新特性](http://www.cocoachina.com/articles/904608)

## 文档

- [Swift 文档修订历史](https://www.runoob.com/manual/gitbook/swift5/source/_book/chapter1/04_revision_history.html)
