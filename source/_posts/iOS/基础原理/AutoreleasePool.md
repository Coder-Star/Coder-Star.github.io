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

> 下列源码为`Runtime objc`内代码，版本之间可能会有差异，但是大致原理应该是一致的。

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

`@autoreleasepool` 包裹的相关代码在编译时，编译器会自动把它编译为如下形式。

```objective-c
// 这个poolSentinelObj其实就是哨兵对象
void *poolSentinelObj = objc_autoreleasePoolPush();

// 自动释放池作用域 {} 中的代码

objc_autoreleasepoolPop(poolSentinelObj);
```

继而查看`objc_autoreleasePoolPush`以及`objc_autoreleasepoolPop`这两个函数的源码实现，如下：

```objective-c
void *objc_autoreleasePoolPush(void) {
    return AutoreleasePoolPage::push();
}

void objc_autoreleasePoolPop(void *ctxt) {
    AutoreleasePoolPage::pop(ctxt);
}
```

上面我们可以看到一个核心类，`AutoreleasePoolPage`。

### `AutoreleasePoolPage`结构

`AutoreleasePoolPage` 是一个 C++ 中的类，在 `NSObject.mm` 中的定义是这样的：

```objective-c
class AutoreleasePoolPage {
    magic_t const magic; // 对当前AutoreleasePoolPage 完整性的校验
    id *next;  // 指向下一个即将产生的autoreleased对象的存放位置（当next == begin()时，表示AutoreleasePoolPage为空；当next == end()时，表示AutoreleasePoolPage已满
    pthread_t const thread; // 当前线程
    AutoreleasePoolPage * const parent; // 指向父节点，第一个结点的 parent 值为 nil；
    AutoreleasePoolPage *child; // 后节点
    uint32_t const depth; // 代表深度，第一个page的depth为0，往后每递增一个page，depth会加1；
    uint32_t hiwat; //
};
```

从上述的结构可以知道，其实每一个`AutoreleasePool`都是以`AutoreleasePoolPage`为节点用双向链表的形式连接起来的。

每个 `AutoreleasePoolPage` 对象有 `4096` 字节的存储空间, 除了存放它自己的成员变量（56 个字节，每个占 8 个字节）外, 剩下的空间用来存储后面加入的 `autorelease` 对象。

> 为什么每个 AutoreleasePoolPage 的大小设置成 4096 个字节呢？ 因为 4096 是虚拟内存一页的大小。

![AutoreleasePool结构](../../../img/iOS/基础原理/AutoreleasePool/AutoreleasePool结构.png)

### 大致流程

- 当进入`@autoreleasepool`作用域时，`objc_autoreleasePoolPush` 方法被调用, `runtime` 会向当前的 `AutoreleasePoolPage` 中添加一个 nil 对象作为哨兵对象，并返回该哨兵对象的地址；
- 对象调用`autorelease`方法，会被加入到对应的的`AutoreleasePoolPage`中去，`next`指针类似一个游标，不断变化，记录位置。如果加入的对象超出一页的大小，便会自动加一个新页。
- 当离开`@autoreleasepool`作用域时，`objc_autoreleasePoolPop(哨兵对象地址)`方法被调用，其会从当前 page 的 next 指标的上一个元素开始查找, 直到最近一个哨兵对象, 依次向这个范围中的对象发送`release`消息；

> 因为哨兵对象的存在，自动释放池的嵌套也是满足的，不管是嵌套还是被嵌套的自动释放池，找自己对应的哨兵对象就行了。

下面看下具体源码流程分析。

### 源码流程分析

#### push 函数

