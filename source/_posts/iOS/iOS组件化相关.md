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

![iOS架构设计](../img/iOS架构设计.png)

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

#### MGJRouter及ModuleManager

MGJRouter本身是一个单例模式，其实是在路由中心中维护了一张路由表（字典），其中url为key，value为Block；注册时，将key以及value保存到路由表中，使用时，根据Url拿到对应的Block进行执行。

**优点**

* 项目简洁，就两个文件；
* 可以协调本地调用、远程推送调用，甚至可以达到同一个url管理三端的效果；（iOS，Android）

**缺点**

* 传值比较麻烦；传值一般有两种方式，一种是直接在url上拼接，一种是通过提供的字典进行传值，对于普通的数据类型还可以接受，但是想要传递类似UIImage以及Data这种对象就比较难了。
* 参数传递以及接收解析时存在hardcode的问题；

为了解决MGJRouter本身的传值麻烦问题以及部分不适合用URL传值的方式，就可以通过注册protocol的方式来实现。核心代码如下。
```swift
ModuleManager.register(className: AnyObject, forProtocol: Protocol)
```

**记录在使用MGJRouter过程中遇到的问题**

* 如果url中含有中文，想要使用canOpenURL这个方法，需要对url进行编码，否则会引起代码Crash，因为其函数实现未对url进行编码；这一点已经在MGJRouter提交了pr，不过根据上次pr还没有处理的情况来看，这个库很有可能不再进行维护了；网上也有很多这个库的swift翻译版本；
  
#### 2、Target-Action方案

**[CTMediator](https://github.com/casatwy/CTMediator)**

CTMediator是casatwy提出的Target-Action（如果不知道什么意思的自行google）方案的实现。其与url方法相比最大的特点就在于不需要去维护路由表以及路由方案存在的hard code问题；

大致思想如下：（假定B模块需要与A模块通信）  

A模块在开发完成之后需要对外提供基于Target-Action的调用方式，根据框架源码的阅读，需要给我们提供的类以及方法加上指定前缀，其中，类名前面需要加上Target_，方法名前面需要加上Action_。这部分代码我们可以封装成一个单独的pod（起名为A)。理论上我们做完这部分工作后就可以通过CTMediator调用模块，但是调用模块时需要进行一定的硬编码，如类名、方法名以及方法参数等等，这样在程序编译时进行检查；于是我们可以进行下一步操作；
```swift
// Target对象必须要继承自NSObject，因为框架中生成的类实例是以NSObject接收的
// Action方法必须带@objc前缀
class Target_A: NSObject {
    // 方法参数只允许有一个，并且类型为NSDictionary，Action方法第一个参数不能有Argument Label
    @objc func Action_Extension_ViewController(_ params:NSDictionary) -> UIViewController {
        if let callback = params["callback"] as? (String) -> Void {
            callback("success")
        }

        let aViewController = ViewController()
        return aViewController
    }
}
``` 
我们可以通过给CTMediator类（单例）提供扩展的方式，将硬编码的部分写入这部分中，这样外部调用时就可以知道我们需要什么参数了，这部分工作也是由编码A模块的人提供；可以也将该部分单独作为一个pod（起名A_Extension），这里解释下这部分代码不与之前A模块的业务代码放入一个pod的好处：
* A模块业务代码不需要集成CTMediator；
* 团队合作的时候A模块的开发方早期不用提供A模块代码，先只提供A_Extension这个pod包即可，类似先定义接口，后期进行实现；
```swift
public extension CTMediator {
    @objc func A_showSwift(callback:@escaping (String) -> Void) -> UIViewController? {
        let params = [
            "callback":callback,
            kCTMediatorParamsKeySwiftTargetModuleName:"A_swift"
            ] as [AnyHashable : Any]
        if let viewController = self.performTarget("A", action: "Extension_ViewController", params: params, shouldCacheTarget: false) as? UIViewController {
            return viewController
        }
        return nil
    }
}
```


#### 3、完整的组件解耦以及通信方案

**[BeeHive](https://github.com/alibaba/BeeHive)**

BeeHive是阿里开源的一个APP模块化编程框架的实现方案，其吸收了Spring框架Service的理念来实现模块间的API解构，路由方面，本质上与蘑菇街后来推出的Protocol方案类似，也加入了url Router的方式，不过readme.md没有对这种方式进行体现，但是在源码中可以看出；其中这种Protocol方法在思想上跟spring的service注入类似；

这种方式决定了业务模块需要引入公共的协议；

在这个框架中，使用了注解方式进行注册的设计很有意思；

**提供的主要功能**

* 模块解耦：通过提前注册（注册方式多种），解决不同模块监听声明周期的问题； 
* 模块间调用解耦：通过提前注册的方式（注册方式多种），将协议与协议的实现类绑定起来，解决模块间的耦合问题，提供更加清晰的调用方案。
* 其他开发支持功能
  


### 四、总结



### 相关资料整理

* [蘑菇街App的组件化之路](https://www.tuicool.com/articles/vyeUf2J)
* [蘑菇街App的组件化之路·续](https://www.tuicool.com/articles/QneYvmi)
* [iOS应用架构谈组件化方案](https://casatwy.com/iOS-Modulization.html)
* [iOS组件化方案探索](http://blog.cnbang.net/tech/3080/)
* [BeeHive，一次iOS模块化解耦实践](https://mp.weixin.qq.com/s?__biz=MzUxMzcxMzE5Ng==&mid=2247488305&idx=1&sn=0f0a2e4d5febe3024adf0578f092b020&source=41#wechat_redirect)
* [BeeHive框架全面解析——iOS开发主流方案比较](https://xiaozhuanlan.com/topic/4052613897)
* [BeeHive —— 一个优雅但还在完善中的解耦框架](https://www.jianshu.com/p/24f6299ebe82)