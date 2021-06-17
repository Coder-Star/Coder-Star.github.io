---
title: iOS日常小坑总结
category:
  - iOS
tags:
  - iOS
  - 坑
date: 2021-05-27 14:51:08
---

- iOS 请求权限回调,第一次用户选择时回调会发生在非主线程,如果在回调中进行 UI 操作,请注意切换到主线程.
- Xib 设置设置颜色使用的色域是 Generic RGB，而 API 设置颜色以及吸色计等大部分都是使用的 sRGB，所以我们需要注意将 xib 的色域换成 sRGB
- WkWebview 字体过小，注入一段 js `var meta = document.createElement('meta'); meta.setAttribute('name', 'viewport'); meta.setAttribute('content', 'width=device-width'); document.getElementsByTagName('head')[0].appendChild(meta);`
