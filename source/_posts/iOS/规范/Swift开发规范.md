---
title: Swift开发规范
category:
  - iOS
  - 开发规范
tags:
  - iOS
  - Swift开发规范
date: 2021-03-22 22:26:59
state: 已发
---

## 前言

开发规范的目的是保证统一项目成员的编码风格，并使代码美观，每个公司对于代码的规范也不尽相同，希望该份规范能给大家起到借鉴作用。该开发规范会持续更新，请关注该[博文链接](https://coder-star.github.io/iOS/%E8%A7%84%E8%8C%83/Swift%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/)。

## 一、命名规约

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
- 【强制】命名中出现缩略词时，缩略词要么全部大写，要么全部小写，以首字母大小写为准，通用缩略词包括 JSON、URL、ID等；
  ```swift
  正例：class IDUtil {} /  func idToString()
  反例：class IdUtils {} / func iDToString()
  ```
- 【强制】不要使用不规范的缩写，力求语义表达完整清楚，不怕名称长。
  ```swift
  反例：AbstractClass 缩写成 AbsClass
  ```
- 【推荐】全局常量命名使用 k 前缀 + UpperCamelCase 命名；
  说明：本质上是不推荐使用全局常量的，主要原因是会散落到代码各处，不方便管理
  ```swift
  正例：kMaxLocaolStoreCount
  ```
- 【推荐】扩展文件，用“原始类型名＋扩展名”作为扩展文件名，其中原始类型名及扩展名也使用 UpperCamelCase 风格，如果扩展文件中功能不属于同一类，也可使用“原生类型名+Extensions”的形式；
  ```swift
  正例：UIView+Frame.swift / MessageViewController+Request.swift / UIViewExtensions.swift
  ```
- 【推荐】工程中文件夹或者 Group 统一使用 UpperCamelCase 风格，一律使用单数形式；
- 【推荐】文件名如果有复数含义，文件名应使用复数形式，如一些工具类；
  ```swift
  正例：MessageUtils.swift
  ```

## 二、定义规约

- 不要使用魔法值(即未经定义的常量)；
- 能用 let 修饰的时候，不要使用 var；
- 修饰符顺序按照 注解、访问限制、static、final 顺序；
- 尽可能利用访问限制修饰符控制类、方法等的访问限制；
- 写方法时，要考虑这个方法是否会被重载。如果不会，标记为 final，final 会缩短编译时间；
- 在编写库的时候需要注意修饰符的选用，遵循开闭原则；

## 三、格式规约

- 类、函数左大括号不另起一行，与名称之间留有空格
- 禁止使用无用分号
- 代码中的空格出现地点
  - 注释符号与注释内容之间有空格
  - 类继承, 参数名和类型之间等, 冒号前面不加空格, 但后面跟空格
  - 任何运算符前后有空格
  - 表示返回值的 -> 两边
  - 参数列表、数组、tuple、字典里的逗号后面有一个空格
- 方法之间空一行
- 重载的声明放在一起，按照参数的多少从少到多向下排列
- 每一行只声明一个变量
- 如果是一个很长的数字时，建议使用下划线按照语言习惯三位或者四位一组分割连接。
- 表示单例的静态属性，一般命名为 shared 或者 default
- 如果是空的 block，直接声明{ }，括号之间不需换行
- 解包时推荐使用原有名字，前提是解包后的名字与解包前的名字在作用域上不会形成冲突
- if 后面的 else\else if, 跟着上一个 if\else if 的右括号
- switch 中, case 跟 switch 左对齐
- 每行代码长度应小于 100 个字符，或者阅读时候不应该需要滚动屏幕，在正常范围内可以看到完整代码
- 实现每个协议时, 在单独的 extension 里来实现

## 四、简略规约

- Swift 会被结构体按照自身的成员自动生成一个非 public 的初始化方法，如果这个初始化方法刚好适合，不要自己再声明
- 类及结构体初始化方法不要直接调用.init，直接直接省略，使用()
- 如果只有一个 get 的计算属性，忽略 get
- 数据定义时，简单类型尽量使用字面量形式进行自动推断，如果上下文不足以推断字面量类型或者类型比较复杂时，需要声明赋值类型
- 省略默认的访问权限（internal）
- 过滤, 转换等, 优先使用 filter, map 等高阶函数简化代码，并尽量使用最简写
- 使用闭包时，尽量使用最简写
- 使用枚举属性时尽量使用自动推断，进行缩写
- 无用的代码及时删除
- 尽量使用各种语法糖
- 访问实例成员或方法时尽量不要使用 `self.`，特殊场景除外，如构造函数时
- 当方法无返回值时，不需添加 void

## 五、注释规约

- 文档注释使用单行注释，即///，不使用多行注释，即/\*\*\*/。 多行注释用于对某一代码段或者设计进行描述
- 对于公开的类、方法以及属性等必须加上文档注释，方法需要加上对应的`Parameter（s）`、`Returns`、`Throws` 标签，强烈建议使用`⌥ ⌘ /`自动生成文档模板
- 在代码中灵活的使用一些地标注释，如`MARK`、`FIXME`、`TODO`，当同一文件中存在多种类型定义或者多种逻辑时，可以使用`Mark`进行分组注释
- 尽量将注释放在代码上一行，而不是放在代码后

## 六、编译效率规约

- 数组合并建议使用append方法而不是+号拼接；
- 字符串合并避免使用+号而是多采用"\(str1)\(str2)"的形式；
- if 的条件部分不要做过多运算，如 `if count == 60 * 60 / 2`;
- 尽量不使用Storyboard或者Xib，会增加编译时间；
- 减少三目运算符的使用

## 七、其他

- 函数参数最多不得超过 8 个；寄存器数目问题，超过 8 个会影响效率；
- 图形化的字面量，`#colorLiteral(...)`, `#imageLiteral(...)`只能用在 playground 当做自我练习使用，禁止在项目工程中使用
- 避免强制解包以及强制类型映射，尽量使用`if let` 或 `guard let`进行解包，禁止`try！`形式处理异常，避免使用隐式解包
- 避免判断语句嵌套层次太深，使用 guard 提前返回
- 如果 for 循环在函数体中只有一个 if 判断，使用 for where 进行替换
- 实现每个协议时, 尽量在单独的 extension 里来实现；但需要考虑到协议的方法是否有 override 的可能，定义在 extension 的方法无法被 override，除非加上@objc 方法修改其派发方式
- 优先创建函数而不是自定义操作符
- 尽可能少的使用全局命名空间，如常量、变量、方法等
- 赋值数组、字典时每个元素分别占用一行时，最后一个选项后面也添加逗号；这样未来如果有元素加入会更加方便
- 布尔类型属性使用 is 作为属性名前缀，返回值为布尔型类型的方法名使用 is 作为方法名作为前缀
- 类似注解的修饰词单独占一行，如@objc，@discardableResult 等
- extension 上不用加任何修饰符，修饰符加在 extension 内的变量或方法上
- 使用 guard 来提前结束条件，避免形成判断嵌套；
- 善用字典去减少判断，可将条件与结果分别当做 key 及 value 存入字典中；
- 封装时善用 assert，方便问题排查；
- 在闭包中使用 self 时使用捕获列表`[weak self]`避免循环引用，闭包开始判断 self 的有效性
- 使用委托和协议时，避免循环引用，定义属性的时候使用 weak 修饰

## 工具

[SwiftLint 工具](https://github.com/realm/SwiftLint) 提示格式错误

[SwiftFormat 工具](https://github.com/nicklockwood/SwiftFormat) 提示并修复格式错误

两者大部分格式规范都是一致的，少许规范不一致，两个工具之间使用不冲突，可以在项目中共存。我们通过配置文件可以控制启用或者关闭相应的规则，具体使用规则参照对应仓库的 REAMME.md 文件。

## 相关规范

[Swift 官方 API 设计指南](https://swift.org/documentation/api-design-guidelines/)

[Google 发布的 Swift 编码规范](https://google.github.io/swift/#apples-markup-format)
