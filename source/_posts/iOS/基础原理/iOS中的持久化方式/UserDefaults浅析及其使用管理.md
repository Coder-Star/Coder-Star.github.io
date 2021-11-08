---
title: UserDefaults 浅析及其使用管理
category:
  - iOS
tags:
  - iOS
date: 2021-07-27 21:20:21
---

## 前言

Hi Coder，我是 CoderStar！

我想每一个 iOSer 对`UserDefaults`都有所了解，但大家真的完全了解它吗？下面，我谈谈我对`UserDefaults`的看法。

> 同时，这也应该是 iOS 持久化方式系列的开篇文章了。

## 对象实例

`UserDefaults`生成对象实例大概有以下三种方式：

```swift
open class var standard: UserDefaults { get }

public convenience init()

@available(iOS 7.0, *)
public init?(suiteName suitename: String?)
```

平时大家经常使用的应该是第一种方式，第二种方式和第一种方式产生的结果是一样的，实际上操作的都是 **APP 沙箱中 `Library/Preferences` 目录下的以 `bundle id` 命名的 `plist` 文件**，只不过第一种方式是获取到的是一个单例对象，而第二种方式每次获取到都是新的对象，从内存优化来看，很明显是第一种方式比较合适，其可以避免对象的生成和销毁。

如果一个 APP 使用了一些 SDK，这些 SDK 或多或少的会使用`UserDefaults`来存储信息，如果都使用前两种方式，这样就会带来一系列问题：

- 各个 SDK 需要保证设置数据 KEY 的唯一性，以防止存取冲突；
- `plist` 文件越来越大造成的读写效率问题；
- 无法便捷的清除由某一个 SDK 创建的 `UserDefaults` 数据；

针对上述问题，我们可以使用第三种方式，也是本文主要介绍的一种方式。

```Swift
@available(iOS 7.0, *)
public init?(suiteName suitename: String?)
```

根据传入的 `suiteName`的不同会产生四种情况：

