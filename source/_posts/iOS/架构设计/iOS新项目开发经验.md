---
title: iOS新项目开发经验
category:
  - iOS
tags:
  - iOS
  - 新项目
date: 2020-11-23 10:51:36
---

## 样式统一

建立相关常量，或者工厂模式产生统一 View，目的是使 APP 中 UI 样式统一；

常量包括：

- 文字（颜色、字号、字体）
- 颜色（多主题色）
- 圆角
- 边框（宽度、颜色）

## 代码规范

- 引入代码规范管理插件

## 代码管理

- 使用 git 管理，编写.gitignore


## 相关规范

1、ViewController 结构
viewDidload 里面只做 addSubview 的事情；然后在 viewWillAppear 里面做布局的事情；最后在 viewDidAppear 里面做 Notification 的监听之类的事情；
顺序：

- life cycle
- Delegate 方法实现
- event response (按钮以及手势的事件响应)
- 一些私有方法