```objective-c
// 哨兵对象定义
#define POOL_BOUNDARY nil

static inline void *push()
{
    id *dest;
    if (slowpath(DebugPoolAllocation)) {
        // Each autorelease pool starts on a new pool page.
        dest = autoreleaseNewPage(POOL_BOUNDARY);
    } else {
        //添加一个哨兵对象到自动释放池
        dest = autoreleaseFast(POOL_BOUNDARY);
    }
    ...
    return dest;
}

//向自动释放池中添加对象
static inline id *autoreleaseFast(id obj)
{
    //获取hotPage:当前正在使用的Page
    AutoreleasePoolPage *page = hotPage();
    //如果有page 并且 page没有被占满
    if (page && !page->full()) {
        //添加一个对象
        return page->add(obj);
    } else if (page) {
        //添加一个对象
        return autoreleaseFullPage(obj, page);
    } else {
        // 如果没有page,则创建一个page
        return autoreleaseNoPage(obj);
    }
}

//创建一个新的page,并将当前page->child指向新的page,将对象添加进去
id *autoreleaseFullPage(id obj, AutoreleasePoolPage *page)
{
    ...
    do {
        if (page->child) page = page->child;
        else page = new AutoreleasePoolPage(page);
    } while (page->full());

    setHotPage(page);
    return page->add(obj);
}

//创建一个新的page
id *autoreleaseNoPage(id obj)
{
    ...
    AutoreleasePoolPage *page = new AutoreleasePoolPage(nil);
    setHotPage(page);
    ...
    // Push the requested object or pool.
    return page->add(obj);
}
```

#### pop 函数

```objective-c
//查看源码发现pop函数最终会调用 releaseUntil
//调用顺序为pop->popPage->releaseUntil

//stop 的值即为最初push时返回的哨兵对象的地址.
void releaseUntil(id *stop)
{
    //循环依次向autorelease对象发送release消息
    while (this->next != stop) {
        //AutoreleasePoolPage 有cold和hot之分.hot是当前正在使用的,cold是没有使用的
        //获取当前正在使用的
        AutoreleasePoolPage *page = hotPage();

        //如果为空,通过parent指针指向它的父节点,并将父节点置为当前使用的page
        while (page->empty()) {
            page = page->parent;
            setHotPage(page);
        }

        page->unprotect();
        //获取当前Page next指针的上一个元素
        id obj = *--page->next;
        memset((void*)page->next, SCRIBBLE, sizeof(*page->next));
        page->protect();

        //从next的上一个元素开始,向上查找只要不是哨兵对象,就向其发送release消息
        if (obj != POOL_BOUNDARY) {
            objc_release(obj);
        }
    }

    setHotPage(this);
}
```

#### autorelease 函数

```objective-c
static inline id autorelease(id obj)
{
    ASSERT(obj);
    ASSERT(!obj->isTaggedPointer());
    //调用autoreleaseFast,添加到自动释放池中
    id *dest __unused = autoreleaseFast(obj);
    ASSERT(!dest  ||  dest == EMPTY_POOL_PLACEHOLDER  ||  *dest == obj);
    return obj;
}
```

### 执行类型

`AutoreleasePool`一般会包括两种执行类型：

- 主 RunLoop 自动加入的`AutoreleasePool`；
- 手动添加`AutoreleasePool`；

#### 主 `Runloop` 自动加入

主线程 `Runloop` 中注册了两个 `Observer`，回调都是 `_wrapRunLoopWithAutoreleasePoolHandler()`。两个 `Observer` 如下：

