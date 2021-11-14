---
title: Swift 中的值类型与引用类型
category:
  - iOS
  - Swift
tags:
  - iOS
  - Swift
date: 2021-11-14 11:01:17
---

## 前言

Hi Coder，我是 CoderStar！

在 Swift 开发过程中，你很可能至少问过自己一次`struct`与`class`之间的区别，即使你自己没问过，你的面试官应该也问过。对其问题的答案中，可能最大的区别就是一个是值类型，而另一个是引用类型，今天我们就来具体聊聊这个区别。

那在介绍值类型与引用类型之前，我们还是先来回顾一下`struct`与`class`之间的区别这个问题。

## `class` & `struct`

在 Swift 中，其实`class` 与 `struct`之间的核心区别不是很多，有很多区别是值类型与引用类型这个区别带来的天然的区别。

- `class` 可以继承，`struct` 不能继承；受此影响的区别有：
  > `struct`可以利用`protocol`来实现类似继承的效果。
  - `struct`中方法的派发方式全都是直接派发，而`class`中根据实际情况有多种派发方式，详情可看[Swift派发机制](../Swift派发机制)；
- `class` 需要自己定义构造函数，`struct` 默认生成；
  > `struct` 默认生成的构造函数必须包括所有成员参数，只有当所有参数都为可选型时，可直接不用传入参数直接简单构造，`class` 中的属性必须都有默认值，否则编译错误, 可以通过声明时赋值或者构造函数赋值两种方式给属性设置默认值。
- `class` 是引用类型，`struct` 是值类型；受此影响的区别有：
  - `struct` 改变其属性受修饰符 let 影响，不可改变，`class` 不受影响；
  - `struct` 方法中需要修改自身属性时 (非 `init` 方法)，方法需要前缀修饰符 `mutating`；
  - `struct` 因为是值类型的原因，所以自动线程安全，而且也不存在循环引用导致内存泄漏的风险；
  - 派发方式上的差异；
  - ...
  - 更多看下一章节
- ...

在 Swift 中，很多基础类型，如`String`，`Int`等等，都是使用`Struct`来定义。对于如何选择两者这个问题上，Apple 在一些官方文档中也给出了它们之间的区别以及官方建议。

- [choosing_between_structures_and_classes](https://developer.apple.com/cn/documentation/swift/choosing_between_structures_and_classes/)
- [Value and Reference Types](https://developer.apple.com/swift/blog/?id=10)
- [ClassesAndStructures](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)

> 来自《choosing_between_structures_and_classes》
>
> 在向 app 中添加新数据类型时，您不妨考虑以下建议来帮助自己做出合理的选择。
> * 默认使用结构。
> * 在需要 Objective-C 互操作性时使用类。
> * 在需要控制建模数据的恒等性时使用类。
> * 将结构与协议搭配，通过共享实现来采用行为。

## 值类型 & 引用类型

那值类型与引用类型之间的区别有哪些呢？

- 存储方式及位置：值类型存储在栈上，引用类型存储在堆上；
- 内存：值类型没有引用计数，也不会存在循环引用以及内存泄漏等问题；
- 线程安全：值类型天然线程安全，而引用类型需要开发者通过加锁等方式来保证；
- 拷贝方式：值类型拷贝的是内容，而引用类型拷贝的是指针，从一定意义上讲就是所谓的深拷贝及浅拷贝；

> 在 Swift 中，值类型除了`struct`之外还有`enum`，引用类型除了`class`之外还有`closure`/`func`。

### 存储方式及位置

上文说的'堆'和'栈'是程序运行中的不同内存空间，在一个进程中。

关于堆、栈存储原理，美团的这篇[【基本功】深入剖析Swift性能优化](https://tech.meituan.com/2018/11/01/swift-compile-performance-optimization.html)给出了细节说明，这里就不再赘述了，大概说下结论。

值类型默认存储在栈区，栈区内存是连续的，通过出栈入栈进行分配和销毁，速度很快，而且每个线程都有自己的栈空间，所以不需要考虑线程安全问题；访问存储内容时一次就可以拿到值。

引用类型，只在栈区存储了对象的指针，指针指向的对象的内存是分配在堆区的。堆在分配和释放时都要调用函数（MALLOC,FREE)，这些都会花费一些时间，而且因为堆空间被所有线程共享，所以在使用时要考虑线程安全。访问存储内容时，需要两次访问内存，第一次得取得指针，第二次才是真正的数据。
> 其中对64位系统上，iOS加入了`Tagged Pointer`优化方式，即直接在指针中存储值，比如`NSNumber`以及`NSString`结构。

从描述来看，我们得到的最重要的结论是使用值类型比使用引用类型更快，具体技术指标可查看[why-choose-struct-over-class](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)，还有一个测试项目[StructVsClassPerformance](https://github.com/knguyen2708/StructVsClassPerformance)。

通过上面的描述，我们可以有一个问题，就是所有的`class`都存储在堆上，所有的`struct`都存储在栈上吗？这也是本篇文章的重点。

对于纯粹的`class`或`struct`，这显然是毫无疑问的，那

虽然我个人不知道苹果是否会把一些简单的类优化到在栈上存储，但是结构体一般情况下是会被优化到栈上面存储的，除非这个结构体的生命周期被延长，函数结束还没释放 (比如被类持有或者被函数闭合)，不过如果这个结构体占用空间太大，则不会被优化到栈上面。

Swift 的结构体一般被存储在栈上，而非堆上。不过对于可变结构体，这其实是一种优化：
默认情况下结构体是存储在堆上的，但是在绝大多数时候，这个优化会生效，并将结构体存储到栈上。
编译器这么做是因为那些被逃逸闭包捕获的变量需要在栈帧之外依然存在。
编译器侦测到结构体变量被一个函数闭合的时候，优化将不再生效，此时这个结构体将存储在堆上。

### 拷贝方式

引用类型，在拷贝时，实际上拷贝的只是栈区存储的对象的指针；值类型拷贝的是实际的值。

对于值类型拷贝，Swift 有一套 **写时复制 COW（Copy-On-Write）** 优化机制，即只有赋值后值类型发生改变的时候才会进行真正的拷贝，当没有改变时，两者共享同一个内存地址。

Apple在 [OptimizationTips](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#advice-use-copy-on-write-semantics-for-large-values) 中，给出了一个示例。

> 该文中有一些 Apple 给出的优化方式，比如减少动态派发的方式等等，建议 enjoy。

```swift
final class Ref<T> {
  var val: T
  init(_ v: T) {val = v}
}

struct Box<T> {
    var ref: Ref<T>
    init(_ x: T) { ref = Ref(x) }

    var value: T {
        get { return ref.val }
        set {
          /// 判断当前对象是否只有一个引用，如果不是才进行拷贝
          if !isKnownUniquelyReferenced(&ref) {
            ref = Ref(newValue)
            return
          }
          ref.val = newValue
        }
    }
}
```

Swift 标准库中，String、Array、Dictionary、Set 等默认实现了`COW`，对于自定义对象，我们需要自己实现。

## 最后

要更加努力呀！

Let's be CoderStar!

- [iOS Swift5：浅析结构体（struct）与类（class）](https://juejin.cn/post/6933231394227240967#heading-7)
- [Why Choose Struct Over Class?](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)
- [Memory Management and Performance of Value Types](https://swiftrocks.com/memory-management-and-performance-of-value-types)
- [Value Types and Reference Types in Swift](https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/)
- [why-choose-struct-over-class](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)
- [reference-vs-value-types-in-swift](https://www.raywenderlich.com/9481-reference-vs-value-types-in-swift)
