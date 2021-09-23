---
title: Flutter积累
category:
  - Flutter
tags:
  - Flutter
date: 2020-11-17 22:00:46
---

### Flutter 四种工程类型

- Flutter Application: Flutter 应用，包含标准的 Dart 层与 Native 平台层
- Flutter Module: Flutter 与原生混合开发，用于混编到已有的 Android/iOS 工程内
- Flutter Plugin: Flutter 插件，包含 Dart 层与 Native 平台层的实现
- Flutter Package: 纯 Dart 组件，仅包含 Dart 层的实现，往往定义一些公共 Widget

## 错误积累

### Waiting for another flutter command to release the startup lock...

进入到你的 flutter sdk 目录中，然后找到 bin/cache/lockfile 文件，删除它即可。

### `The following assertion was thrown while applying parent data.:Incorrect use of ParentDataWidget.`

只要记住一点Expanded的上级控件一定是Column或者Row就好了

### 代码段

刘海屏屏幕适配，正常顶部20，刘海屏44，正常底部0，刘海34
配套Widget，SafeArea
```dart
final double topPadding = MediaQuery.of(context).padding.top;
final double bottomPadding = MediaQuery.of(context).padding.bottom;
```