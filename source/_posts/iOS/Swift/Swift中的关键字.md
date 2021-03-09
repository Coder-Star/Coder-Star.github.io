---
title: Swift中的关键字
category:
  - Swift
tags:
  - Swift
  - 关键字
date: 2020-11-17 09:08:55
---

swift 中的关键字被保留，不可将其用作标识符，除非使用反引号``，如

```swift
default -> `default`
```

### 声明中的关键字

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

### 语句中的关键字

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
- repeat
- return
- switch
- where
- while

### 表达式和类型中的关键字

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

### 起始于#的关键字

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

### 起始于@的关键字

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


## 其他

- @inline(never) 声明这个函数never永远不被编译成inline的形式
- @inline(__always) 声明这个函数总是编译成inline的形式， 这种方式对防止反编译拿到核心方法逻辑有帮助。