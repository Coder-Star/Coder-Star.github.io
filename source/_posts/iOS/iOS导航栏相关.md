---
title: iOS导航栏相关
date: 2019-10-19 14:41:32
categories: [iOS]
tags: [iOS]
---

## 1、注意事项

1. rightBarButtonItems在设置按钮时，导航栏布局顺序是从右到左；leftBarButtonItems在设置按钮时，导航栏布局顺序是从左到右；
2. 使用rightBarButtonItem来设置按钮时，实际上改变的是rightBarButtonItems数组中的第一个元素，最终页面的按钮数据来源还是来自rightBarButtonItems；leftBarButtonItems同理。