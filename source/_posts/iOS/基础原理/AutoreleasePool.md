---
title: AutoreleasePool
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - AutoreleasePool
date: 2021-03-02 23:15:40
---

## 使用形式

```swift
// OC
@autoreleasepool {
// 生成自动释放对象
}

// swift
autoreleasepool {
// 生成自动释放对象
}
```

## 基本原理

autoreleasepool 包裹的相关代码在编译时，编译器会自动把它编译为如下形式。

```

void *context = objc_autoreleasepoolPush();
// {} 中的代码
objc_autoreleasepoolPop();

```

主要就是一个类:AutoreleasePoolPage。两个函数: objc_autoreleasePoolPush()、objc_autoreleasePoolPop()。

运作方式: 每一个 autoreleasepool 由若干个 autoreleasePoolPage 类以双向链表的形式组合而成, 当程序运行到 @autoreleasepool{时, objc_autoreleasePoolPush() 将被调用, runtime 会向当前的 AutoreleasePoolPage 中添加一个 nil 对象作为哨兵, 在{}中创建的对象会被依次记录到 AutoreleasePoolPage 的栈顶指针, 当运行完 @autoreleasepool{}时, objc_autoreleasePoolPop(哨兵) 将被调用, runtime 就会向 AutoreleasePoolPage 中记录的对象发送 release 消息直到哨兵的位置, 即完成了一次完整的运作。

**在没有手动加 Autorelease Pool 的情况下，Autorelease 对象是在当前的 runloop 迭代结束时释放的，而它能够释放的原因是系统在每个 runloop 迭代中都加入了自动释放池 Push 和 Pop**。

程序启动后，系统在主线程 Runloop 中注册了两个 Observer，回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。两个 Observer 分别如下：

- 监测 Entry 事件，回调里会调用 push 创建自动释放池，order 优先级最高，为 -214748364，保证创建释放池发生在其他所有回调之前；
- 监测 BeforeWaiting 及 Exit 事件， BeforeWaiting(准备进入休眠) 时调用 \_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池。Exit(即将退出 Loop) 时调用 \_objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

**如果手动加了，则在作用域大括号结束时释放。**



## 关于 main 文件中的`autoreleasepool`


Xcode 11前
```swift
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

Xcode 11
```swift
int main(int argc, char * argv[]) {
    NSString * appDelegateClassName;
    @autoreleasepool {
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}
```




main()函数中的@autoreleasepool只是负责管理它的作用域中的autorelease对象。
在Xcode11之前，是将整个应用程序运行放在@autoreleasepool内，由于RunLoop的存在，要return即程序结束后@autoreleasepool作用域才会结束，这意味着程序结束后main函数中的@autoreleasepool中的autorelease对象才会释放。
在 Xcode 11后，触发主线程RunLoop的UIApplicationMain函数放在了@autoreleasepool外面，这可以保证@autoreleasepool中的autorelease对象在程序启动后立即释放。正如新版本的@autoreleasepool中的注释所写 “Setup code that might create autoreleased objects goes here.”

对于GUI程序来说可以认为是摆设，对于命令行程序不是

## 应用场景

autoreleasepool 核心作用就是降低内存使用峰值。当你使用类似 for 循环这样的逻辑需要产生大量的中间变量时，Autorelease Pool 无意是最佳的一种解决方案；

使用容器的 block 版本的枚举器时，内部会自动添加一个 AutoreleasePool：

```swift
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    // 这里被一个局部 @autoreleasepool 包围着
}];
```

当然，在普通 for 循环和 for in 循环中没有，所以，还是新版的 block 版本枚举器更加方便。for 循环中遍历产生大量 autorelease 变量时，就需要手加局部 AutoreleasePool。
```
使用NSThread的detachNewThreadSelector:toTarget:withObject:方法创建新线程时，新线程自动带有autoreleasepool。





**其余场景**

- 文件的读写处理
- 数据的批量处理, 包括图像裁剪生成, 音视频编解码
- 线程中 block 的回调处理

-[黑幕背后的 Autorelease](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)


还有使用 NSOperation 时 main 方法要不要用 autoreleasepool 包起来，官方文档说 iOS 8 之后不需要了，系统会帮你做。但是实际在使用 addOperationWithBlock 时，operation 执行完，block 中的 autorelease 对象没有被释放，感觉是没有生效，配合着比较重的解码或者其他操作时，也导致过 OOM。现在在异步线程搞事情时，除了关注并发问题外，特别注意 autorelease 问题[破涕为笑]

之前遇到过一个 C++ 实现的 SDK，开了几个常驻线程，在线程中向上层 OC 回调数据，为了方便使用，会将 C++ 的 buffer 转成 NSData 再序列化成 OC 的 Protobuf 对象。SDK 中的异步线程没有显式创建 autoreleasepool 包起来，虽然回调中创建的 OC 对象最终还会进入 autoreleasepool（NoPage 逻辑），但线程销毁 pool 才释放，而线程是常驻的，导致细水长流，每次回调就泄漏一点点内存，最后 OOM[捂脸]


有两种情况生成的对象会加入到autoreleasepool中：

非alloc/new/copy/mutablecopy 开始的方式初始化时。
id的指针或对象的指针在没有显示指定时