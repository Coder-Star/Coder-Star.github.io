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

## 前言

Hi Coder，我是 CoderStar！

在 MRC 时代，我们可能会经常用到`AutoreleasePool`来帮助我们管理内存，在 ARC 时代，一些内存管理的操作被编译器替代了，不用再去手动的`release`以及`autorelease`等操作了，但是`AutoreleasePool`仍然在背后默默发挥着作用，并且有些场景下我们还是需要显式用到它，今天我们就来聊一聊`AutoreleasePool`。

## 使用形式

在

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

```objective-c
// 这个poolSentinelObj其实就是哨兵对象
void *poolSentinelObj = objc_autoreleasePoolPush();
// 自动释放池作用域 {} 中的代码
objc_autoreleasepoolPop(poolSentinelObj);
```

主要就是一个类:AutoreleasePoolPage。两个函数: objc_autoreleasePoolPush()、objc_autoreleasePoolPop()。

AutoreleasePoolPage 本身一个栈结构，从

每一个 autoreleasepool 由若干个 autoreleasePoolPage 类以双向链表的形式组合而成。

objc_autoreleasePoolPush() 将被调用, runtime 会向当前的 AutoreleasePoolPage 中添加一个 nil 对象作为哨兵, 在{}中创建的对象会被依次记录到 AutoreleasePoolPage 的栈顶指针。

哨兵对象定义为：

`#define POOL_SENTINEL nil`

当运行完 @autoreleasepool{}时, objc_autoreleasePoolPop(哨兵) 将被调用, runtime 就会向 AutoreleasePoolPage 中记录的对象发送 release 消息直到遇到第一个哨兵的位置, 即完成了一次完整的运作。

每个 AutoreleasePoolPage 的大小设置成 4096 个字节呢？ 因为 4096 是虚拟内存一页的大小。

**在没有手动加 Autorelease Pool 的情况下，Autorelease 对象是在当前的 runloop 迭代结束时释放的，而它能够释放的原因是系统在每个 runloop 迭代中都加入了自动释放池 Push 和 Pop**。

程序启动后，系统在主线程 Runloop 中注册了两个 Observer，回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。两个 Observer 分别如下：

- 监测 Entry 事件，回调里会调用 push 创建自动释放池，order 优先级最高，为 `-214748364`，保证创建释放池发生在其他所有回调之前；
- 监测 BeforeWaiting 及 Exit 事件， BeforeWaiting(准备进入休眠) 时调用 \_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池。Exit(即将退出 Loop) 时调用 \_objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

**如果手动加了，则在作用域大括号结束时释放。**

子线程默认不会开启 Runloop，那出现 Autorelease 对象如何处理？不手动处理会内存泄漏吗？

子线程如果没有创建 Pool ，但是产生了 Autorelease 对象，就会调用 autoreleaseNoPage 方法。在这个方法中，会自动帮你创建一个 hotpage，也就是默认生成一个 AutoreleasePoolPage 来添加 autorelease 对象。pool 中的对象会等到线程销毁后得到释放。

## 关于 main 文件中的`autoreleasepool`

Xcode 11 前

```objective-c
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

Xcode 11

```objective-c
int main(int argc, char * argv[]) {
    NSString * appDelegateClassName;
    @autoreleasepool {
        // Setup code that might create autoreleased objects goes here.
        appDelegateClassName = NSStringFromClass([AppDelegate class]);
    }
    return UIApplicationMain(argc, argv, nil, appDelegateClassName);
}
```

main() 函数中的 @autoreleasepool 只是负责管理它的作用域中的 autorelease 对象。
在 Xcode11 之前，是将整个应用程序运行放在 @autoreleasepool 内，由于 RunLoop 的存在，要 return 即程序结束后 @autoreleasepool 作用域才会结束，这意味着程序结束后 main 函数中的 @autoreleasepool 中的 autorelease 对象才会释放。

在 Xcode 11 后，触发主线程 RunLoop 的 UIApplicationMain 函数放在了 @autoreleasepool 外面，这可以保证 @autoreleasepool 中的 autorelease 对象在程序启动后立即释放。正如新版本的 @autoreleasepool 中的注释所写 "Setup code that might create autoreleased objects goes here."

对于 CLI 程序而言更有用。

## 应用场景

autoreleasepool 核心作用就是降低内存使用峰值。

- 当你使用类似 for 循环这样的逻辑需要产生大量的 Autorelease 局部变量时，AutoreleasePool 无意是最佳的一种解决方案；
- 需要自己管理一个辅助线程时，当开辟常驻线程后，需要加线程加入 AutoreleasePool；

### 自动加释放池的地方

- 使用容器的 block 版本的枚举器时，内部会自动添加一个 AutoreleasePool：

```swift
[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
    // 这里被一个局部 @autoreleasepool 包围着
}];
```

当然，在普通 for 循环和 for in 循环中没有，所以，还是新版的 block 版本枚举器更加方便。for 循环中遍历产生大量 autorelease 变量时，就需要手加局部 AutoreleasePool。

- 使用 NSThread 的 detachNewThreadSelector:toTarget:withObject: 方法创建新线程时，新线程自动带有 autoreleasepool。
- iOS 8 之后，自定义 NSOperation 时 main 方法会自动被 autoreleasepool 包起来（但实际上好像没有就加，需要还需要测试下），之前是需要自己手动添加的。当然，系统的 NSBlockOperation 和 NSInvocationOperation 这种默认的 Operation ，一直是自动添加的。

## 什么对象会加入 autoreleasePool?

通过 objc_autoreleaseReturnValue 和 objc_retainAutoreleasedReturnValue 来判断是否需要加入 autoreleasePool（objc_retainAutoreleasedReturnValue 和 objc_autoreleaseReturnValue 成对出现时就不会注册到 autoreleasepool），这是编译器的优化。 [arc-runtime-objc-autoreleasereturnvalue]((https://clang.llvm.org/docs/AutomaticReferenceCounting.html#arc-runtime-objc-autoreleasereturnvalue))

- 编译器会检查方法名是否以 alloc, new, copy, mutableCopy 开始，如果不是则自动将返回值的对象注册到 autoreleasepool 中，比如一些类方法。
- iOS5 及之前的编译器，关键字 __weak 修饰的对象，会自动加入 autoreleasePool；iOS5 及之后的编译器，则直接调用的 release，不会加入 autoreleasePool。
- id 指针 (id *) 和对象指针（NSError *），会自动加上关键字 `__autorealeasing`，加入 autoreleasePool。


## 子线程中的自动释放池

子线程中原本是没有自动释放池的，但是如果有runloop或者autorelease对象的时候，就会自动的创建自动释放池。

## swift 还需不需要 autoreleasepool；

当然需要，如果使用 NSData、Data 等方式创建实例，

## 最后

要更加努力呀！

Let's be CoderStar!

- [自动释放池的前世今生 ---- 深入解析 autoreleasepool](https://draveness.me/autoreleasepool/)
- [iOS autoreleasePool原理总结](https://www.jianshu.com/p/20496cbb6dc3)
- [黑幕背后的 Autorelease](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)
- [autoreleasepool 探究](https://github.com/Yuan91/autoreleasepool)
- [Transitioning to ARC Release Notes](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226)
- [iOS - 聊聊 autorelease 和 @autoreleasepool](https://juejin.cn/post/6844904094503567368)