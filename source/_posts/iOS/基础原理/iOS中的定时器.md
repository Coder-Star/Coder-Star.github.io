---
title: iOS中的定时器
category:
  - iOS
  - 定时器
tags:
  - iOS
  - 定时
date: 2021-02-15 10:42:29
---

## Timer

Timer的创建分别实例方法以及类方法，还分为Block以及Target-Action两种形式，共四种方式。其中Block方式自动解决了循环引用，不需手动处理，类方法创建的Timer自动执行，不需要手动加入到Runloop，而实例方法需要手动加入到Runloop。

如果是一次性调用的NSTimer，会在本次调用完毕之后invalidate掉NSTimer自身，而NSTimer做retain的对象也会被进行一次release。但是如果是多次重复调用的NSTimer，就需要我们自己在某个特定的时刻来invalidate掉NSTimer，这个invalidate的时刻是根据我们代码情况来自己决定的，否则将会一直存在。

timer 循环引用出现的原因很多人会说是 timer 所在的类与类之间存在循环引用，但是如果将 timer 不设置为类的一个属性，而是在一个方法里面定义，这样类便不会持有 timer，但是你你发现 timer 和类还是不能被释放，其实主要原因是 timer 在创建时需要加入到 Runloop 中，Runloop 不退出一直持有 timer，timer 又以 target 的形式强引用类，则导致都不销毁的问题。

解决方法：
- 使用 Block 的形式创建定时器，使 timer 不再强引用所在类；
- 使用中间类来解决类与 timer 之间的强应用，timer 强引用中间类，中间类弱引用 timer 所在类，可以使用 NSProxy(仅OC可以使用，Swift不可使用)；
- 如果类是 ViewController，可以在页面出现时启动定时器，消失时关闭定时器，前提是你的需求可以允许这么做；

Timer的实时性是不准确的，会存在延迟，这与timer所在的线程有很大关系，当线程在进行大量计算时，这期间有可能会错过很多次NSTimer的循环周期，但是NSTimer并不会将前面错过的执行次数在后面都执行一遍，而是继续执行后面的循环，也就是在一个循环周期内只会执行一次循环。无论循环延迟的多离谱，循环间隔都不会发生变化，在进行完大数据处理之后，有可能会立即执行一次NSTimer循环，但是后面的循环间隔始终和第一次添加循环时的间隔相同。

## GCD


## 