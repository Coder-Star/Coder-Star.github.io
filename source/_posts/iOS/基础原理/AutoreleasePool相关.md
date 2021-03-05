---
title: AutoreleasePool相关
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - AutoreleasePool
date: 2021-03-02 23:15:40
---

### 使用形式

```
// OC
@autoreleasepool {
// 生成自动释放对象
}

// swift
autoreleasepool {
// 生成自动释放对象
}

```

### 基本原理

autoreleasepool 包裹的相关代码在编译时，编译器会自动把它编译为如下形式。

```

void \*context = objc_autoreleasepoolPush();
// {} 中的代码
objc_autoreleasepoolPop();

```

主要就是一个类:AutoreleasePoolPage。两个函数: objc_autoreleasePoolPush()、objc_autoreleasePoolPop()。

运作方式: autoreleasepool 由若干个 autoreleasePoolPage 类以双向链表的形式组合而成, 当程序运行到@autoreleasepool{时, objc_autoreleasePoolPush()将被调用, runtime 会向当前的 AutoreleasePoolPage 中添加一个 nil 对象作为哨兵,在{}中创建的对象会被依次记录到 AutoreleasePoolPage 的栈顶指针,当运行完@autoreleasepool{}时, objc_autoreleasePoolPop(哨兵)将被调用, runtime 就会向 AutoreleasePoolPage 中记录的对象发送 release 消息直到哨兵的位置, 即完成了一次完整的运作。

在没有手动加 Autorelease Pool 的情况下，Autorelease 对象是在当前的 runloop 迭代结束时释放的，而它能够释放的原因是系统在每个 runloop 迭代中都加入了自动释放池 Push 和 Pop。程序启动后，系统在主线程 Runloop 中注册了两个 Observer，回调都是\_wrapRunLoopWithAutoreleasePoolHandler()。两个 Observer 分别如下：

- 监测 Entry 事件，回调里会调用 push 创建自动释放池，order 优先级最高，为-214748364，保证创建释放池发生在其他所有回调之前；
- 监测 BeforeWaiting 及 Exit 事件， BeforeWaiting(准备进入休眠) 时调用\_objc_autoreleasePoolPop() 和 \_objc_autoreleasePoolPush() 释放旧的池并创建新池。Exit(即将退出 Loop) 时调用 \_objc_autoreleasePoolPop() 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

在 Runloop 的 Entry 时调用 push 方法

如果手动加了，则在作用域大括号结束时释放。

main 函数中的 autoreleasepool 目前根据实际情况好像没什么用。

### 应用场景

```
autoreleasepool 核心作用就是降低内存使用峰值。当你使用类似 for 循环这样的逻辑需要产生大量的中间变量时，Autorelease Pool 无意是最佳的一种解决方案；

使用容器的 block 版本的枚举器时，内部会自动添加一个 AutoreleasePool：

```

[array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
// 这里被一个局部@autoreleasepool 包围着
}];
//

```

当然，在普通 for 循环和 for in 循环中没有，所以，还是新版的 block 版本枚举器更加方便。for 循环中遍历产生大量 autorelease 变量时，就需要手加局部 AutoreleasePool。
```

**其余场景**

- 文件的读写处理
- 数据的批量处理, 包括图像裁剪生成, 音视频编解码
- 线程中 block 的回调处理
