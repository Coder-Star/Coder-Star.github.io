---
title: iOS中的持久化方式
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 持久化
date: 2021-02-24 22:43:50
---

## 总览

![iOS持久化方式.webp](../../../img/iOS持久化方式.webp)

- UserDefaults  
  这种方式本质上还是 plist 文件存储，只不过对操作数据进行了封装，使用上更加方便，其生成的 plist 文件放置在 Library/Preference，生成的 plist 文件为 包名.plist。存储的类型是有限制的，如果想存储自定义类型，如果转换成可存储的类型，可以被获取到，不安全，写入时最好进行加密；
- plist 文件  
  plist 文件是将某些特定的类，通过 XML 文件的方式保存在目录中，其中还包括 NSArray、NSMutableArray、NSDictionary、NSMutableDictionary、NSData、NSMutableData、NSString、NSMutableString; 可以利用其读写文件方法。(writeToFile 、 WithContentsOfFile)。这种方式不可以保存自定义对象，除非先序列化成 json 文件，再存入。
- keychain 钥匙串
  此种方式存储的信息不会随着 APP 的卸载还删除。很安全。
- 归档
  数据对象需要遵守 NSCoding 协议。缺点：只能一次性归档保存或者一次性解压。所以只能针对小量数据，对数据操作比较笨拙，如果想改动数据的某一个小部分，需要解压或者归档整个数据；（NSKeyedArchiver、NSKeyedUnarchiver）。这种方式可以存储自定义对象，如用户 Model 等。相对 UserDefaults 以及 plist 文件来说更安全一些，因为数据归档时候是进行加密的。
- 沙盒文件（Sandbox）
  应用沙盒机制：每个 iOS 应用都有自己的应用沙盒（文件系统目录），与其他文件系统隔离。每个应用必须在自己的沙盒里运行，其他应用不能访问该沙盒。

- 数据库
  - SQLite
  - CoreData
  - Realm

## UserDefaults

**关于 UserDefaults 中的 suitename**

在咱们平时使用 UserDefaults 时一般使用`UserDefaults.standard`这个方式这存储信息，通过这种方式实际上操作的是 APP 沙箱中 Library/Preferences 目录下的以 bundle id 命名的 plist 文件；如果一个 APP 使用了一些 SDK，这些 SDK 会或多或沙的使用该方式存储，这样就会带来一系列问题；

- 各个 SDK 需要保证设置数据 KEY 的唯一性，以防止存取冲突；
- plist 文件越来越大造成的读写效率问题；
- 无法便捷的清除由某一个 SDK 创建的 UserDefaults 数据；

针对上述问题，我们可以使用 UserDefaults 提供的另外一个方法，

```Swift
@available(iOS 7.0, *)
public init?(suiteName suitename: String?)
```

根据传入的 suitname 有四种情况

- 传入 nil；跟使用`UserDefaults.standard`效果相同
- 传入 bundle id；无效，返回 nil
- 传入 App Groups 配置中 Group ID；会操作 APP 的共享目录中创建的 plist 文件，方便跨 APP 或宿主 APP 与扩展应用之间共享数据；
- 传入其他值；操作的是沙箱的 Documents/Library/Preferences 目录下以 suitename 命名的 plist 文件

UserDefaults 底层也是使用 plist 文件，那和普通的 plist 文件读取有什么区别呢？

iOS 系统会自动帮你做 plist 文件的读入以及 Cache，根据情况会把你在内存中所做的操作同步到 plist 文件（UserDefaults 同步到内存是同步的，同步到 Database 是异步的， iOS 8 开始，会有一个常驻进程 cfprefsd 来负责同步）。本质上，我们是可以对 UserDefaults 的最终产物 plist 文件进行操作的，但这是有风险的，最好不要这么操作。

## 沙盒（Sandbox）

### 目录

Sandbox 详细目录（真机目录：/private/var/mobile/Containers）

- Bundle
  - Application
    - app 的 md5 标识
      - MyApp.app
      - BundleMetadata.plist
- Data
  - Application
    - app 的 md5 标识
      - Documents
      - Library
        - Application Support
        - Caches
        - Preference
      - SystemData
      - tmp
  - InternalDaemon
  - PluginKitPlugin
  - System
- Shared
  - AppGroup
    - app 的 md5 标识
      - Library
        - Caches
        - Preferences  
          当创建 group 的 UserDefault 时会创建该目录，默认没有，对应 plist 的名称为 group 名称
  - SystemGroup
- Staging
- Temp
- iCloud

```
  Application: 包含了所有的资源文件和和可执行文件，上架前经过数字签名，上架后不可修改。

  Documents: 保存应运行时生成的需要持久化的、重要的数据（比如用户下载的歌曲）。iTunes、iCloud会备份该目录。在此目录下不要保存从网络上下载的文件，否则app无法上架。用户可以通过文件分享分享该目录下的文件,建议保存你希望用户看得见的文件。
  Tip: 在iOS11 以后新增了一个“文件”APP，集中管理iOS上应用内创建的文件，以及各个云盘服务中保存的文件。在iOS工程info.plist中设置Application supports iTunes file sharing 和 Supports opening documents in place这两个选项为YES（默认为NO），就可以将该应用的沙盒路径Documents文件暴露在“文件”APP中。


  Library/Application Support: 建议用来存储除用户数据相关以外的所有文件，如游戏的新关卡。在iTunes和iCloud备份时会备份该目录。

  Library/Caches: 保存应用运行时生成的需要持久化的数据，一般存储体积大、不需要备份的非重要数据（例如，网络请求的音视频与图片等的缓存）。需要程序员手动清除。iTunes、iCloud不会备份该目录；

  Library/Preference: 保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes、iCloud会备份该目录。通过UserDefaults生成的plist文件也会存储在该目录下，我们不应该直接在这里创建文件，而是只通过UserDefaults。

  tmp: 保存应用运行时产生的一些临时数据；应用程序退出、系统空间不够、手机重启等情况下都会自动清除该目录的数据。无需程序员手动清除。iTunes不会备份该目录。比如相机拍摄完成时的照片视频都会被暂时保存到这个路径。
```

### 操作方式

**Data 目录下**

```swift
 // 沙盒主目录
let path = NSHomeDirectory()
// tmp目录
let tmpPath = NSTemporaryDirectory()

// Documtents目录
// Library目录 -> libraryDirectory
// Application Support目录 -> applicationSupportDirectory
// Caches目录 -> cachesDirectory
// Preferences目录 -> preferencePanesDirectory
let documtentsPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!

// 上面形式获取路径返回值为String，下面形式返回值为URL

let dataURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first?.appendingPathComponent("sqliteName").appendingPathExtension("sqlite")

```

**AppGroup 目录下**

```swift
let appGroupIdentifier = "group.com.star.LTXiOSUtils.extension"
let groupURL = FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: appgroupIdentifier)


// 这种方式创建的userDefaults是可以被共享的，其创建后对应的plist文件名称就是appGroupIdentifier
let userDefaults = UserDefaults(suiteName: appGroupIdentifier)

```

获取到路径后就可以通过文件写入、读取的方式操作沙盒了
可以操作的数据结构有：

- NSMutableArray、NSArray
- NSData、Data、NSMutableData
- String、NSString
- NSDictionary、NSMutableDictionary

以 NSDictionary 举例，其余类似

```swift
let dicPath = path.appendingPathComponent("dic").appendingPathExtension("plist")
let dic = ["姓名": "张三", "年龄": 24] as [String : Any]

// 写入
NSDictionary(dictionary: dic).write(to: dicPath, atomically: true)
// 读取
NSDictionary(contentsOf: dicPath)
```
