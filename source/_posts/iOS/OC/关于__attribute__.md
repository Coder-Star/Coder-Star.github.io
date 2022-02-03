---
title: 关于__attribute__
category:
  - iOS
  - OC
tags:
  - iOS
  - OC
date: 2021-11-08 21:57:18
---

## 前言

Hi Coder，我是 CoderStar！

__attribute__ 是 编译器提供的机制。

[Function-attribute](https://developer.arm.com/documentation/dui0774/k/Compiler-specific-Function--Variable--and-Type-Attributes/Function-attributes?lang=en)

## __attribute__((format(archetype, string-index, first-to-check)))

第一参数需要传递 archetype 指定是哪种风格，这里是 NSString；
string-index 指定传入函数的第几个参数是格式化字符串；
first-to-check 指定第一个可变参数所在的索引。

## __attribute__((noreturn))

标志该函数没有返回值，比如一些内部死循环等函数。

## __attribute__((availability))

## __attribute__((unused))

## __attribute__((constructor))

确保此函数在 在 main 函数被调用之前调用

## __attribute__((destructor))

确保此函数在 在 main 函数被调用之后调用

## __attribute__((cleanup))

可以用来实现 Swift 中  `defer` 关键字的效果，

[黑魔法__attribute__((cleanup))](http://blog.sunnyxx.com/2014/09/15/objc-attribute-cleanup/)

[ReactiveObjC](https://github.com/ReactiveCocoa/ReactiveObjC/blob/1af6617f007cae727dce48d2031cc1e66d06f04a/ReactiveObjC/extobjc/EXTScope.h#L31)

``` objective-c
#define onExit \
    rac_keywordify \
    __strong rac_cleanupBlock_t metamacro_concat(rac_exitBlock_, __LINE__) __attribute__((cleanup(rac_executeCleanupBlock), unused)) = ^
```

## __attribute__ used section

`#define BeeHiveDATA(sectname) __attribute((used, section("__DATA,"#sectname" ")))`

[BHAnnotation](https://github.com/alibaba/BeeHive/blob/master/BeeHive/BHAnnotation.h)

[SRouter](https://tannerjin.github.io/2019/11/04/SRouter/)

## 最后

要更加努力呀！

Let's be CoderStar!
