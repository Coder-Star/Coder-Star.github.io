---
title: iOS面试题积累
date: 2019-10-16 14:29:09
categories: [iOS]
tags: [iOS]
---
# 1、Class与Struct之间的区别
说区别之前，先明确可选型其实是有默认值的，默认值为nil
* Class是引用类型，Struct是值类型;(引用类型与值类型区别其实可以与深拷贝与浅拷贝对应起来)
* Struct不能继承，Class可以继承
* Class需要自己定义构造器，而Struct不需要；(Struct默认生成的构造器必须包括所有成员参数，只有当所有参数都为可选型时，可直接不用传入参数直接简单构造) 举一反三：Class中的属性必须都有默认值，否则编译错误,可以通过声明时赋值或者构造器时赋值两种方式给属性设置默认值
* Struct改变其属性受修饰符let影响，不可改变，Class不受影响
* struct方法中需要修改自身property时(非init方法)，方法需要前缀修饰符 mutating