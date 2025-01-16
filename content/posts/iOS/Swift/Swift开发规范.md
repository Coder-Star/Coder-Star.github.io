---
title: Swift开发规范
date: 2024-11-16T23:56:02+08:00
categories:
  - Swift
tags:
  - iOS
  - Swift
share: true
description: ""
cover.image: ""
lastmod: 2024-11-27T19:20:00+08:00
---

## 前言

代码的字里行间流淌的是软件生命中的血液，质量的提升是尽可能少踩坑，杜绝踩重复的坑，切实提升质量意识。另外，现代软件架构都需要协同开发完成，高效协作即降低协同成本，提升沟通效率，所谓无规矩不成方圆，无规范不能协作。众所周知，制订交通法规表面上是要限制行车权，实际上是保障公众的人身安全。试想如果没有限速，没有红绿灯，谁还敢上路行驶。对软件来说，适当的规范和标准绝不是消灭代码内容的创造性、优雅性，而是限制过度个性化，以一种普遍认可的统一方式一起做事，提升协作效率。-- 摘自《阿里巴巴 Java 代码规范》

该开发规范会持续更新，请关注该 [博文链接](https://coder-star.github.io/iOS/%E8%A7%84%E8%8C%83/Swift%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/)。

规约分为【强制】、【推荐】两大类。`说明` 对内容做了引申和解释；`正例` 给出正确的代码示例；`反例` 给出错误的代码示范；

## 命名规约

- 【强制】代码中的命名严禁使用拼音及英文混合的方式，更不允许直接出现中文的方式，最好也不要使用下划线或者美元符号开头；

  ```swift
  反例：_name $name / 学生 / getPingfenByName()[评分]
  ```

- 【强制】文件名、class、struct、enum、protocol 命名统一使用 UpperCamelCase 风格；

  ```swift
  正例：class LoginName { } / enum SexType { }
  反例：class loginName { } / enum SexTYPE { }
  ```

- 【强制】方法名、参数名、成员变量、局部变量、枚举成员统一使用 lowerCamelCase 风格

  ```swift
  正例：localValue / getMessageInfo()
  反例：LocalValue / GetMessageInfo()
  ```

- 【强制】命名中出现缩略词时，缩略词要么全部大写，要么全部小写，以首字母大小写为准，通用缩略词包括 `JSON`、`URL`、`ID` 等；

  ```swift
  正例：class IDUtil {} /  func idToString()
  反例：class IdUtils {} / func iDToString()
  ```

- 【强制】不要使用不规范的缩写，力求语义表达完整清楚，能直观的表达意图，不怕名称长；

  ```swift
  正例：class RoundAnimatingButton: UIButton {}
  反例：class CustomButton: UIButton {}  / AbstractClass 缩写成 AbsClass
  ```

- 【推荐】全局常量命名使用 k 前缀 + UpperCamelCase 命名；
  说明：本质上是不推荐使用全局常量的，主要原因是会散落到代码各处，不方便管理

  ```swift
  正例：kMaxLocaolStoreCount
  ```

- 【推荐】扩展文件，用 `原始类型名＋扩展名` 作为扩展文件名，其中原始类型名及扩展名也使用 UpperCamelCase 风格，如果扩展文件中功能不属于同一类，也可使用 `原生类型名 +Extensions` 的形式；

  ```swift
  正例：UIView+Frame.swift / MessageViewController+Request.swift / UIViewExtensions.swift
  ```

- 【推荐】工程中文件夹或者 Group 统一使用 UpperCamelCase 风格，一律使用单数形式；

  ```swift
  正例：Resource / Util
  ```

- 【推荐】文件名如果有复数含义，文件名应使用复数形式，如一些工具类；

  ```swift
  正例：MessageUtils.swift
  ```

- 【推荐】布尔类型属性使用 `is` 作为属性名前缀，返回值为布尔型类型的方法名使用 `is` 作为方法名作为前缀；

## 定义、修饰规约

- 【强制】不要使用魔法值（即未经定义的常量）；

  ```swift
  正例：
      let maxDisplayCount = 5
      if index == maxDisplayCount {}
  反例：
      /// 5的含义不明确
      if index == 5 {}
  ```

- 【强制】能用 `let` 修饰的时候，不要使用 `var`；
- 【强制】`extension` 上不用加任何修饰符，修饰符加在 `extension` 内的变量或方法上；
  说明：目的是当修改 `extension` 中某个方法的访问限制时，不需去考虑外部的 `extension` 访问限制，降低影响面。

  ```swift
  正例：
      extension UIView {
        public func removeAllSubView() {}
      }
  反例：
      public extension UIView {
        func removeAllSubView() {}
      }
  ```

- 【推荐】修饰符顺序按照 注解、访问限制、`static`、`final` 顺序；
  说明：注解是指起始于 @的关键字，如 `@discardableResult、@objc` 等；访问限制是指 `public`、`private` 等；

  ```swift
  正例：
      @objc
      public static final func getMessageInfo() {}
  ```

- 【推荐】考虑方法是否会被重写。如果不会，标记为 `final`；
  说明：Swift 在编译时会优化 `final` 修饰的方法，派发方式可能由函数表派发优化为直接派发。
- 【推荐】尽可能利用访问限制修饰符控制类、方法等的访问限制，遵循开闭原则；
  说明：如确定某方法或变量不应该被外部调用，就使用 `private` 进行修饰，编译程序阻止外部不合适的调用。同时 `private` 在 Swift 中会被隐式加上 `final` 修饰，从而得到优化。善用 `private(set)` 这种方式对读写权限单独控制。并且当范围足够小时，我们后期对其调整的代价就会更小，如果太大，导致外部使用了，后续你就需要考虑更多的兼容。
- 【推荐】表示单例的静态属性，一般命名为 `shared` 或者 `default`，并切记将构造函数私有，否则单例毫无意义；

  ```swift
  正例：
      class ApplicationServiceManager {
          public static let shared =  ApplicationServiceManager()
          private init {}
      }
  ```

## 格式规约

- 【强制】类、函数左大括号不另起一行，与名称之间留有空格；
- 【强制】代码中的空格出现地点
  - 注释符号与注释内容之间有空格；
  - 类继承，参数名和类型之间等，冒号前面不加空格，但后面跟空格；
  - 任何运算符前后有空格；
  - 表示返回值的 -> 两边；
  - 参数列表、数组、元祖、字典里的逗号后面有一个空格；
- 【强制】禁止使用无用分号；
- 【强制】方法之间空一行；
- 【强制】重载的声明放在一起，按照参数的多少从少到多向下排列；
- 【强制】每一行只声明一个常、变量；
- 【强制】如果大括号内为空，直接简写为{}，括号之间不需换行；
- 【强制】`if` 后面的 `else\else if`, 跟着上一个 `if\else if` 的右括号；
- 【强制】`switch` 中，`case` 跟 `switch` 左对齐；
- 【推荐】每行代码长度应小于 100 个字符，或者阅读时候不应该需要滚动屏幕，在正常范围内可以看到完整代码；
- 【推荐】解包时推荐使用原有名字，前提是解包后的名字与解包前的名字在作用域上不会形成冲突；
- 【推荐】实现每个协议时，在单独的 `extension` 里来实现；
- 【推荐】赋值数组、字典时每个元素分别占用一行时，最后一个选项后面也添加逗号；
   说明：这样未来如果有元素加入会更加方便；

代码示例（代码不具有业务含义，只是简单的格式规约示例）

```swift
/**
 涉及规约
 1、类左大括号不另起一行；
 2、类继承后跟空格；
 */

/// 格式规约示例
class FormatSample: NSObject {
    /**
     涉及规约
     1、注释符号与注释内容之前有空格；
     2、每一行只声明一个变量；
     3、不使用分号；
     4、注释另起一行，不放在行尾；
     5、数组、元祖、字典里的逗号后面有一个空格；
     */

    private var resultCode = ""
    private var resultArr = ["one", "two"]
    private var resultDic = ["key": "name", "value": "张三"]
    private var resultTuple = (key: "name", value: "张三")
}

/**
 涉及规约
 1、方法之间空一行；
 2、重载的声明放在一起，按照按照参数的多少从少到多排序；
 3、返回值 -> 两遍增加空格；
 4、参数名与类型之间空格；
 5、如果大括号内为空，则直接简写为{}，括号内不换行；
 6、if 后面的 else\else if, 跟着上一个 if\else if 的右括号；
 7、解包时推荐使用原有名字；
 */
extension FormatSample {
    private func logInfo(message: String) {
        logInfo(message: message, type: nil)
    }

    private func logInfo(message: String, type: String?) {
        if let type = type {
            print("\(type): \(message)")
        } else {
            print(message)
        }
    }

    private func canLog() -> Bool {
        #if DEBUG
            return true
        #else
            return false
        #endif
    }

    /**
     涉及规约
     1、switch 中, case 跟 switch 左对齐；
    */
    private func showSwitchStandardFormat() {
      let count = 10
      switch count {
      case 1:
        print(1)
      // 如case包含所有情况，可不加default，如遍历枚举类型时
      default:
        break
      }
    }
}
```

## 简略规约

- 【强制】Swift 会被结构体按照自身的成员自动生成一个非 public 的初始化方法，如果这个初始化方法刚好适合，不要自己再声明；

  ```swift
  /// 会自动生成 init(name: String) 这样的构造函数，如果符合使用，不要再手动添加该构造函数
  struct LoginInfo {
    var name: String
  }
  ```

- 【强制】类及结构体初始化方法不要直接调用 `.init`，直接直接省略，使用 ()；

  ```swift
  正例：
     let loginView = UIView()
  反例：
     let loginView = UIView.init()
  ```

- 【强制】如果只有一个 `get` 的计算属性，忽略 `get`；

  ```swift
  正例：
      var info: String {
        return ""
      }
  反例：
      var info: String {
        get {
          return ""
        }
      }
  ```

- 【强制】数据定义时，简单类型尽量使用字面量形式进行自动推断，如果上下文不足以推断字面量类型或者类型比较复杂时，需要声明赋值类型；

  ```swift
  正例：var info = ""
  反例：var info: String = ""
  ```

- 【强制】省略默认的访问权限（internal）；

  ```swift
  正例：var info = ""
  反例：internal var info = ""
  ```

- 【强制】使用枚举属性时使用自动推断，进行缩写；

  ```
  enum Sex {
    case male
    case female
  }
  正例：let sex: Sex = .male
  反例：let sex: Sex = Sex.male
  ```

- 【强制】switch-case 里不用显式添加 `break`;

  ```swift
  let count = 10
  switch count {
  case 1:
    print(1)
    // 此处不用显式添加break，Swift中每个case都会默认break。
  }
  ```

- 【强制】访问实例成员或方法时不要使用 `self.`，特殊场景除外，如构造函数，闭包内；

  ```swift
  class LoginInfo {
    func log() {}

    func recordInfo() {
      /// 正例
      log()

      /// 反例
      self.log()
    }
  }
  ```

- 【强制】当方法无返回值时，不需添加 `void`；

  ```swift
  正例：func getMessageInfo() {}
  反例：func getMessageInfo() -> Void {}
  ```

- 【强制】无用的代码及时删除；
  **说明：可能有开发者觉得有些代码可能后续会用到，所以采取了注释的方式。但是这种方式很容易演变成代码会一直放在那，永远不会删掉。即使觉得后续会用到，也请及时删除掉，不然 Git 留着干什么用呢？**
- 【推荐】使用闭包时，尽量使用最简写，如优先使用尾随闭包等；
- 【推荐】过滤，转换等，优先使用 filter, map 等高阶函数简化代码，并尽量使用最简写；
- 【推荐】**尽量**使用各种语法糖；
   说明：语法糖一定程度上会降低代码的可度性，尽量这个度根据各自公司进行调整。
- 【推荐】类似注解的修饰词单独占一行，如 `@objc`，`@discardableResult` 等；
- 【推荐】如果 for 循环在函数体中只有一个 if 判断，使用 `for where` 进行替换；

## 注释、可读规约

- 【强制】文档（API）注释使用单行注释，即 `///`，不使用多行注释，即 `/** */`。 多行注释用于对某一代码段、设计或者复杂业务进行描述；
- 【强制】对于公开的类、方法以及属性等必须加上文档（API）注释，方法需要加上对应的 `Parameter（s）`、`Returns`、`Throws` 等标签，建议使用 `⌥ ⌘ /` 自动生成文档模板；
- 【强制】将注释放在代码上一行，而不是放在代码后；
  说明：放在代码后有两个弊端，一是当代码稍微长一点后，注释可能需要横向滚动后才能看全；另一个弊端是，当代码修改，极易将注释删除，或者由于后面有注释，前面的代码修改起来有些许不方便。
- 【推荐】在代码中灵活的使用一些地标注释，如 `MARK`、`FIXME`、`TODO`，当同一文件中存在多种类型定义或者多种逻辑时，可以使用 `Mark` 进行分组注释，方便通过 `Xcode` 顶部面包屑进行切换；
- 【推荐】实现每个协议时，尽量在单独的 extension 里来实现；
代码示例：

```swift
// MARK: - View子视图操作相关

extension UIView {
    /// 同时添加多个视图
    /// - Parameter subviews: 子View可变参数
    public func addSubviews(_ subviews: UIView...) {
        subviews.forEach(addSubview)
    }

    /// 移除所有子View
    public func removeAllSubview() {
        subviews.forEach {
          $0.removeFromSuperview()
        }
    }
}
```

## 编译效率规约

该段是提高编译效率，减少编译时间总结提出的规约，对代码可读性及统一风格等影响不大，所以该部分只是作为推荐，不强制一定遵守。

- 【推荐】数组合并建议使用 append 方法而不是 + 号拼接；

    ```swift
   var resultArr = ["1", "2"]
   let extraArr = ["3", "4"]
   正例：resultArr.append(contentsOf: extraArr) / let newArray = [resultArr, extraArr].joined()
   反例：resultArr += extraArr  / let newArray = resultArr + extraArr
   ```

- 【推荐】字符串合并避免使用 + 号而是多采用 `"\(str1)\(str2)"` 的形式；

   ```swift
      let code = "200"
      let message = "success"
   正例：let result = "\(code)\(message)"
   反例：let result = code + message
   ```

- 【推荐】if 的条件部分不要做过多运算

  ```swift
  正例：if count == 900 {}
  反例：if count == 60 * 60 / 2 / 2 {}
  ```

- 【推荐】不要让可选值使用？? 赋默认值再嵌套其他运算；
- 【推荐】将长计算式代码拆分，最后组合计算；
- 【推荐】尽量不使用 `Storyboard` 或者 `Xib`，会增加编译时间；
    还存在一些问题可见 [think-twice-before-scaling-your-app-with-interface-builder](https://craft.faire.com/think-twice-before-scaling-your-app-with-interface-builder-90214ebdb12a)
- 【推荐】减少三目运算符的使用；

## 优化规约

- 【强制】函数参数数量最多不得超过 8 个；
  说明：寄存器数目问题，超过 8 个会影响效率；
- 【强制】避免强制解包以及强制类型映射，尽量使用 `if let` 或 `guard let` 进行解包，禁止 `try！` 形式处理异常，避免使用隐式解包；
- 【强制】避免判断语句嵌套层次太深，使用 guard 提前返回；
- 【推荐】在闭包中使用 self 时使用捕获列表 `[weak self]` 避免循环引用，闭包开始判断 self 的有效性；

  ```swift
  正例：
      timer = Timer.scheduledTimer(withTimeInterval: 2, repeats: true) { [weak self] _ in
          guard let self = self else { return }
          self.timerHandle()
      }
  ```

- 【推荐】使用委托和协议时，避免循环引用，定义属性的时候使用 `weak` 修饰；
- 【推荐】能用 `struct` 解决的，尽量使用 `struct` 而不是 `class`；
  说明：`struct` 属于值类型，使用其有很多好处：效率高、不需担心循环引用问题、线程安全等。
- 【推荐】对 String 判空时优先采用 `isEmpty`
  说明：Swift 里面的 String 的 `index` 和 `count` 不是一一对应的（兼容 `Unicode`），所以 `stirng.count == 0` 的效率不如 `string.isEmpty`；[Why using isEmpty is faster than checking count == 0](https://www.hackingwithswift.com/articles/181/why-using-isempty-is-faster-than-checking-count-0)
- 【推荐】如果协议只会被类使用，建议加上 `AnyObject` 限制；
  说明：编译器可以对此进行优化，其可以明确在处理一个类，继而进行引用计数，而不用考虑结构体以及其带来的额外的逻辑及结构。

   ```swift
   正例：protocol Pingable: AnyObject { func ping() -> Int }
   ```

-

## 设计规约

- 【推荐】善用 `func` 嵌套；
  说明：比如一段逻辑需要在方法内复用，可以在方法中定义方法，比如闭包中以及正常方法体中需要执行同一段逻辑；
- 【推荐】委托性质的协议，第一个参数应尽量保证为代理源，类似 `UITableViewDatasoure`；
- 【推荐】在方法设计时，其需要返回值，但是大部分情况业务方不需要使用返回值，可使用 `@discardableResult` 来对该方法进行标注，防止出现大量 `Warning`；
- 【推荐】尽可能少的使用全局命名空间，如常量、变量、方法等；
- 【推荐】优先创建函数而不是自定义操作符；
- 【推荐】Swift 中的协议和泛型十分强大，设计时应优先考虑面向协议编程；
- 【推荐】尽量不使用 `Bool？` 或者 `Array?` 这两者类型，会造成使用模糊。

## 其他

- 【强制】图形化的字面量，`#colorLiteral(...)`, `#imageLiteral(...)` 只能用在 playground 当做自我练习使用，禁止在项目工程中使用，这会让维护者在后期维护时无法第一眼了解到颜色的 hex 值或者图片的名称。
  说明：图形化的字面量不仅不方便直观的查看其颜色值或者图片名字，也不利于颜色、图片统一配置、管理。

## 工具

[SwiftLint 工具](https://github.com/realm/SwiftLint) 提示格式错误

[SwiftFormat 工具](https://github.com/nicklockwood/SwiftFormat) 提示并修复格式错误

两者大部分格式规范都是一致的，少许规范不一致，两个工具之间使用不冲突，可以在项目中共存。我们通过配置文件可以控制启用或者关闭相应的规则，具体使用规则参照对应仓库的 REAMME.md 文件。

## 相关规范

- [Swift 官方 API 设计指南](https://swift.org/documentation/api-design-guidelines/)
- [Google 发布的 Swift 编码规范](https://google.github.io/swift/#apples-markup-format)
- [linkedin编码规范](https://github.com/linkedin/swift-style-guide)
- [raywenderlich编码规范](https://github.com/raywenderlich/swift-style-guide)
- [airbnb编码规范](https://github.com/airbnb/swift)
- [swift-best-practices](https://github.com/Lickability/swift-best-practices)
