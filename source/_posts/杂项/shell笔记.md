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
# 通过 readlink 获取绝对路径，再取出目录
CURRENT_PATH=$(dirname $(readlink -f $0 ))

CURRENT_PATH=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )

# 这个是有缺陷的，比如脚本A source了另一个目录下的脚本B、然后脚本B尝试使用此法获取路径时得到的是A的路径
CURRENT_PATH=$(cd "$(dirname "$0")";pwd)

# 在某些情况下会拿到错误结果
CURRENT_PATH=$(dirname $0)
CURRENT_PATH=$(pwd)
```

## Linux常用命令
```shell
// 后台执行
nohup ./main &

// 找到指定端口相关信息
netstat -antp | grep :7780

// 找到指定端口对应pid
netstat -antp | grep :7780 | awk '{print $7}' | awk -F '/' '{print $1}'

// 杀死指定端口对应进程
kill -9 $(netstat -antp | grep :7780 | awk '{print $7}' | awk -F '/' '{print $1}')

```

## sed 命令

unix 与 linux 在执行 sed 指令时有些许区别，当使用 sed -i 命令时，需要在 -i 指令后面多加一段 "", 即 `sed -i "" "s/192.168.0.2/192.168.0.3/g" filePath`

## curl 命令

## shell 工具

- jq：shell 下处理 json 神器，mac 安装命令 `brew install jq`
