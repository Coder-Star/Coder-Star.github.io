---
title: OC学习
category:
  - OC
tags:
  - OC
date: 2020-03-16 11:36:19
---

## 修饰符

- @property 自动生成属性 setter 和 getter 方法的声明，自动生成对应的实例变量(下划线+属性名)
- @synthesize 生成实例变量（开发人员可设置）及属性 setter 和 getter 方法的声明。
- @dynamic 实例变量，属性 setter 和 getter 方法由用户自己实现，不自动生成，

当属性是只读属性，但是重写了 getter 方法，系统不会自动生成成员变量。当属性可读可写，同时重写了 setter/getter 方法，系统不会为你自动生成成员变量，但是如果只重写其中一个，系统还是会自动生成。

@property 有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize 和 @dynamic 都没写，那么默认的就是@syntheszie var = \_var; （自动合成的）。正常情况下，我们都是使用自动合成的，一般用不上@synthesize 及@dynamic。

什么情况下需要使用@synthesize 呢？

- 修改生成的成员变量名字
- 手动添加了 setter/getter 方法
- 实现了带有 property 属性的 protocol

怎么用呢？

```
// .h文件
@property (nonatomic, readonly, unsafe_unretained) int i;

// .m文件
@synthesize i = __i; // __i为指定的实例变量，@synthesize i相当于 @synthesize i = i;
```

### @property 属性修饰符

**内存管理**

- retain 属性必须是 objc 对象，此时输入会增加对象的引用计数加 1，MRC 模式下使用
- strong 表示只要该属性一直指向某个对象，这个对象就不会被销毁，与 retain 效果一样，在 ARC 模式下使用
- copy 属性必须是 objc 对象，并遵守了 NSCopying 协议；在赋值时使用传入一份拷贝，拷贝工作由 copy 方法执行，将指向新的内存地址，常常用于(NSArray,NSDictionary,NSString)，释放旧对象
- assign 简单直接赋值，不更改索引计数，适用简单数据类型（如 NSInteger,double,bool）。也可以修改对象，但是对象释放后，指针地址还存在没有置为 nil，造成野指针
- weak 表示只是指向对象，不隐式发送 retain, 指向对象一旦被销毁就会自动 nil 化
- \_\_unsafe_unretained 功能几乎等同于 weak, 但是对象被销毁不会自动 nil 化， 成了野指针

> 当修饰可变类型的属性时，如 NSMutableArray、NSMutableDictionary、NSMutableString，用 strong。
> 当修饰不可变类型的属性时，如 NSArray、NSDictionary、NSString，用 copy。

**读写权限**

- readwrite 可读可写，会生成 getter 和 setter 方法
- readonly 是只能读取，只会生成 getter 方法，不会生成 setter 方法，在值不想被外界修改时使用

**原子性**

- nonatomic 非原子性，与 atomic 是相反的
- atomic 原子性，在多线程的环境下是有必要的，会在 setter 里加锁，保证线程安全, 效率降低，底层加锁的方式为 spinlock_t 自旋锁来实现的，在 iOS10 之后，底层换成了`os_unfair_lock`

> atomic 不是一定线程安全的，如果一个对象的改变不是直接调用 getter/setter 方法，而是直接对对象内部属性修改、字符串拼接、数组增加和移除元素等操作，就不能保证这个对象是线程安全的。

**允许为空，iOS9 新关键字，用于引用类型**

- nullable/\_Nullable/\_\_nullable 告诉编译器表示值可以空
- nonnull/\_\_nonnull/\_Nonnull 表示不可以为 nil，NULL， 不遵守规定编译器会警告
- null_resettable get 不能返回空, set 可以为空，如果使用，必须重写 set 或者 get 方法，处理传递值为空的情况
- \_Null_unspecified 不确定是否为空

```
// nonnull
@property (nonatomic, copy, nonnull) NSString *name;
@property (nonatomic, copy) NSString * _Nonnull name;
@property (nonatomic, copy) NSString * __nonnull name;
// 方法时使用
- (nonnull NSString *)test:(nonnull NSString *)name;
- (NSString * _Nonnull)test1:(NSString * _Nonnull)name;

// nullable
@property (nonatomic, copy, nullable) NSString *name;
@property (nonatomic, copy) NSString *_Nullable name;
@property (nonatomic, copy) NSString *__nullable name;

// null_resettable
@property (nonatomic, copy, null_resettable) NSString *name;

// _Null_unspecified
@property (nonatomic, strong) NSString *_Null_unspecified name;
@property (nonatomic, strong) NSString *__null_unspecified name;

```

关于 ARC 下，不显示指定属性关键字时，默认关键字：

- 基本数据类型：atomic readwrite assign
- 普通 OC 对象： atomic readwrite strong
