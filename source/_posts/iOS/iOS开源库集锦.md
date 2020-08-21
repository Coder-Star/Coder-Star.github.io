---
title: iOS开源库集锦
date: 2020-04-24 15:35:40
categories: [iOS]
tags: [iOS,开源库]
---
## 一、UI


## 二、工具

### 1、AppDelegate解耦以及组件化通信
**AppDelegate解耦**  
实现原理基本相同，都是利用协议、组合模式
* [FRDModuleManager](https://github.com/lincode/FRDModuleManager) 豆瓣的方案，OC
* [PluggableApplicationDelegate](https://github.com/fmo91/PluggableApplicationDelegate) swift
* [PluggableAppDelegate](https://github.com/pchelnikov/PluggableAppDelegate) swift，比PluggableApplicationDelegate多了UNUserNotificationCenterDelegate支持，并进行了文件拆分

**组件间通信**
* [JLRoutes](https://github.com/joeldev/JLRoutes)：现在github上star数最高的是JLRoutes，但是其用不到的功能比较多，而且基于遍历查找URL效率比较低下，OC；
* [HHRouter](https://github.com/lightory/HHRouter)：布丁动画的方案，对URl的查找方式不再使用遍历，还是使用匹配的方式，效率较高，但是HHRouter耦合程序比较高，过度依赖ViewController，OC;
* [MGJRouter](https://github.com/meili/MGJRouter)：蘑菇街的设计方案，根据HHRouter方案进行改写，MGJRouter的功能比较简单，但是查找URL的效率比较高，也解决了HHRouter过度依赖ViewController的问题，OC；
* [LDBusMediator](https://github.com/Lede-Inc/LDBusMediator)：网易的设计方案，其逻辑跟MGJRouter类似，区别在于MGJRouter使用的是先注册的方式，而LDBusMediator使用的是后查询的方式，OC。
* [routable-ios](https://github.com/clayallsopp/routable-ios)：in-app的方案；OC
* [URLNavigator](https://github.com/devxoul/URLNavigator) swift

**AppDelegate解耦以及组件化通信**
* [BeeHive](https://github.com/alibaba/BeeHive) 阿里的iOS解耦框架，OC语言，主要解决AppDelegate解耦以及组件之间通信