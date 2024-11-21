---
title: 从SIL角度看Swift中的值类型与引用类型
date: 2024-11-21T09:56:02
categories: Swift
tags:
  - iOS
  - Swift
share: true
description: ""
cover.image: ""
lastmod: 2024-11-21T09:56:02
---

## 前言

Hi Coder，我是 CoderStar！

在 Swift 开发过程中，你很可能至少问过自己一次 `struct` 与 `class` 之间的区别，即使你自己没问过，你的面试官应该也问过。对这个问题的答案中，可能最大的区别就是一个是值类型，而另一个是引用类型，今天我们就来具体聊聊这个区别。

那在介绍值类型与引用类型之前，我们还是先来回顾一下 `struct` 与 `class` 之间的区别这个问题。

## `class` & `struct`

在 Swift 中，其实 `class` 与 `struct` 之间的核心区别不是很多，有很多区别是值类型与引用类型这个区别隐形带来的天然的区别。

- `class` 可以继承，`struct` 不能继承（当然 `struct` 可以利用 `protocol` 来实现类似继承的效果。）；受此影响的区别有：
  - `struct` 中方法的派发方式全都是直接派发，而 `class` 中根据实际情况有多种派发方式，详情可看 [Swift派发机制](../Swift派发机制)；
- `class` 需要自己定义构造函数，`struct` 默认生成；

  > `struct` 默认生成的构造函数必须包括所有成员参数，只有当所有参数都为可选型时，可直接不用传入参数直接简单构造，`class` 中的属性必须都有默认值，否则编译错误, 可以通过声明时赋值或者构造函数赋值两种方式给属性设置默认值。

- `class` 是引用类型，`struct` 是值类型；受此影响的区别有：
  - `struct` 改变其属性受修饰符 let 影响，不可改变，`class` 不受影响；
  - `struct` 方法中需要修改自身属性时 (非 `init` 方法)，方法需要前缀修饰符 `mutating`；
  - `struct` 因为是值类型的原因，所以自动线程安全，而且也不存在循环引用导致内存泄漏的风险；
  - ...
  - 更多看下一章节
- ...

在 Swift 中，很多基础类型，如 `String`，`Int` 等等，都是使用 `Struct` 来定义。对于如何选择两者这个问题上，Apple 在一些官方文档中也给出了它们之间的区别以及官方建议。

