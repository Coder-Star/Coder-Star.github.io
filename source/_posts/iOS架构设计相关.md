---
title: iOS架构设计相关
date: 2019-10-12 14:36:41
categories: [iOS]
tags: [iOS]
---
本来是根据Casa Taloyum的相关文章 [文章链接](https://casatwy.com/) 总结出来的。

## 一、相关规范

1、ViewController结构
viewDidload里面只做addSubview的事情；然后在viewWillAppear里面做布局的事情；最后在viewDidAppear里面做Notification的监听之类的事情；至于属性的初始化，则交给getter去做。
顺序：

* life cycle
* Delegate方法实现
* event response (按钮以及手势的事件响应)
* 一些私有方法
* getters and setters方法（暂定）

## 二、架构设计

### 1、MVVM

![MVVM架构示意图.gif](/iOS架构设计相关/MVVM架构示意图.gif)
