---
title: iOS组件化相关
date: 2020-04-10 19:12:43
categories: [iOS]
tags: [iOS,组件化]
---

### 一、前言

APP组件化的过程我觉得主要分为下面几个部分。

* **APP架构设计**  
    我们在将APP进行组件化搭建之前，我们首先得有一个整体的架构设计，哪些功能下沉，哪些功能归为一块，功能的颗粒度怎么设计，这都是我们需要考虑的问题。当然，架构肯定需要根据我们的业务来设计，脱离业务需求的架构肯定不是好架构。
* **APP组件之间的通信**  
    将APP拆分成组件之后，我们就要考虑组件之间的通信问题。其实组件之间的通信问题说到底就是APP中的路由方案设计问题。


### 二、架构设计、组件拆分

![iOS架构设计](/iOS组件化相关/iOS架构设计.png)

#### 1、基础支撑层

#### 2、基础模块及UI组件层

#### 3、业务模块层

#### 4、APP应用层

如果项目业务不涉及到系列APP，这一层可以忽略；

### 三、组件间通信(路由方案)
**路由解决问题**
* 点击推送消息进入到指定页面
* 3Dtouch快捷方式
* 系列APP之间页面互相调用、跳转
* 组件与组件之前的互相调用
* ...

目前市面上主要流行的组件间通信方案主要是以蘑菇街为主的URL/Protocol方案和反革命casatwy提出的Target-Action方案。

#### 1、URL/Protocol方案

URL/Protocol方案

#### 相关库
**OC库**
* [JLRoutes](https://github.com/joeldev/JLRoutes)：现在github上star数最高的是JLRoutes，但是其用不到的功能比较多，而且基于遍历查找URL效率比较低下；
* [HHRouter](https://github.com/lightory/HHRouter)：布丁动画的方案，对URl的查找方式不再使用遍历，还是使用匹配的方式，效率较高，但是HHRouter耦合程序比较高，过度依赖ViewController;
* [MGJRouter](https://github.com/meili/MGJRouter)：蘑菇街的设计方案，根据HHRouter方案进行改写，MGJRouter的功能比较简单，但是查找URL的效率比较高，也解决了HHRouter过度依赖ViewController的问题；
* [LDBusMediator](https://github.com/Lede-Inc/LDBusMediator)：网易的设计方案，其逻辑跟MGJRouter类似，区别在于MGJRouter使用的是先注册的方式，而LDBusMediator使用的是后查询的方式。
* [routable-ios](https://github.com/clayallsopp/routable-ios)：in-app的方案；

**swift库**
* [URLNavigator](https://github.com/devxoul/URLNavigator)

#### MGJRouter

记录在使用MGJRouter过程中遇到的问题

* 如果url中含有中文，想要使用canOpenURL这个方法，需要对url进行编码，否则会引起代码Crash；
  
#### 2、Target-Action方案

* [CTMediator](https://github.com/casatwy/CTMediator)




### 四、总结



### 相关资料整理

* [蘑菇街App的组件化之路](https://www.tuicool.com/articles/vyeUf2J)
* [蘑菇街App的组件化之路·续](https://www.tuicool.com/articles/QneYvmi)
* [iOS应用架构谈组件化方案](https://casatwy.com/iOS-Modulization.html)
* [iOS组件化方案探索](http://blog.cnbang.net/tech/3080/)