- 传入 `nil`：跟使用`UserDefaults.standard`效果相同；
- 传入 `bundle id`：无效，返回 nil；
- 传入 `App Groups` 配置中 `Group ID`：会操作 APP 的共享目录中创建的以`Group ID`命名的 `plist` 文件，方便宿主应用与扩展应用之间共享数据；
- 传入其他值：操作的是沙箱中 `Library/Preferences` 目录下以 `suiteName` 命名的 `plist 文件。

## 相关问题

### `UserDefaults`的存储范围

因为`UserDefaults`底层使用的`plist`文件，所以`plist`文件支持的数据类型就是`UserDefaults`的存储范围，其中包括`Array`、`Data`、`Dictionary`、`String`、`Int`、`Bool`、`Float`、`Double`、`Date`等基础数据类型。并且对于`Array`、`Dictionary`这两个容器对象，其包含的内容也必须是上述类型，并且`Dictionary`的 key 必须是`String`类型。

对于不是基本数据类型的数据结构，需要自己通过`JSONEncoder`、`NSKeyedArchiver`等方式将其转换为 Data，然后再将其存入`UserDefaults`中。

需要注意，`UserDefaults`的设计初衷就不是用来存储大数据的，因为为了提高取值时的效率，当应用启动时会自动加载 `Userdefault` 里所有的数据，如果数据量太大的话就会造成启动缓慢，影响性能。

因为`UserDefaults`存储的数据都是明文，没有经过加密，所以尽量不要使用`UserDefaults`存储敏感数据，即使使用，也要使用加密算法对其进行加密后再存储进去。

### 尺寸限制

在`UserDefaults`中，有一个`sizeLimitExceededNotification`属性很清楚的回答了这个问题。

```swift
/**
NSUserDefaultsSizeLimitExceededNotification is posted on the main queue when more data is stored in user defaults than is allowed.
Currently there is no limit for local user defaults except on tvOS, where a warning notification will be posted at 512kB, and the process terminated at 1MB.
For ubiquitous defaults, the limit depends on the logged in iCloud user.
*/
@available(iOS 9.3, *)
public class let sizeLimitExceededNotification: NSNotification.Name
```

翻译过来就是
- 除了 tvOS 之外，其他的系统是没有限制。
- 在 tvOS 上，警告通知将在 512kB 处发布，进程在 1MB 处终止;

### `value(forKey:)` 和 `object(forKey:)`

首先明确这两者是完全不同的东西，`value(forKey:)`定义于`NSKeyValueCoding`，就是我们常说的 KVC，其并不是`UserDefaults`的直接方法，`object(forKey:)`才是。

但由于`UserDefaults`也是遵循了`NSKeyValueCoding`协议的，所以使用`value(forKey:)`也是可以获取到数据，但是不建议这种用法。在 `UserDefaults` 里面最好使用`object(forKey:)`，这是标准用法。

### `UserDefaults` 底层也是使用的 `plist` 文件，那它和普通的 `plist` 文件读取有什么区别呢？

主要区别是：`UserDefaults`会自动帮我们做 `plist` 文件的存取并在内存中做了缓存。其中需要注意的是`UserDefaults`对数据的操作影响`plist`文件的改变这一过程是异步的，也就是说你修改了`UserDefaults`某一个 key 的值，紧接着去获取这个 key 的值，得到的也会是修改后的值，但此时`plist`文件中对应的值可能还是修改前的。

从 iOS 8 开始，会有一个常驻进程 `cfprefsd` 来负责异步更新`plist`文件这一任务。所以 `UserDefaults` 的`synchronize`函数废弃也是有道理的，因为其本质上保证不了调用之后会将值立即存储到 plist 文件中。看一下`synchronize`函数上的注释吧。

```swift
/**
-synchronize is deprecated and will be marked with the API_DEPRECATED macro in a future release.

-synchronize blocks the calling thread until all in-progress set operations have completed. This is no longer necessary. Replacements for previous uses of -synchronize depend on what the intent of calling synchronize was. If you synchronized...
- ...before reading in order to fetch updated values: remove the synchronize call
- ...after writing in order to notify another program to read: the other program can use KVO to observe the default without needing to notify
- ...before exiting in a non-app (command line tool, agent, or daemon) process: call CFPreferencesAppSynchronize(kCFPreferencesCurrentApplication)
- ...for any other reason: remove the synchronize call
*/
open func synchronize() -> Bool
```

> 本质上，我们是可以通过文件操作的方式对 `UserDefaults` 的最终产物 `plist` 文件进行操作的，但这是有风险的，最好不要这么操作。

> 其实 iOS 的`UserDefaults`和 Android 的`SharedPreferences`两者很像，都分为内存、磁盘两级存储，其中Android磁盘使用的是XML形式存储，内存使用的是HashMap结构。而iOS磁盘使用的是plist文件，其本质其实还是一个XML文件，而内存使用的是C实现结构体，源码可见[UserDefaults底层存储结构](https://github.com/apple/swift-corelibs-foundation/blob/34887cb/CoreFoundation/Base.subproj/ForFoundationOnly.h#L193)

## 使用管理

经常会在一些项目中看到`UserDefaults`的数据存、取操作，`key`直接用的字符串魔法变量，搞到最后都不知道项目中`UserDefaults`到底用了哪些 key，对 key 的管理没有很好的重视起来。下面介绍两种`UserDefaults`使用管理的两种方式。

### protocol

利用 Swift 中`protocol`可以有默认实现的特性，可以对`UserDefaults`进行有效的管理。

直接上代码吧，相信大家一看应该就能明白。

```swift
/// UserDefaults存储协议，建议用枚举去实现该协议
public protocol UserDefaultsProtocol {
    // MARK: - 存储key

    /// 存储key
    var key: String { get }

    // MARK: - 存在nil

    /// 获取值
    var object: Any? { get }

    /// 获取url
    var url: URL? { get }

    // MARK: - 存在nil，有默认值

    /// 获取字符串值
    var string: String? { get }
    /// 获取字符串值,默认值为空
    var stringValue: String { get }

