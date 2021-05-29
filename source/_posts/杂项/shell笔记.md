---
title: shell笔记
category:
  - 杂项
tags:
  - shell
date: 2021-05-27 20:59:49
---

这几天因为要搭建公司的CI平台，需要用到shell进行打包，所以简单了解了一下shell。

## shell基础

```
$()与``都是用来做命令替换的，当需要获取命令返回的结果时，比较常用，如RESULT=`cat main.swift`，就可以将main.swift中的内容作为字符串赋值给变量RESULT。

$与${}都是
```

## sed命令

unix与linux在执行sed指令时有些许区别，当使用sed -i 命令时，需要在-i指令后面多加一段 "", 即 `sed -i "" "s/192.168.0.2/192.168.0.3/g" filePath`

## curl命令


