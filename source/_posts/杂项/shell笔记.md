---
title: Shell 笔记
category:
  - 杂项
tags:
  - shell
date: 2021-05-27 20:59:49
---

## shell 基础

$() 与``都是用来做命令替换的，当需要获取命令返回的结果时，比较常用，如 RESULT=`cat main.swift`，就可以将 main.swift 中的内容作为字符串赋值给变量 RESULT。

`$`与`${}`都是用来获取变量内容的

`$?` 来获取上条代码返回值，一般退出状态非 0 表示执行出错

判断整数

```
-eq //equals等于
-ne //no equals不等于
-gt //greater than 大于
-lt //less than小于
-ge //greater equals大于等于
-le //less equals小于等于
```

文件描述符是与打开文件或者数据流相关联的整数，0、1、2 是系统保留的三个文件描述符，分别对应标准输入（stdin）、标准输出（stdout）、标准错误（stderr）。Linux Shell 使用 " > " ">>" 进行对文件描述符进行重定位。

`RESULT=$(git rev-parse --is-inside-work-tree 2>&1)`

`2>&1`  将原先标准错误调整到标准输出

## 代码段

判断上行代码是否执行成功，如果命令执行成功，则返回 0，否则返回 1

```shell
# 上一行命令执行结果不为0，表示执行失败
if [ $? -ne 0 ]; then
fi
```

判断文件是否存在

```shell
if [ -f "$IPA_FILE" ] ; then
fi
```

获取当前路径

```shell
CURRENT_PATH=$(cd "$(dirname "$0")";pwd)

CURRENT_PATH=$(dirname $(readlink -f $0 ))
```

## sed 命令

unix 与 linux 在执行 sed 指令时有些许区别，当使用 sed -i 命令时，需要在 -i 指令后面多加一段 "", 即 `sed -i "" "s/192.168.0.2/192.168.0.3/g" filePath`

## curl 命令

## shell 好用工具

- jq：shell 下处理 json 神器，mac 安装命令 `brew install jq`
