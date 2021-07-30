---
title: Swift中模式匹配
category:
  - iOS
  - Swift
tags:
  - Swift
  - 模式匹配
date: 2021-02-04 23:10:42
---

## Swift 中模式匹配模式总览

- 通配符模式（Wildcard Pattern）
- 标识符模式（Identifier Pattern）
- 值绑定模式（Value-Binding Pattren）
- 元祖模式（Tuple Pattern）
- 枚举模式（Enumeration Case Pattern）
- 可选模式（Optional Pattern）
- 类型转换模式(Type-Casting Pattern)
- 表达式模式（Expression Pattern）

## 通配符模式

使用 `_` 来匹配任何值，使用 `_?` 来匹配非 nil 值，其中`_?`是通配符以及可选模式的混合

```swift
for _ in 1...3 {  f
}
```

## 标识符模式

```swift
let someValue = 42
```

## 值绑定模式

```swift
// 值绑定模式
case let (x, y):
  Log.d("\(x),\(String(describing: y))")
  break
// 值绑定模式
case (let x, let y):
  Log.d("\(x),\(String(describing: y))")
  break
```

## 元祖模式

```swift
// 元祖模式
for (x, y) in list {
  Log.d("\(x),\(String(describing: y))")
}
```

## 枚举模式

```swift
let age = 2
if case 0...9 = age {

}

guard case 0...9 = age else { return }

let ages: [Int?] = [2, 3, nil, 5]
for case nil in ages {
  Log.d("有nil值")
  break
}

let points = [(1, 0), (2, 1), (3, 0)]
for case let (x, 0) in points {
    Log.d(x)
}
```

## 可选模式

```swift
let ages: [Int?] = [2, 3, nil, 5]
// age?是匹配非空值
for case let age? in ages {
  Log.d(age)
}

// 非空并且里面包装的是2
for case 2? in ages {
  Log.d(age)
}
```

## 类型转换模式

```swift
let num: Any = 6
switch num {
  // 仅仅是判断num是否是Int类型，编译器并没有进行强转，编译器依然认为num是Any类型
  case is Int:
        Log.d(num)
  // 判断能不能将num强转成Int，如果可以就强转，然后赋值给n，最后n是Int类型，num还是Any类型
  case let n as Int:
        Log.d(n)
}
```

## 表达式模式

```swift

let point = (1, 2)
switch point {
case (-2...2, -2...2):
    Log.d(point)
}
```

复杂的 case 匹配⾥⾯其实调⽤了`~=`运算符来匹配，我们可以重载`~=`运算符来重新定制匹配规则

```swift
 //⾃定义Student和Int的匹配规则
 static func ~= (pattern: Int, value: Student) -> Bool { value.score >= pattern }
 //⾃定义Student和闭区间的匹配规则
 static func ~= (pattern: ClosedRange<Int>, value: Student) -> Bool { pattern.contains(value.score) }
 //⾃定义Student和开区间的匹配规则
 static func ~= (pattern: Range<Int>, value: Student) -> Bool { pattern.contains(value.score) }
```
