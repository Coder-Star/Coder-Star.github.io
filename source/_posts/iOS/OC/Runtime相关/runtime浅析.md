---
title: runtime 浅析
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - runtime
date: 2021-02-07 14:05:23
---

## 消息转发机制

OC 的方法调用整体分为三个步骤

- 消息发送
- 动态方法解析
- 消息转发

### 消息发送

1. 首先判断消息接受者 receiver 是否为 nil，如果为 nil 直接退出消息发送
2. 如果存在消息接受者 receiverClass，首先在消息接受者 receiverClass 的 cache（哈希表） 中查找方法，如果找到方法，直接调用。如果找不到，往下进行
3. 没有在消息接受者 receiverClass 的 cache 中找到方法，则从 receiverClass 的 class_rw_t 中查找方法 (查找时如果方法有序则`二分查找`，如果无序，则遍历查找)，如果找到方法，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，往下进行
4. 没有在 receiverClass 中找到方法，则通过 superClass 指针找到 superClass，也是现在缓存中查找，如果找到，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，往下进行
5. 没有在消息接受者 superClass 的 cache 中找到方法，则从 superClass 的 class_rw_t 中查找方法，如果找到方法，执行方法，并把该方法缓存到 receiverClass 的 cache 中；如果没有找到，重复 4、5 步骤。如果找不到了 superClass 了，往下进行
6. 如果在最底层的 superClass 也找不到该方法，则要转到动态方法解析

#### 子类的方法缓存是否会存储从父类方法列表查询到的方法？

方法列表缓存存在`metaclass`上，所以每个类都只有一份方法缓存，而不是每一个类的 object 都保存一份。即便是从父类取到的方法，也会存在类本身的方法缓存里。而当用一个父类对象去调用那个方法的时候，也会在父类的 metaclass 里缓存一份。

缓存 Map，key 是方法名`SEL`，Value 为方法实现`IMP`；

#### 为什么不直接使用散列表来存方法列表呢？

>* 散列表是没有顺序的，Objective-C 的方法列表是一个 list，是有顺序的；Objective-C 在查找方法的时候会顺着 list 依次寻找，并且 category 的方法在原始方法 list 的前面，需要先被找到，如果直接用 hash 存方法，方法的顺序就没法保证；
>* list 的方法还保存了除了 selector 和 imp 之外其他很多属性；
>* 散列表是有空槽的，会浪费空间；

#### 方法列表查找时，什么情况下是有序的，什么情况下是无需的；

需要注意的是，当方法列表的结构发生改变的时候，就会触发对列表的排序（注意是方法所在的列表，而不是 rw 中所有的方法列表，即如果是二维数组，只需要重新排列当前方法所在的列表即可）。在以下函数的调用中，会触发对 mlist 的排序：

* methodizeClass
* attachCategories
* addMethod
* addMethods

### 动态方法解析 / 动态方法决议

开发者可以实现以下方法，来动态添加方法实现

```Objective-C
// 实例方法
+ (BOOL)resolveInstanceMethod:(SEL)aSEL {

}

// 类方法
+ (BOOL)resolveClassMethod:(SEL)aSEL {
}
```

动态解析 (实际由开发者手动增加一个方法) 过后，会重新走消息发送的流程，从 receiverClass 的 cache 中查找方法这一步开始执行；

其中这个步骤可以放置一些不常用的方法，起到一个按需加载的作用；

在 CoreData 中，有些属性标记为 @dynamic，这些属性的值背后是通过数据库来更新和获取的，并不需要一个成员变量。所以就会为这些属性的 setter 和 getter 方法实现 resolveInstanceMethod:，返回 YES，并通过数据库来设置或者获取该属性的值。

### 消息转发

#### 快速转发

本质是替换消息接收者

- 调用 forwardingTargetForSelector，返回值不为 nil 时，会调用 objc_msgSend(返回值, SEL)，返回值为 nil，执行下一步。

```objective-c
- (id)forwardingTargetForSelector:(SEL)selector {

}
```

应用场景：继承`NSProxy`达到切断循环引用链的作用；

#### 慢速转发

调用 methodSignatureForSelector, 返回值不为 nil，调用 forwardInvocation: 方法；返回值为 nil 时，调用 doesNotRecognizeSelector: 方法

```objective-c
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
   
}


- (void)forwardInvocation:(NSInvocation *)anInvocation {
    SEL selector = [anInvocation selector];
}
```

### 为什么需要方法签名这个步骤？

IMP 的本质是函数指针，在调用 C 语言函数时，CPU 需要为其分配栈空间，栈空间的栈底若干字节是固定用来存储函数的参数列表以及返回值的，方法签名就是为了告诉 CPU 应如何对栈底的这段内存空间进行布局

### 为什么转发分成快速、慢速转发？

forwardingTargetForSelector 仅支持一个对象的返回，也就是说消息只能被转发给一个对象;forwardInvocation 可以将消息同时转发给任意多个对象。

## 方法交换

方法交换 (method-swizzling)，主要目的是替换两个 Method 的 IMP

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

为什么要先调用`class_addMethod`？

因为如果这个类没有实现原始方法"originalSelector" ，但其父类实现了，那 `class_getInstanceMethod` 会返回父类的方法。这样 `method_exchangeImplementations` 替换的是父类的那个方法。

假设父类有个方法`method`，子类未重写`method`方法，子类的中想要拿来替换的方法为`swizzledMethod`，交换完成之后，在子类实例调用`method`方法可以正常调用到`swizzledMethod`的实现，但是当父类实例调用`method`方法时，因为在父类里面找不到`swizzledMethod`的实现，所以直接就崩溃了。

## runtime 应用场景

- 数据埋点
- Crash 防护
- Category 中为类添加成员变量，利用关联对象。
- JSON 转 model

> 关联对象由 AssociationsManager 管理并在 AssociationsHashMap 中存储。所有对象的关联内容都被放在一个统一的全局容器中.

## runtime 三方库

- Aspects（AOP 必备，'取缔'baseVC，无侵入埋点）
- MJExtension（JSON 转 model，一行代码实现 NSCoding 协议的自动归档和解档）
- JSPatch（动态下发 JS 进行热修复）
- NullSafe（防止因发 unrecognised messages 给 NSNull 导致的崩溃）
- UITableView-FDTemplateLayoutCell（自动计算并缓存 table view 的 cell 高度）
- UINavigationController+FDFullscreenPopGesture（全屏滑动返回）

- [OC源码分析之方法的查找原理](https://juejin.cn/post/6844904118973104135#heading-19)
