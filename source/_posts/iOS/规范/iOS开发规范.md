---
title: iOS开发规范
date: 2020-08-31 10:50:18
categories: 
  - iOS
  - 规范
tags: [Swift, iOS, 规范]
---

## 语法规范

- [Swift 开发规范](../Swift开发规范)
- [Objective-C开发规范](../Objective-C开发规范)

## 资源文件命名及使用

- 资源文件命名单词之间使用下划线`_`连接，单词小写，规则如下：
  `模块名_业务功能描述_控件描述_控件状态限定词`，如`home_message_btn_normal`;
  ```
  控件描述包括 背景：bg；图标：icon；按钮：btn；图片：img；等
  控件状态包括 正常：normal；按下：pressed；选中：selected；禁用：disabled；高亮：highlighted；等
  ```
- 尽量使用 xcassets 管理图片资源，根据实际情况创建一个或多个 xcassets 文件，在 xcassets 内部也可以创建多个文件夹进行图片资源管理分组，其中文件夹名称使用 UpperCamelCase 形式，根据实际情况考虑，可不再放入1x的图片；
- png 图片使用 TinyPNG 或者类似工具压缩处理，减少包体积；
- 图片资源推荐使用 png 格式
- 颜色、字体等样式放在一处统一管理


## 基本组件

## UI 相关

## 数据存储

- UserDefault 不存储大容量数据，如果存储大容量数据，尽量使用数据库存储
- UserDefault 存储数据时，考虑数据安全性，决定是否将数据加密

## 安全

- 禁止使用自带的`print()`方法打印 Log，需进行封装，控制生产环境下不可输出调试 Log

## 其他