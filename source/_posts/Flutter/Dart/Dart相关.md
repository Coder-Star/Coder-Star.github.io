---
title: Dart相关
category:
  - Dart
tags:
  - Dart
date: 2020-12-04 22:24:27
---

## Dart 库相关

**导入库**

```dart
// 完全导入
import 'package:hello/hello.dart';

// 只导入了math库中的Random对象
import 'package:math' show Random;
导入math库但不导入math库种的Random对象
import 'package:math' hide Random;

// 将引入的库加上别名，避免变量名冲突
import 'package:math' as mymath;
```

**库拆分**

```dart
使用part的形式，库之间共享空间，即color_extension.dart中的私有内容，flutter_star_extension可以直接使用

// 子库 color_extension.dart
part of library flutter_star_extension;
// 子库 string_extension.dart
part of library flutter_star_extension;

// 总库
library flutter_star_extension;
part 'src/color_extension.dart';
part 'src/string_extension.dart';

```

**库导出**

```dart
// 当使用flutter_star
library flutter_star;
export 'flutter_star_extension.dart';
```

* flutter packages pub publish --dry-run  检查包运行是否正常
* flutter packages pub publish 发布包
* flutter packages pub publish --server=https://pub.dartlang.org 就是指定服务器发布
* flutter packages pub publish -v 显示发布过程中的信息