    /// 获取字典值
    var dictionary: [String: Any]? { get }
    /// 获取字典值,默认值为空
    var dictionaryValue: [String: Any] { get }

    /// 获取列表值
    var array: [Any]? { get }
    /// 获取列表值,默认值为空
    var arrayValue: [Any] { get }

    /// 获取字符串列表值
    var stringArray: [String]? { get }
    /// 获取字符串列表值,默认值为空
    var stringArrayValue: [String] { get }

    /// 获取Data值
    var data: Data? { get }
    /// 获取Data值,默认值为空
    var dataValue: Data { get }

    // MARK: - 不存在nil

    /// 获取Bool值,有默认值
    var bool: Bool { get }

    /// 获取int值,有默认值
    var int: Int { get }

    /// 获取float值,有默认值
    var float: Float { get }

    /// 获取double值,有默认值
    var double: Double { get }

    // MARK: - 方法

    /// 存储
    /// - Parameter object: 存储object型
    func save(object: Any?)

    /// 存储
    /// - Parameter int: 存储int型
    func save(int: Int)

    /// 存储
    /// - Parameter float: 存储float型
    func save(float: Float)

    /// 存储
    /// - Parameter double: 存储double型
    func save(double: Double)

    /// 存储
    /// - Parameter bool: 存储bool型
    func save(bool: Bool)

    /// 存储
    /// - Parameter url: 存储url型
    func save(url: URL?)

    /// 移除
    func remove()
}

// MARK: - 协议方法及计算属性实现

extension UserDefaultsProtocol {
    // MARK: - 存在nil

    /// 获取object
    public var object: Any? {
        return UserDefaults.standard.object(forKey: key)
    }

    /// 获取url
    public var url: URL? {
        return UserDefaults.standard.url(forKey: key)
    }

    // MARK: - 存在nil，有默认值

    /// 获取字符串值
    public var string: String? {
        return UserDefaults.standard.string(forKey: key)
    }

    /// 获取字符串值,默认值为空
    public var stringValue: String {
        return UserDefaults.standard.string(forKey: key) ?? ""
    }

    /// 获取字典值
    public var dictionary: [String: Any]? {
        return UserDefaults.standard.dictionary(forKey: key)
    }

    /// 获取字典值,默认值为空
    public var dictionaryValue: [String: Any] {
        return UserDefaults.standard.dictionary(forKey: key) ?? [String: Any]()
    }

    /// 获取列表值
    public var array: [Any]? {
        return UserDefaults.standard.array(forKey: key)
    }

    /// 获取列表值,默认值为空
    public var arrayValue: [Any] {
        return UserDefaults.standard.array(forKey: key) ?? [Any]()
    }

    /// 获取字符串列表值
    public var stringArray: [String]? {
        return UserDefaults.standard.stringArray(forKey: key)
    }

    /// 获取字符串列表值,默认值为空
    public var stringArrayValue: [String] {
        return UserDefaults.standard.stringArray(forKey: key) ?? [String]()
    }

    /// 获取Data值
    public var data: Data? {
        return UserDefaults.standard.data(forKey: key)
    }

    /// 获取Data值,默认值为空
    public var dataValue: Data {
        return UserDefaults.standard.data(forKey: key) ?? Data()
    }

    // MARK: - 不存在nil

    /// 获取Bool值
    public var bool: Bool {
        return UserDefaults.standard.bool(forKey: key)
    }

    /// 获取int值
    public var int: Int {
        return UserDefaults.standard.integer(forKey: key)
    }

    /// 获取float值
    public var float: Float {
        return UserDefaults.standard.float(forKey: key)
    }

    /// 获取double值
    public var double: Double {
        return UserDefaults.standard.double(forKey: key)
    }

    // MARK: - 方法

    /// 存储
    /// - Parameter value: 存储object
    public func save(object: Any?) {
        UserDefaults.standard.set(object, forKey: key)
    }

    /// 存储
    /// - Parameter int: 存储int型
    public func save(int: Int) {
        UserDefaults.standard.set(int, forKey: key)
    }

