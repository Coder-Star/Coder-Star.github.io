---
title: iOS检查清单
date: 2020-04-24 15:35:40
categories: [iOS]
tags: [iOS]
---

## 工具

### 1、AppDelegate 解耦以及组件化通信

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

## 常用库

以 Swift 为主，都可以通过 pod 的方式引入项目中

- **Alamofire** 网络请求

- **SnapKit** 自动布局
- **MJRefresh** 上拉刷新、下拉加载(OC)
- **Moya** 网络抽象层
- **MBProgressHUD** 进度框(OC)
- **IQKeyboardManagerSwift** 键盘控制
- **SwiftyJSON** json 取值
- **SwifterSwift** swift 扩展库，很全，重量级
- **SwiftDate** 日期扩展库
- **R.swift** 资源映射，可以像 Android 一样使用资源文件
- **JXSegmentedView** 分页控制器 ViewPager
- **TZImagePickerController** 照片选择(OC)
- **SKPhotoBrowser** 图片浏览(OC)
- **Kingfisher** 异步加载图片，还有一个 OC 的，SDWebImage
- **ESTabBarController-swift** 底部 tabbar
- **UITableView+FDTemplateLayoutCell** 缓存 tableview 高度(OC)
- **DZNEmptyDataSet** 空数据背景
- **FSPagerView** 轮播控件
- **Hero** 动画
- **QQ_XGPush** 信鸽推送
- **Bugly** 错误上报
- **SwiftyUserDefaults** UserDefaults 存储
- **KeychainSwift** 钥匙串存储
- **JSBadgeView** 角标
- **AMPopTip** 文字气泡
- **BRPickerView** 选择器组件，时间、日期以及列表等(OC)

## 大神博客

### 1、https://casatwy.com/

casatwy 的博客，看他的博客及网上的相关资料，之前是安居客的架构师，后来去了天猫，但一直没找到他的真实名字，其他人在介绍他的时候也是直接说他的英文名 casatwy，只有一篇博客称呼他为田神，可能是姓田吧，不过这些都不重要，他带给我们 iOS 开发者的技术分享才是最重要的，我们在心中默默的敬佩就好了。因为他之前是架构师，所以他的博客中对于 iOS 的架构设计有很多分享，非常建议大家去看一看。博客中有《iOS 应用架构谈》系列文章。

### 2、https://onevcat.com/#blog

喵神的博客，为 Swift 技术在国内的推广做了很大贡献。著作有《Swifter-Swift 开发者必备 Tips(第四版)》

### 3、http://blog.cnbang.net/

JSPatch 作者 bang 的博客

### 4、http://yulingtianxia.com/

腾讯杨萧玉的博客，底层细节能深入到汇编的实现，令人惊叹

### 5、https://halfrost.com/

人称“霜神”冰霜的博客，冰霜的文章针对某个问题总是能有深入、全面的剖析。

### 6、https://blog.ibireme.com/

YYKit 作者的博客，很多文章如《iOS 保持界面流畅的技巧》《深入理解 RunLoop》等都有极高的实用价值

### 7、https://ming1016.github.io/

滴滴技术专家戴铭的博客，每篇都是精品。极客时间《iOS 开发高手课》和纸书《跟戴铭学 iOS 编程》作者

### 8、https://draveness.me/

被人称为灯塔的 Draveness 的博客。Draveness 涉猎很广，文章也很有深度

### 9、https://www.jianshu.com/u/96a14318a4de

iOS 程序犭袁，CYLTabBarController 作者，博客也是篇篇经典

## 学习资源

### 1、https://www.hangge.com/(航歌 - 做最好的开发者知识平台)

主要是 swift 在 iOS 开发过程中的用法以及使用优秀第三方库的示例 Demo，适合新手入门。

### 2、http://www.code4app.com/code.php（code4app）

轮子网站，提供搜索，不过资源个人感觉较少

### 3、http://code.cocoachina.com/（cocoachina）

轮子网站，虽然资源相对较多，但是主要都是 oc 为主，比较坑爹的是不提供搜索功能，而且瀑布流一直变动位置，真是无法理解站长这么做的意义，难道是遇到上一个无良的产品经理吗？

## iOS 开发效率工具
