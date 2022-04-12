---
title: SwiftUI 基本原理
category:
  - SwiftUI
tags:
  - SwiftUI
date: 2022-04-11 20:24:32
---

## 前言

Hi Coder，我是 CoderStar！

## 基础

### @propertyWarpper

### @resultBuilder

> 该`Attribute`在未公开之前叫做`@_functionBuilder`，Swift 5.4 修改为 `@resultBuilder`，按照 Apple 的习惯，一般没有公开的`Attribute`，前面都会有一个下划线`_`。

[awesome-result-builders](https://github.com/carson-katri/awesome-result-builders)

## SwiftUI 使用

### @ViewBuilder


> 因为Swift中泛型还不支持可变参数，所以只能像下面一样，提供11个模板方法，至于为什么参数最多是10个，可能更多是一个经验值。

```swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@_functionBuilder public struct ViewBuilder {
  @_alwaysEmitIntoClient public static func buildBlock() -> SwiftUI.EmptyView {
        EmptyView()
    }
  @_alwaysEmitIntoClient public static func buildBlock<Content>(_ content: Content) -> Content where Content : SwiftUI.View {
        content
    }
}

@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension SwiftUI.ViewBuilder {
  @_alwaysEmitIntoClient public static func buildBlock<C0, C1>(_ c0: C0, _ c1: C1) -> SwiftUI.TupleView<(C0, C1)> where C0 : SwiftUI.View, C1 : SwiftUI.View {
        return .init((c0, c1))
    }
}
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension SwiftUI.ViewBuilder {
  @_alwaysEmitIntoClient public static func buildBlock<C0, C1, C2>(_ c0: C0, _ c1: C1, _ c2: C2) -> SwiftUI.TupleView<(C0, C1, C2)> where C0 : SwiftUI.View, C1 : SwiftUI.View, C2 : SwiftUI.View {
        return .init((c0, c1, c2))
    }
}

...

@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
extension SwiftUI.ViewBuilder {
  @_alwaysEmitIntoClient public static func buildBlock<C0, C1, C2, C3, C4, C5, C6, C7, C8, C9>(_ c0: C0, _ c1: C1, _ c2: C2, _ c3: C3, _ c4: C4, _ c5: C5, _ c6: C6, _ c7: C7, _ c8: C8, _ c9: C9) -> SwiftUI.TupleView<(C0, C1, C2, C3, C4, C5, C6, C7, C8, C9)> where C0 : SwiftUI.View, C1 : SwiftUI.View, C2 : SwiftUI.View, C3 : SwiftUI.View, C4 : SwiftUI.View, C5 : SwiftUI.View, C6 : SwiftUI.View, C7 : SwiftUI.View, C8 : SwiftUI.View, C9 : SwiftUI.View {
        return .init((c0, c1, c2, c3, c4, c5, c6, c7, c8, c9))
    }
}
```

### @State

### @ObservedObject

@Binding

## 最后

要更加努力呀！

Let's be CoderStar!
