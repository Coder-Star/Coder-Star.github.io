---
title: runtime底层
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - runtime
date: 2021-02-23 23:33:51
---

## Method

```objective-c
struct objc_method {
    SEL _Nonnull method_name           OBJC2_UNAVAILABLE;
    char * _Nullable method_types      OBJC2_UNAVAILABLE;
    IMP _Nonnull method_imp            OBJC2_UNAVAILABLE;
}                                      OBJC2_UNAVAILABLE;
```

SEL、IMP 都是 Method 结构体的属性。

**SEL：方法的名称、标识**

**获取方式**

- sel_registerName
- #selector
- NSSelectorFromString()
- method_getName

**后三种底层全是 sel_registerName**

**Method：方法体类定义中的方法**

**获取方式**

- class_getInstanceMethod() //获取 Method，传入 SEL

**IMP：函数指针，指向方法实现的地址**

**获取方式**

- method_getImplementation //获取 IMP，传入 Method

## class