    /// 存储
    /// - Parameter float: 存储float型
    public func save(float: Float) {
        UserDefaults.standard.set(float, forKey: key)
    }

    /// 存储
    /// - Parameter double: 存储double型
    public func save(double: Double) {
        UserDefaults.standard.set(double, forKey: key)
    }

    /// 存储
    /// - Parameter bool: 存储bool型
    public func save(bool: Bool) {
        UserDefaults.standard.set(bool, forKey: key)
    }

    /// 存储
    /// - Parameter url: 存储url型
    public func save(url: URL?) {
        UserDefaults.standard.set(url, forKey: key)
    }

    /// 移除
    public func remove() {
        UserDefaults.standard.removeObject(forKey: key)
    }
}
```

上述协议主要是将`UserDefaults`的数据存取操作在协议中定义出来，并给出了协议默认方法实现。在取值的方法上借鉴了`SwiftyJSON`的思想，为每种基本结构提供可选值及非可选值两种方式，在使用时可根据自己的使用场景灵活使用。

我们如何进行使用呢？见下方代码示例，相关说明见注释。

```swift
/// 定义枚举，统一管理 UserDefaults的所有key
enum UserInfoEnum: String {
    case name
    case age
}

extension UserInfoEnum: UserDefaultsProtocol {
    /// 存储key值，可增加前缀、后缀等
    var key: String {
        return "CoderStar_\(rawValue)" rawValue
    }

    /// UserDefaults示例，协议默认实现为 UserDefaults.standard
    /// 如果想存储在另外的plist文件中，便可以单独实现
    var userDefaults: UserDefaults {
        return UserDefaults(suiteName: "CoderStar") ?? UserDefaults.standard
    }
}


func test() {
   /// 存
   UserInfoEnum.age.save(int: 18)

   /// 取
   let name = UserInfoEnum.age.int
}

```

> 如果公众号看代码不方便，可以直接访问[UserDefaultsProtocol.swift](https://github.com/Coder-Star/LTXiOSUtils/blob/master/LTXiOSUtils/Classes/Util/UserDefaultsProtocol.swift)进行查看。

### @propertyWrapper

Swift 5.1 推出了为 SwiftUI 量身定做的`@propertyWrapper`关键字，翻译过来就是`属性包装器`，有点类似 java 中的元注解，它的推出其实可以简化很多属性的存储操作，使用场景比较丰富，用来管理`UserDefaults`只是其使用场景中的一种而已。

先上代码，相关说明请看代码注释。

```swift
@propertyWrapper
public struct UserDefaultWrapper<T> {
    let key: String
    let defaultValue: T
    let userDefaults: UserDefaults

    /// 构造函数
    /// - Parameters:
    ///   - key: 存储key值
    ///   - defaultValue: 当存储值不存在时返回的默认值
    public init(_ key: String, defaultValue: T, userDefaults: UserDefaults = UserDefaults.standard) {
        self.key = key
        self.defaultValue = defaultValue
        self.userDefaults = userDefaults
    }

    /// wrappedValue是@propertyWrapper必须需要实现的属性
    /// 当操作我们要包裹的属性时，其具体的set、get方法实际上走的都是wrappedValue的get、set方法
    public var wrappedValue: T {
        get {
            return userDefaults.object(forKey: key) as? T ?? defaultValue
        }
        set {
            userDefaults.setValue(newValue, forKey: key)
        }
    }
}

// MARK: - 使用示例

enum UserDefaultsConfig {
    /// 是否显示指引
    @UserDefaultWrapper("hadShownGuideView", defaultValue: false)
    static var hadShownGuideView: Bool

    /// 用户名称
    @UserDefaultWrapper("username", defaultValue: "")
    static var username: String

    /// 保存用户年龄
    @UserDefaultWrapper("age", defaultValue: nil)
    static var age: Int?
}

func test() {
  /// 存
  UserDefaultsConfig.hadShownGuideView = true
  /// 取
  let hadShownGuideView = UserDefaultsConfig.hadShownGuideView
}

```

## 最后

一定要更加努力呀！

Let's be CoderStar!