- 监测 `Entry` 事件，回调里自动创建自动释放池，`order`为 `-214748364`， 优先级最高，保证创建释放池发生在其他所有回调之前；
- 监测 `BeforeWaiting` 及 `Exit` 事件；
  - `BeforeWaiting`时调用 `_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池。
  - `Exit`时调用 `_objc_autoreleasePoolPop()` 来释放自动释放池。这个 Observer 的 `order` 是 `2147483647`，优先级最低，保证其释放池子发生在其他所有回调之后。

> 系统的自动释放池也并不总是在 `BeforeWaiting` 和 `Exit` 才释放，在处理完 `Timer` 和 `Source` 事件之后, 也可能会进行释放操作。

当然系统部分方法内部也自动添加了`AutoreleasePool`，比如：

- 使用容器的 `block` 版本的枚举器时，内部会自动添加一个 `AutoreleasePool`；

    ```objective-c
    [array enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
        // 这里被一个局部 @autoreleasepool 包围着
    }];
    ```

- 使用 `NSThread` 的 `detachNewThreadSelector:toTarget:withObject:`方法创建新线程时，新线程自动带有 `AutoreleasePool`；
- ...

#### 手动添加

如果手动加了，则在作用域大括号结束时释放；

## `main.m` 文件中的`@autoreleasepool`

main() 函数中的 @autoreleasepool 只是负责管理它的作用域中的 autorelease 对象。
在 Xcode11 之前，是将整个应用程序运行放在 @autoreleasepool 内，由于 RunLoop 的存在，要 return 即程序结束后 @autoreleasepool 作用域才会结束，这意味着程序结束后 main 函数中的 @autoreleasepool 中的 autorelease 对象才会释放。

在 Xcode 11 后，触发主线程 RunLoop 的 UIApplicationMain 函数放在了 @autoreleasepool 外面，这可以保证 @autoreleasepool 中的 autorelease 对象在程序启动后立即释放。正如新版本的 @autoreleasepool 中的注释所写 "Setup code that might create autoreleased objects goes here."

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

## 什么对象会加入 autoreleasePool?

通过 objc_autoreleaseReturnValue 和 objc_retainAutoreleasedReturnValue 来判断是否需要加入 autoreleasePool（objc_retainAutoreleasedReturnValue 和 objc_autoreleaseReturnValue 成对出现时就不会注册到 autoreleasepool），这是编译器的优化。 [arc-runtime-objc-autoreleasereturnvalue]((https://clang.llvm.org/docs/AutomaticReferenceCounting.html#arc-runtime-objc-autoreleasereturnvalue))

- 编译器会检查方法名是否以 alloc, new, copy, mutableCopy 开始，如果不是则自动将返回值的对象注册到 autoreleasepool 中，比如一些类方法。
- iOS 5 及之前的编译器，关键字 __weak 修饰的对象，会自动加入 autoreleasePool；iOS5 及之后的编译器，则直接调用的 release，不会加入 autoreleasePool。
- id 指针 (id *) 和对象指针（NSError *），会自动加上关键字 `__autorealeasing`，加入 autoreleasePool。

## 应用场景

### CLI 程序

### 大量遍历生成临时对象

autoreleasepool 核心作用就是降低内存使用峰值。

- 当你使用类似 for 循环这样的逻辑需要产生大量的 Autorelease 局部变量时，AutoreleasePool 无意是最佳的一种解决方案；

### 常驻线程

子线程默认不会开启 Runloop，

那出现 Autorelease 对象如何处理？不手动处理会内存泄漏吗？

子线程如果没有创建 Pool ，但是产生了 Autorelease 对象，就会调用 autoreleaseNoPage 方法。在这个方法中，会自动帮你创建一个 hotpage，也就是默认生成一个 AutoreleasePoolPage 来添加 autorelease 对象。pool 中的对象会等到线程销毁后得到释放。

需要自己管理一个辅助线程时，当开辟常驻线程后，需要在任务外包一层 `AutoreleasePool`；这里对子线程中的自动释放池扩展一下。

子线程中原本是没有自动释放池的，但是如果有`RunLoop`或者`autorelease`对象的时候，就会自动的创建自动释放池，正常情况下，释放池里面对象释放的时机是线程销毁时，如果是常驻线程，就容易导致线程中所有的`autorelease`对象都迟迟得不到释放，所以需要手动添加`AutoreleasePool`，让相关对象可以得到及时释放。

```swift
class KeepAliveThreadManager {
    private init() {}
    static let shared = KeepAliveThreadManager()

    private(set) var thread: Thread?

    /// 开启常驻线程
    public func start() {
        if thread != nil, thread!.isExecuting {
            return
        }
        thread = Thread {
            autoreleasepool {
                let currentRunLoop = RunLoop.current

                // 如果想要加对该RunLoop的状态观察，需要在获取后添加，而不是等到启动之后再添加，

                currentRunLoop.add(Port(), forMode: .common)
                currentRunLoop.run()
            }
        }
        thread?.start()
    }

    /// 关闭常驻线程
    public func end() {
        thread?.cancel()
        thread = nil
    }
}

class Test: NSObject {
    func test() {
        if let thread = KeepAliveThreadManager.shared.thread {
            perform(#selector(task), on: thread, with: nil, waitUntilDone: false)
        }
    }

    @objc
    func task() {
        /// 在任务外加一层 autoreleasepool
        autoreleasepool {

        }
    }
}

```

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
