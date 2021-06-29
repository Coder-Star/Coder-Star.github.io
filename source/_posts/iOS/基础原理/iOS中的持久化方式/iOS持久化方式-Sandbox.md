---
title: iOS持久化方式-Sandbox
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 持久化
date: 2021-02-24 22:43:50
---

## 沙盒目录

Sandbox 详细目录（真机目录：/private/var/mobile/Containers）

- Bundle：
  - Application
    - app 的 md5 标识为名的文件夹
      - MyApp.app
      - BundleMetadata.plist
- Data
  - Application
    - app 的 md5 标识为名的文件夹
      - Documents
      - Library
        - Application Support
        - Caches
        - Cookies
        - Preferences
        - Saved Application State
        - SplashBoard
        - WebKit：存储WKWebView持久化的一些东西，如IndexDB、LocalStorage、WebSQL等；
      - SystemData
      - tmp
  - InternalDaemon
  - PluginKitPlugin
  - System
- Shared
  - AppGroup：
    - app 的 md5 标识为名的文件夹
      - Library
        - Caches
        - Preferences：默认没有该目录，当创建 group 的 UserDefault 时会创建该目录，UserDefault对应 plist 的名称为 group 名称；
  - SystemGroup
- Staging
- Temp
- iCloud

关键沙盒路径解读

- Bundle/Application: 包含了所有的资源文件和和可执行文件，上架前经过数字签名，上架后不可修改。

- Documents: 保存应运行时生成的需要持久化的、重要的数据（比如用户下载的歌曲）。iTunes、iCloud会备份该目录。在此目录下不要保存从网络上下载的文件，否则app无法上架。用户可以通过文件分享分享该目录下的文件,建议保存你希望用户看得见的文件。
  > 在iOS11 以后新增了一个“文件”APP，集中管理iOS上应用内创建的文件，以及各个云盘服务中保存的文件。在iOS工程info.plist中设置Application supports iTunes file sharing 和 Supports opening documents in place这两个选项为YES（默认为NO），就可以将该应用的沙盒路径Documents文件暴露在“文件”APP中。

- Library/Application Support: 建议用来存储除用户数据相关以外的所有文件，如游戏的新关卡。在iTunes和iCloud备份时会备份该目录。

- Library/Caches: 保存应用运行时生成的需要持久化的数据，一般存储体积大、不需要备份的非重要数据（例如，网络请求的音视频与图片等的缓存）。需要程序员手动清除。iTunes、iCloud不会备份该目录；

- Library/Preference: 保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes、iCloud会备份该目录。通过UserDefaults生成的plist文件也会存储在该目录下，我们不应该直接在这里创建文件，而是只通过UserDefaults；

- Library/SplashBoard: 存储启动屏缓存，缓存文件格式为ktx，本质上就是图片，如果启动屏不生效的问题可以考虑从删除该路径下缓存这个点；

- tmp: 保存应用运行时产生的一些临时数据；应用程序退出、系统空间不够、手机重启等情况下都会自动清除该目录的数据。无需程序员手动清除；iTunes不会备份该目录。比如相机拍摄完成时的照片视频都会被暂时保存到这个路径。

- AppGroup：宿主程序与Extension数据共享区域，

## 操作方式

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
