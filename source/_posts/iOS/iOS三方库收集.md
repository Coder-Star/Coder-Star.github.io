---
title: iOS 三方库收集
category:
  - iOS
tags:
  - iOS
  - 三方库
date: 2021-01-09 08:49:49
---

## 常用

- **Alamofire(Swift)、AFNetworking(OC)** 网络请求
- **SnapKit(Swift)、Masonry(OC)** 自动布局
- **Kingfisher(Swift)，SDWebImage(OC)** 加载图片
- **IQKeyboardManagerSwift(Swift)、IQKeyboardManager(OC)** 键盘控制

## 网络

- **Moya** 网络抽象层

## UI

- **MJRefresh(OC)** 上拉刷新、下拉加载
- **MBProgressHUD(OC)** 进度框

- **JXSegmentedView** 分页控制器，其作者有好几个 UI 库，都值得看看

- **富文本相关**
  - **YYText**
  - **ZSSRichTextEditor**
  - **TYAttributedLabel**

- **动画**
  - **Hero** 动画库
  - **Spring**
  - **EasyAnimation**
  - **PeekPop** 3D 动画
  - **Animations** 动画 Demo

- **DBSphereTagCloud** 类似 soul 首页星球的效果

## 工具

### AppDelegate 解耦以及组件化通信

**AppDelegate 解耦**
实现原理基本相同，都是利用协议、组合模式

- [FRDModuleManager](https://github.com/lincode/FRDModuleManager) 豆瓣的方案，OC
- [PluggableApplicationDelegate](https://github.com/fmo91/PluggableApplicationDelegate) swift
- [PluggableAppDelegate](https://github.com/pchelnikov/PluggableAppDelegate) swift，比 PluggableApplicationDelegate 多了 UNUserNotificationCenterDelegate 支持，并进行了文件拆分

**组件间通信**

- [JLRoutes](https://github.com/joeldev/JLRoutes)：现在 github 上 star 数最高的是 JLRoutes，但是其用不到的功能比较多，而且基于遍历查找 URL 效率比较低下，OC；
- [HHRouter](https://github.com/lightory/HHRouter)：布丁动画的方案，对 URl 的查找方式不再使用遍历，还是使用匹配的方式，效率较高，但是 HHRouter 耦合程序比较高，过度依赖 ViewController，OC;
- [MGJRouter](https://github.com/meili/MGJRouter)：蘑菇街的设计方案，根据 HHRouter 方案进行改写，MGJRouter 的功能比较简单，但是查找 URL 的效率比较高，也解决了 HHRouter 过度依赖 ViewController 的问题，OC；
- [LDBusMediator](https://github.com/Lede-Inc/LDBusMediator)：网易的设计方案，其逻辑跟 MGJRouter 类似，区别在于 MGJRouter 使用的是先注册的方式，而 LDBusMediator 使用的是后查询的方式，OC。
- [routable-ios](https://github.com/clayallsopp/routable-ios)：in-app 的方案；OC
- [URLNavigator](https://github.com/devxoul/URLNavigator) swift

**AppDelegate 解耦以及组件化通信**

- [BeeHive](https://github.com/alibaba/BeeHive) 阿里的 iOS 解耦框架，OC 语言，主要解决 AppDelegate 解耦以及组件之间通信

### JSON 解析

- **SwiftyJSON** json 取值
- **HandyJSON**

### 数据库

- **FMDB(OC)** SQLite 数据库框架
- **Realm** 移动端数据库
- **MMKV**


### 内存泄漏
- MLeaksFinder 找到内存泄漏的对象
- OOMDetector 手Q自研
- FBRetainCycleDetector 检测该对象有没有循环引用即可
- 

## 待定

- **SwifterSwift** swift 扩展库，很全，重量级
- **SwiftDate** 日期扩展库
- **R.swift** 资源映射，可以像 Android 一样使用资源文件

- **TZImagePickerController(OC)** 照片选择
- **SKPhotoBrowser(OC)** 图片浏览
- **ZLPhotoBrowser**  图片选择

- **ESTabBarController-swift** 底部 tabbar
- **UITableView+FDTemplateLayoutCell(OC)** 缓存 tableview 高度
- **DZNEmptyDataSet** 空数据背景
- **FSPagerView** 轮播控件
- **TPNS-iOS** 腾讯云推送
- **Bugly** 错误上报
- **SwiftyUserDefaults** UserDefaults 存储
- **KeychainSwift** 钥匙串存储
- **JSBadgeView** 角标
- **AMPopTip** 文字气泡
- **BRPickerView(OC)** 选择器组件，时间、日期以及列表等
- **QRCodeReader.swift** 二维码扫描
- **EFQRCode** 二维码生成
- **PromiseKit** 异步编程
- **AsyncDisplayKit** 异步 UI
- **ReactiveSwift** rx 语法
- **SwiftGen** 二维码扫描
- **EFQRCode** 二维码生成
- **MMWormhole** 进程间通信

## 调试

- **GDPerformanceView** 显示 FPS、CPU 等信息
