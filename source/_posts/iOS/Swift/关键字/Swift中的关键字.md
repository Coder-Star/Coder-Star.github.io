---
title: Swift 中的关键字
category:
  - iOS
  - Swift
tags:
  - Swift
  - 关键字
date: 2020-11-17 09:08:55
---

## 声明中的关键字

- class
- enum
- protocol
- struct
- let
- var
- init
- deinit
- private
- fileprivate
- internal
- public
- open
- extension
- func
- import
- inout
- operator
- static
- subscript
- typealias
- associatedtype

## 语句中的关键字

- break
- case
- continue
- default
- defer
- else
- fallthrough
- for
- guard
- if
- in
- repeat  // `repeat { } while`，swift 2.0 之间可以 do while 之后 do 只用来
- return
- switch
- where
- while

## 表达式和类型中的关键字

- as
- is
- Type //swift4.0 中已经废弃了
- nil
- do
- try
- catch
- throw
- throws
- rethrows
- super
- self
- Self
- false
- true

特定上下文中的关键字

- associativity
- convenience
- dynamic
- willSet
- didSet
- final
- required
- indirect
- lazy
- left
- right
- mutating
- nonmutating
- none
- optional
- override
- precedence
- postfix
- prefix
- infix
- Protocol
- get
- set
- Type
- unowned
- weak

## 起始于#的关键字

- #available、
- #if
- #elseif
- #else
- #endif
- #file
- #line
- #column
- #function
- #selector
- #sourceLocation

## 起始于 @的关键字

- @available
- @discardableResult
- @GKInspectable
- @nonobjc
- @objc
- @NSCopying
- @NSManaged
- @objcMembers
- @testable
- @NSApplicationMain
- @UIApplicationMain
- @IBAction
- @IBOutlet
- @IBDesignable
- @IBInspectable
- @autoclosure
- @escaping
- @convention
- @_functionBuilder
- @dynamicMemberLookup
- @propertyWrapper
- @frozen
- @inlinable
- @_dynamicReplacement
- @_transparent // 是为了告诉编译器在需要的时候可以将声明的函数内联，即使在 -Onone 下也是如此。@_transparent 实际上是比 @_inlineable 更强的存在，所有的 @_transparent 声明其实也隐式包含了 @_inlineable

[Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)

```swift
@frozen 和 @inlinable 是保证这个enum, struct, function的结构不变
@frozen 是对 enum, struct 使用
@inlinable 是对 function 使用

可以保证在项目中引用的某 framework 替换后仍然不需要重新编译,
因为 enum, struct, function 的链接没有发生改变
```

非官方关键字

- _specialize

```swift
@_specialize(where T == Int)
@_specialize(where T == Int)
public func min<T: Comparable>(_ x: T, _ y: T) -> T {
    returny<x?y:x
}
```

### @available

@available: 可用来标识计算属性、函数、类、协议、结构体、枚举等类型的生命周期。（依赖于特定的平台版本 或 Swift 版本）

至少有两个特征参数，参数之间逗号分割。参数由以下平台起头：

- iOS
- iOSApplicationExtension
- macOS
- macOSApplicationExtension
- watchOS
- watchOSApplicationExtension
- tvOS
- tvOSApplicationExtension
- swift

可以使用 \*号代替所有平台。

剩余参数如下：

- unavailable: 在当前平台下无效
- introduced：指定平台从哪一版本开始引入该声明，后需要跟版本号
- deprecated：指定平台从哪一版本开始弃用该声明。虽然被弃用，但是依然使用的话也是没有问题的。后可以跟版本号，若省略版本号，则表示目前弃用，同时可直接省略冒号。
- obsoleted：指定平台从哪一版本开始废弃该声明。当一个声明被废弃后，它就从平台中移除，不能再被使用。
- message：说明消息，当使用被弃用或者被废弃的声明时，编译器会抛出警告或错误信息。
- renamed：新的声明名称信息。当使用旧声明时，编译器会报错提示用户修改为新名字。即 fix 操作。

完成使用可以如下：

```swift
@available(iOS, introduced: 1.0, deprecated: 2.0, obsoleted: 3.0, renamed: "new
Info", message: "use newInfo")
@available(macOS, unavailable)
public func info() { }


// 如果除了平台参数外，只指定了一个introduced参数，可简写为如下
@available(iOS 12, *)
public func info() { }


@available(iOS 12, macOS 10.14 ,*)
public func info() { }
```

### @_silgen_name

@_silgen_name 是 Swift 中间语言 SIL 的一个属性

* 第一个是指定函数符号，为了让 C 调用；
* 第二个可以只声明一个函数符号，没有函数体，真正的函数实现是对应 C 的函数符号实现；

第一个场景可以是这样，加在 Swift 函数上，这样可以将该函数符号暴露出去，外部可以通过不直接引用的方式直接访问到该函数，用在组件路由上。[SRouter](https://tannerjin.github.io/2019/11/04/SRouter/)

第二个场景可以是这样，获取 iOS 或者 Swift 底层函数的实现，进而获取到一些 Swift 层没有的东西。

## 其他

- @inline(never) 声明这个函数 never 永远不被编译成 inline 的形式
- @inline(\_\_always) 声明这个函数总是编译成 inline 的形式， 这种方式对防止反编译拿到核心方法逻辑有帮助。
