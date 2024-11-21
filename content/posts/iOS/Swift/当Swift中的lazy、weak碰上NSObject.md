---
title: 当Swift中的lazy、weak碰上NSObject
date: 2024-11-21T10:24:13
categories: Swift
tags:
  - iOS
  - Swift
share: true
description: ""
cover.image: ""
lastmod: 2024-11-21T10:24:13
---

## 前言

Hi Coder，我是 CoderStar！

今天给大家介绍一个我遇到的小坑。过程大概是这样的，一个复用页面通过不同的入口进入，等返回时，有的正常，有的却出现了 Crash，log 信息如下。

`Cannot form weak reference to instance XXXXXX. It is possible that this object was over-released, or is in the process of deallocation.`

然后看了一下 Crash 时候的调用栈，发现崩溃在 `deinit` 时 `KVO` 释放 `Observer` 的过程中。一段排查之后，新的坑点出炉了。

具体业务代码就不贴了，贴一个能触发 Bug 的 Demo 吧（不包含使用合理性，仅用来测试 Crash）。

## 问题

```swift
protocol MyServiceDelegate: AnyObject {}

class MyService {
    weak var delegate: MyServiceDelegate?
    func stop() {}
}

class MyClass: NSObject, MyServiceDelegate {
    private lazy var service: MyService = {
        let service = MyService()
        service.delegate = self
        return service
    }()

    deinit {
        service.stop()
    }
}

// 测试
func test() {
    let myClass = MyClass()
}
```

大家觉得这段代码会发生什么？可能大家看了上面的介绍心中已经有了预想的答案。是的，跟上面 Crash 报错信息一致。

那我们分析一下，问题出在哪里？其实 Crash 信息相对已经比较明显了，结合到代码就是 self 当前已经在释放中了（`deinit`），不可以被弱引用了（`service.delegate = self`）。

其实出现这个 Crash 有三个条件：

- lazy
- weak
- NSObject

示例代码去除这三个条件中任何一个，Crash 都不会发生。

具体原因：附上 [automatic-reference-counting](https://www.mikeash.com/pyblog/friday-qa-2011-09-30-automatic-reference-counting.html) 中的一段描述。

> ARC's implementation of zeroing weak references requires close coordination between the Objective-C reference counting system and the zeroing weak reference system. This means that any class which overrides retain and release can't be the target of a zeroing weak reference. While this is uncommon, some Cocoa classes, like NSWindow, suffer from this limitation. Fortunately, if you hit one of these cases, you will know it immediately, as your program will crash with a message like this:
>
> objc[2478]: cannot form weak reference to instance (0x10360f000) of class NSWindow

如果大家有兴趣看源码，可以查看 [objc4-680](https://opensource.apple.com/source/objc4/objc4-680/runtime/objc-weak.mm.auto.html) 中的 `weak_register_no_lock` 函数。其抛出了 Crash。

## 解决

解决方式其实可以很简单，先介绍简单的一种：

### 解决方式一

定义一个 `service` 是否初始化的变量，然后在 `deinit` 时根据变量控制是否继续调用 `service.stop()`。就是下面这样：

```swift
class MyClass: NSObject, MyServiceDelegate {
    private var isServiceInit = false
    private lazy var service: MyService = {
        isServiceInit = true

        let service = MyService()
        service.delegate = self
        return service
    }()

    deinit {
        /// 实际业务中，service未初始化使用过，也不会需要在`deinit`时再进行一些处理
        if isServiceInit {
            service.stop()
        }
    }
}
```

### 解决方式二

如果这样的属性比较多，可能上面的方式就需要定义同等数量的控制变量，相对比较繁琐，那我们就简单封装一下。

```swift
final public class Lazy<T> {
    public init(initializer: @escaping () -> T) {
        self.initializer = initializer
    }

    public private(set) var valueIfInitialized: T?

    public var wrapped: T {
        if let value = valueIfInitialized {
            return value
        } else {
            let value = initializer()
            valueIfInitialized = value
            return value
        }
    }

    private let initializer: () -> T
}
```

使用变成了这样。

```swift
class MyClass: NSObject, MyServiceDelegate {
    private lazy var service = Lazy<MyService> {
        let service = MyService()
        service.delegate = self
        return service
    }

    /// 过程中使用service，可使用 service.wrapped

    deinit {
        service.valueIfInitialized?.stop()
    }
}
```

虽然也还是有点丑陋，但是聊胜于无吧。暂时没找到更好的写法了。

## 最后

本次文章比较简短，现在就 Enjoy 吧！

要更加努力呀！

Let's be CoderStar!
