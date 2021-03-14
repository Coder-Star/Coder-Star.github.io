---
title: 关于CF对象与NS对象
category:
  - iOS
  - OC
tags:
  - iOS
  - OC
  - Core Foundation
date: 2021-03-14 19:10:36
---

Core Foundation 是一组 C 语言接口（Core Graphics，Core Text 等），与 Foundation 提供的功能相同，只是 Foundation 提供的是 Objective-C 接口。

> 先说下出现这两个框架的历史原因，当年乔布斯被自己创办的公司驱逐后，成立了`NeXT Computer`公司，拥有`NeXTSTEP`操作系统，后来乔布斯回到苹果后，就出现了一个问题。如何让旧的系统（Mac OS 9）和 NeXTSTEP 合成为一个新系统，这就需要一个更为底层的核心库可以供 Mac Toolbox 和 OPENSTEP 双方调用。Core Foundation 就这么诞生了。其中 Foundation 对象是 NS 开头的原因也是由于 NeXTSTEP 系统。

Core Foundation 中的对象也是使用引用计数的方式管理内存，对应的方式为 CFRetain/CFRelease，但是不存在 CFReleaseAutoRelease 的方式。

在 MRC 环境下，CF 对象与 NS 对象是可以直接进行强制转换的。

```Objective-C
-(void)bridgeInMRC {
    // 将Foundation对象转换为Core Foundation对象，直接强制类型转换即可
    NSString *strOC1 = [NSString stringWithFormat:@"xxxxxx"];
    CFStringRef strC1 = (CFStringRef)strOC1;
    NSLog(@"%@ %@", strOC1, strC1);
    [strOC1 release];
    CFRelease(strC1);

    // 将Core Foundation对象转换为Foundation对象，直接强制类型转换即可
    CFStringRef strC2 = CFStringCreateWithCString(CFAllocatorGetDefault(), "12345678", kCFStringEncodingASCII);
    NSString *strOC2 = (NSString *)strC2;
    NSLog(@"%@ %@", strOC2, strC2);
    [strOC2 release];
    CFRelease(strC2);
}
```

在 ARC 环境下，ARC 仅管理 Objective-C 指针（retain、release、autorelease），不管理 Core Foundation 指针。在 ARC，CF 与 OC 之间的转化桥梁是\_\_bridge，有三种方式。

- \_\_bridge： 只做类型转换，不改变对象所有权，是我们最常用的转换符
- \_\_bridge_transfer：ARC 接管 管理内存
- \_\_bridge_retained：ARC 释放 内存管理

```Objective-C
// OC->CF __bridge
// 只是单纯地执行了类型转换，没有进行所有权的转移，也就是说，当aNSString对象被ARC释放的时候，aCFString也不能被使用了。
NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];
CFStringRef aCFString = (__bridge CFStringRef)aNSString;
(void)aCFString;

// CF->OC __bridge
// 需要人工CFRelease，否则，OC对象的指针释放后，对象引用计数仍为1，不会被销毁
CFStringRef aCFString = CFStringCreateWithCString(NULL, "test", kCFStringEncodingASCII);
NSString *aNSString = (__bridge NSString *)aCFString;
(void)aNSString;
CFRelease(aCFString);

// CF->OC __bridge_transfer
NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];
CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;
aNSString = (__bridge_transfer NSString *)aCFString;

// OC->CF __bridge_retained
NSString *aNSString = [[NSString alloc]initWithFormat:@"test"];
CFStringRef aCFString = (__bridge_retained CFStringRef) aNSString;
(void)aCFString;
//这时候，即使开启ARC，也需要手动执行CFRelease
CFRelease(aCFString);
```
