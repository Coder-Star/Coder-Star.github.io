---
title: Flutter积累
category:
  - Flutter
tags:
  - Flutter
date: 2020-11-17 22:00:46
---

### Flutter四种工程类型
* Flutter Application: Flutter应用，包含标准的Dart层与Native平台层
* Flutter Module: Flutter与原生混合开发，用于混编到已有的Android/iOS工程内
* Flutter Plugin: Flutter插件，包含Dart层与Native平台层的实现
* Flutter Package: 纯Dart组件，仅包含Dart层的实现，往往定义一些公共Widget



## 错误积累

### Waiting for another flutter command to release the startup lock...

进入到你的flutter sdk目录中，然后找到bin/cache/lockfile文件，删除它即可。