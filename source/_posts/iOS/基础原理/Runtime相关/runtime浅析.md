---
title: runtime浅析
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - runtime
date: 2021-02-07 14:05:23
---

其中 runtime 的核心包含下面几个部分

- 消息转发
- 方法交换

### 方法调用相关

OC 的方法调用整体分为三个步骤

- 消息发送
- 动态方法解析
- 消息转发

**消息发送**

1. 首先判断消息接受者 receiver 是否为 nil，如果为 nil 直接退出消息发送
2. 如果存在消息接受者 receiverClass，首先在消息接受者 receiverClass 的 cache（哈希表） 中查找方法，如果找到方法，直接调用。如果找不到，往下进行
3. 没有在消息接受者 receiverClass 的 cache 中找到方法，则从 receiverClass 的 class_rw_t 中查找方法(查找时如果方法有序则二分查找，如果无序，则遍历查找，其中类实现了协议，会触发一个排序方法，使方法有序)，如果找到方法，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，往下进行
4. 没有在 receiverClass 中找到方法，则通过 superClass 指针找到 superClass，也是现在缓存中查找，如果找到，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，往下进行
5. 没有在消息接受者 superClass 的 cache 中找到方法，则从 superClass 的 class_rw_t 中查找方法，如果找到方法，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，重复 4、5 步骤。如果找不到了 superClass 了，往下进行 6.如果在最底层的 superClass 也找不到该方法，则要转到动态方法解析

**动态方法解析/动态方法决议**

开发者可以实现以下方法，来动态添加方法实现

```oc
  +resolveInstanceMethod: //实例方法
  +resolveClassMethod: //类方法
```

动态解析(实际由开发者手动增加一个方法)过后，会重新走消息发送的流程，从 receiverClass 的 cache 中查找方法这一步开始执行

**消息转发**

**快速转发**

- 调用 forwardingTargetForSelector，返回值不为 nil 时，会调用 objc_msgSend(返回值, SEL)，返回值为 nil，执行下一步。

**慢速转发**

- 调用 methodSignatureForSelector,返回值不为 nil，调用 forwardInvocation:方法；返回值为 nil 时，调用 doesNotRecognizeSelector:方法
- 开发者可以在 forwardInvocation:方法中自定义任何逻辑
- 以上方法都有对象方法、类方法 2 个版本（前面可以是加号+，也可以是减号-）

> forwardingTargetForSelector 仅支持一个对象的返回，也就是说消息只能被转发给一个对象;forwardInvocation 可以将消息同时转发给任意多个对象。

### 方法交换

方法交换(method-swizzling)，主要目的是替换两个 Method 的 IMP

```swift
// 获取SEL值
// 获取方式包括 1、#selector 2、NSSelectorFromString()
let originalSelector = #selector("老方法引用")
let swizzledSelector = #selector("新方法引用")

// 原有方法，通过方法编号找到方法
let originalMethod = class_getInstanceMethod(self, originalSelector)
let swizzledMethod = class_getInstanceMethod(self, swizzledSelector)
// 先尝试給原有SEL添加新IMP，这里是为了避免原有SEL没有实现IMP的情况
let didAddMethod = class_addMethod(self, originalSelector, method_getImplementation(swizzledMethod!), method_getTypeEncoding(swizzledMethod!))

if didAddMethod {
    // 添加成功，说明原SEL没有实现IMP，此时新SEL还是对应新IMP，添加成功后将新的SEL的IMP替换成老IMP
    // 一个IMP可以对应多个SEL
    class_replaceMethod(self, swizzledSelector, method_getImplementation(originalMethod!), method_getTypeEncoding(originalMethod!))
} else {
    // 添加失败，说明原SEL已经实现了IMP，直接将两个SEL的IMP实现交换即可
    method_exchangeImplementations(originalMethod!, swizzledMethod!)
}
```

## runtime 应用场景

- 数据埋点
- Crash 防护
- Category 中为类添加成员变量，利用关联对象。
- JSON 转 model

> 关联对象由AssociationsManager管理并在AssociationsHashMap中存储。所有对象的关联内容都被放在一个统一的全局容器中.

## runtime 三方库

- Aspects（AOP 必备，“取缔” baseVC，无侵入埋点）
- MJExtension（JSON 转 model，一行代码实现 NSCoding 协议的自动归档和解档）
- JSPatch（动态下发 JS 进行热修复）
- NullSafe（防止因发 unrecognised messages 给 NSNull 导致的崩溃）
- UITableView-FDTemplateLayoutCell（自动计算并缓存 table view 的 cell 高度）
- UINavigationController+FDFullscreenPopGesture（全屏滑动返回）