- [choosing_between_structures_and_classes](https://developer.apple.com/cn/documentation/swift/choosing_between_structures_and_classes/)
- [Value and Reference Types](https://developer.apple.com/swift/blog/?id=10)
- [ClassesAndStructures](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)

> 来自《choosing_between_structures_and_classes》
>
> 在向 app 中添加新数据类型时，您不妨考虑以下建议来帮助自己做出合理的选择。
> - 默认使用结构。
> - 在需要 Objective-C 互操作性时使用类。
> - 在需要控制建模数据的恒等性时使用类。
> - 将结构与协议搭配，通过共享实现来采用行为。

## 值类型 & 引用类型

那在 Swift 中，值类型与引用类型之间的区别有哪些呢？

- 存储方式及位置：大部分值类型存储在栈上，大部分引用类型存储在堆上；
- 内存：值类型没有引用计数，也不会存在循环引用以及内存泄漏等问题；
- 线程安全：值类型天然线程安全，而引用类型需要开发者通过加锁等方式来保证；
- 拷贝方式：值类型拷贝的是内容，而引用类型拷贝的是指针，从一定意义上讲就是所谓的深拷贝及浅拷贝；

> 在 Swift 中，值类型除了 `struct` 之外还有 `enum`、`tuple`，引用类型除了 `class` 之外还有 `closure`/`func`。

### 存储方式及位置

上文说的 ' 堆 ' 和 ' 栈 ' 是程序运行中的不同内存空间。

关于堆、栈存储原理，美团的这篇 [【基本功】深入剖析Swift性能优化](https://tech.meituan.com/2018/11/01/swift-compile-performance-optimization.html) 给出了细节说明，这里就不再赘述了，大概说下结论。

值类型默认存储在栈区，栈区内存是连续的，通过出栈入栈进行分配和销毁，速度很快，而且每个线程都有自己的栈空间，所以不需要考虑线程安全问题；访问存储内容时一次就可以拿到值。

引用类型，只在栈区存储了对象的指针，指针指向的对象的内存是分配在堆区的。堆在分配和释放时都要调用函数（MALLOC,FREE) 动态申请 / 释放内存，这些都会花费一些时间，而且因为堆空间被所有线程共享，所以在使用时要考虑线程安全。访问存储内容时，需要两次访问内存，第一次得取得指针，第二次才是真正的数据。

> 其中在 64 位系统上，iOS 加入了 `Tagged Pointer` 优化方式，即直接在指针中存储值，比如 `NSNumber`、`NSString` 以及 `NSDate` 结构，也就是所谓的

从描述来看，我们得到的最重要的结论是使用值类型比使用引用类型更快，具体技术指标可查看 [why-choose-struct-over-class](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)，还有一个测试项目 [StructVsClassPerformance](https://github.com/knguyen2708/StructVsClassPerformance)。

通过上面的描述，我们可以有一个问题，就是所有的 `class` 都存储在堆上，所有的 `struct` 都存储在栈上吗？这也是本篇文章的重点。其实对于绝大多数情况而言，这种说法都是没问题的，但是总会有些特殊情况。

在阅读下文之前，我们先看一下，如何判断对象是在栈分配还是在堆分配。对于这个问题我们可以在 [SIL.rst](https://github.com/apple/swift/blob/main/docs/SIL.rst) 中找到答案。Swift 编译生成的 SIL 文件中，会包含派发指令，与内存分配相关的命令中，有 [alloc-stack](https://github.com/apple/swift/blob/main/docs/SIL.rst#alloc-stack) 和 [alloc-box](https://github.com/apple/swift/blob/main/docs/SIL.rst#alloc-box) 命令可以来帮助我们解决这个问题，简单来说前者就是来栈上分类内存的指令，而后者就是在堆上分配任务的指令。

**栈上的引用类型**

栈上的分配和释放成本远低于堆上的分配和释放，因此有时编译器可能会提升引用类型也存储在堆栈上，这个过程实际发生在 SIL 优化阶段，官方术语叫做 `Memory promotion`。关于这一说法，我们可以在 [Guaranteed Optimization and Diagnostic Passes](https://github.com/apple/swift/blob/main/docs/SIL.rst#guaranteed-optimization-and-diagnostic-passes) 找到支撑。

> Memory promotion is implemented as two optimization phases, the first of which performs capture analysis to promote alloc_box instructions to alloc_stack, and the second of which promotes non-address-exposed alloc_stack instructions to SSA registers.

大致意思是就是 SIL 阶段会尽量进行内存提升，将原来堆内存提升为栈内存，栈内存提升为 SSA 寄存器内存。

具体优化部分代码我们可以在 [AllocBoxToStack.cpp](https://github.com/apple/swift/blob/main/lib/SILOptimizer/Transforms/AllocBoxToStack.cpp) 中看到。

**堆上的值类型**

在《Swift 进阶》书中有过这么一段话，（在 3.0 版本中出现，5.0 版本删除掉了）：

> Swift 的结构体一般被存储在栈上，而非堆上。不过这其实是一种优化: 默认情况下结构体是存储在堆上的，但是在绝大多数时候，这个优化会生效，并将结构体存储到栈上。当结构体变量被一个函数闭合的时候，优化将不再生效，此时这个结构体将存储在堆上。

看到这句话有些同学会有点摸不着头脑，为什么默认情况结构体会存在堆上，然后经过优化时候才存储到栈上。下面我们来看 `struct` 编译生成的相关 SIL 文件。

```swift
struct Test {}
```

这是一个非常简单的 `struct` 结构体，简单到连属性都没了，我们使用 `swiftc` 命令生成 SIL 文件，命令如下：

`swiftc Test.swift -emit-silgen | xcrun swift-demangle > TestSILGen.sil`

其含义就是生成 `Raw SIL`，也就是原生 SIL 文件，没有经过任何优化和处理。更多命令可以看之前输出的一篇文章 [iOS编译简析](../../进阶/iOS编译简析)。

生成的 SIL 文件内容如下：

```swift
sil_stage raw

import Builtin
import Swift
import SwiftShims

struct Test {
  init()
}

// main
sil [ossa] @main : $@convention(c) (Int32, UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>) -> Int32 {
bb0(%0 : $Int32, %1 : $UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>):
  %2 = integer_literal $Builtin.Int32, 0          // user: %3
  %3 = struct $Int32 (%2 : $Builtin.Int32)        // user: %4
  return %3 : $Int32                              // id: %4
} // end sil function 'main'

// Test.init()
sil hidden [ossa] @$s4main4TestVACycfC : $@convention(method) (@thin Test.Type) -> Test {
// %0 "$metatype"
bb0(%0 : $@thin Test.Type):
  %1 = alloc_box ${ var Test }, let, name "self"  // user: %2
  %2 = mark_uninitialized [rootself] %1 : ${ var Test } // users: %5, %3
  %3 = project_box %2 : ${ var Test }, 0          // user: %4
  %4 = load [trivial] %3 : $*Test                 // user: %6
  destroy_value %2 : ${ var Test }                // id: %5
  return %4 : $Test                               // id: %6
} // end sil function '$s4main4TestVACycfC'
```

我们可以很明显的看到 `alloc_box` 字眼。

然后我们再使用生成优化后 SIL 文件的命令，如下：

`swiftc Test.swift -emit-sil | xcrun swift-demangle > TestSIL.sil`

```swift
sil_stage canonical

import Builtin
import Swift
import SwiftShims

struct Test {
  init()
}

// main
sil @main : $@convention(c) (Int32, UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>) -> Int32 {
bb0(%0 : $Int32, %1 : $UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>):
  %2 = integer_literal $Builtin.Int32, 0          // user: %3
  %3 = struct $Int32 (%2 : $Builtin.Int32)        // user: %4
  return %3 : $Int32                              // id: %4
} // end sil function 'main'

// Test.init()
sil hidden @$s4main4TestVACycfC : $@convention(method) (@thin Test.Type) -> Test {
// %0 "$metatype"
bb0(%0 : $@thin Test.Type):
  %1 = alloc_stack $Test, let, name "self"        // user: %3
  %2 = struct $Test ()                            // user: %4
  dealloc_stack %1 : $*Test                       // id: %3
  return %2 : $Test                               // id: %4
} // end sil function '$s4main4TestVACycfC'
```

我们很明显看到 `alloc_stack` 字眼。

相信大家已经明白发生了什么，`struct` 在生成原始的 SIL 文件中实际上会使用堆指令，然后在 SIL 优化阶段会根据代码上下文环境判断是否可以优化到栈上继而对指令进行修改。那大部分情况下是都可以优化到栈上的。这个过程就有上述 `AllocBoxToStack.cpp` 文件的参与。

当然，那肯定还有另外的少部分情况。比如说：

```swift
func uniqueIntegerProvider() -> () -> Int {
    // i是Int类型，本质也是一个结构体
    var i = 0
    return {
        i+=1
        return i
    }
}
```

> 如果这个 i 不修改，情况又会不同，具体可看 [CapturePromotion.cpp](https://github.com/apple/swift/blob/31b167468793ec5b25a6c4e0769e2883d6125049/lib/SILOptimizer/IPO/CapturePromotion.cpp) 开头注释。

对此代码生成的两份 SIL 文件，核心部分如下：

**优化前：**

```swift
// uniqueIntegerProvider()
sil hidden [ossa] @main.uniqueIntegerProvider() -> () -> Swift.Int : $@convention(thin) () -> @owned @callee_guaranteed () -> Int {
bb0:
  %0 = alloc_box ${ var Int }, var, name "i"      // users: %11, %8, %1
  %1 = project_box %0 : ${ var Int }, 0           // users: %9, %6
  %2 = integer_literal $Builtin.IntLiteral, 0     // user: %5
  %3 = metatype $@thin Int.Type                   // user: %5
  // function_ref Int.init(_builtinIntegerLiteral:)
  %4 = function_ref @Swift.Int.init(_builtinIntegerLiteral: Builtin.IntLiteral) -> Swift.Int : $@convention(method) (Builtin.IntLiteral, @thin Int.Type) -> Int // user: %5
  %5 = apply %4(%2, %3) : $@convention(method) (Builtin.IntLiteral, @thin Int.Type) -> Int // user: %6
  store %5 to [trivial] %1 : $*Int                // id: %6
  // function_ref closure #1 in uniqueIntegerProvider()
  %7 = function_ref @closure #1 () -> Swift.Int in main.uniqueIntegerProvider() -> () -> Swift.Int : $@convention(thin) (@guaranteed { var Int }) -> Int // user: %10
  %8 = copy_value %0 : ${ var Int }               // user: %10
  mark_function_escape %1 : $*Int                 // id: %9
  %10 = partial_apply [callee_guaranteed] %7(%8) : $@convention(thin) (@guaranteed { var Int }) -> Int // user: %12
  destroy_value %0 : ${ var Int }                 // id: %11
  return %10 : $@callee_guaranteed () -> Int      // id: %12
} // end sil function 'main.uniqueIntegerProvider() -> () -> Swift.Int'
```

**优化后：**

```swift
// uniqueIntegerProvider()
sil hidden @main.uniqueIntegerProvider() -> () -> Swift.Int : $@convention(thin) () -> @owned @callee_guaranteed () -> Int {
bb0:
  %0 = alloc_box ${ var Int }, var, name "i"      // users: %8, %7, %6, %1
  %1 = project_box %0 : ${ var Int }, 0           // user: %4
  %2 = integer_literal $Builtin.Int64, 0          // user: %3
  %3 = struct $Int (%2 : $Builtin.Int64)          // user: %4
  store %3 to %1 : $*Int                          // id: %4
  // function_ref closure #1 in uniqueIntegerProvider()
  %5 = function_ref @closure #1 () -> Swift.Int in main.uniqueIntegerProvider() -> () -> Swift.Int : $@convention(thin) (@guaranteed { var Int }) -> Int // user: %7
  strong_retain %0 : ${ var Int }                 // id: %6
  %7 = partial_apply [callee_guaranteed] %5(%0) : $@convention(thin) (@guaranteed { var Int }) -> Int // user: %9
  strong_release %0 : ${ var Int }                // id: %8
  return %7 : $@callee_guaranteed () -> Int       // id: %9
} // end sil function 'main.uniqueIntegerProvider() -> () -> Swift.Int'
```

可以很明显的看出，无论是优化前还是优化后，使用的都是 `alloc_box` 指令，也就是说此时的变量 `i` 是存储在堆上的。其实原因也很好理解，其实就是变量 `i` 被函数闭合了，即使在退出作用域的情况下，仍然得保持 i 的存在。当然这只是一种情况，还会有其他的情况。

当然，这个举例可能会有些特殊，因为涉及到了 closure，可以再换个例子，比如说:

```swift
struct Content {
    var str = ""
}

class Info {
    var content = Content()
}

func test() {
    print(Info().content.str)
}
```

其实就是 struct 值类型对 class 引用类型包含，这种情况下其实 `content` 所处位置也是在在堆上。

看一下相关核心的 SIL 文件片段

```swift
// Info.init()
sil hidden @main.Info.init() -> main.Info : $@convention(method) (@owned Info) -> @owned Info {
// %0 "self"                                      // users: %2, %7, %1
bb0(%0 : $Info):
  debug_value %0 : $Info, let, name "self", argno 1 // id: %1
  %2 = ref_element_addr %0 : $Info, #Info.content // user: %6
  %3 = metatype $@thin Content.Type               // user: %5
  // function_ref Content.init()
  %4 = function_ref @main.Content.init() -> main.Content : $@convention(method) (@thin Content.Type) -> @owned Content // user: %5
  %5 = apply %4(%3) : $@convention(method) (@thin Content.Type) -> @owned Content // user: %6
  store %5 to %2 : $*Content                      // id: %6
  return %0 : $Info                               // id: %7
} // end sil function 'main.Info.init() -> main.Info'


// Content.init()
sil hidden @main.Content.init() -> main.Content : $@convention(method) (@thin Content.Type) -> @owned Content {
// %0 "$metatype"
bb0(%0 : $@thin Content.Type):
  %1 = alloc_stack $Content, let, name "self"     // users: %10, %11, %12
  %2 = string_literal utf8 ""                     // user: %7
  %3 = integer_literal $Builtin.Word, 0           // user: %7
  %4 = integer_literal $Builtin.Int1, -1          // user: %7
  %5 = metatype $@thin String.Type                // user: %7
  // function_ref String.init(_builtinStringLiteral:utf8CodeUnitCount:isASCII:)
  %6 = function_ref @Swift.String.init(_builtinStringLiteral: Builtin.RawPointer, utf8CodeUnitCount: Builtin.Word, isASCII: Builtin.Int1) -> Swift.String : $@convention(method) (Builtin.RawPointer, Builtin.Word, Builtin.Int1, @thin String.Type) -> @owned String // user: %7
  %7 = apply %6(%2, %3, %4, %5) : $@convention(method) (Builtin.RawPointer, Builtin.Word, Builtin.Int1, @thin String.Type) -> @owned String // user: %8
  %8 = struct $Content (%7 : $String)             // users: %10, %13, %9
  retain_value %8 : $Content                      // id: %9
  store %8 to %1 : $*Content                      // id: %10
  destroy_addr %1 : $*Content                     // id: %11
  dealloc_stack %1 : $*Content                    // id: %12
  return %8 : $Content                            // id: %13
} // end sil function 'main.Content.init() -> main.Content'
```

我想你肯定能看出点啥，其实 `Content` 构造时还是使用 `alloc_stack` 指令，但是在 `Info` 构造时实际上会进行一次拷贝，就是上面的 `store %5 to %2 : $*Content`。

那除此之外，还会有其他的吗？当然还有，可以自己再去发现。

**总结：所以说在 Swift 中所有的 `class` 都存储在堆上，所有的 `struct` 都存储在栈上这种说法是有问题的，只能说大部分情况是如此的，总有些情况会跟你淘气，具体存储位置还得结合结构所在上下文以及 SIL 优化手段等等因素综合分析。**

### 拷贝方式

引用类型，在拷贝时，实际上拷贝的只是栈区存储的对象的指针；值类型拷贝的是实际的值。

对于值类型拷贝，Swift 有一套 **写时复制 COW（Copy-On-Write）** 优化机制，即只有赋值后值类型发生改变的时候才会进行真正的拷贝，当没有改变时，两者共享同一个内存地址。

Apple 在 [OptimizationTips](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst#advice-use-copy-on-write-semantics-for-large-values) 中，给出了一个示例，代码很简单，相信大家一下就能明白。

> 该文档中还有一些 Apple 给出的另外的优化方式，比如减少动态派发的方式等等，建议 enjoy。

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

Swift 标准库中，`String`、`Array`、`Dictionary`、`Set` 等默认实现了 `COW`，对于自定义对象，我们需要自己实现。

## 最后

在编写本地文章过程中，查看了 Swift 开源仓库 [docs](https://github.com/apple/swift/tree/main/docs) 目录下的一些文档，学到了很多，也建议各位读者同学 enjoy！

要更加努力呀！

Let's be CoderStar!

更多资料

- [iOS Swift5：浅析结构体（struct）与类（class）](https://juejin.cn/post/6933231394227240967#heading-7)
- [Why Choose Struct Over Class?](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)
- [Memory Management and Performance of Value Types](https://swiftrocks.com/memory-management-and-performance-of-value-types)
- [Value Types and Reference Types in Swift](https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/)
- [why-choose-struct-over-class](https://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24243626#24243626)
- [reference-vs-value-types-in-swift](https://www.raywenderlich.com/9481-reference-vs-value-types-in-swift)
