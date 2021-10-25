---
title: Shell笔记
category:
  - 杂项
tags:
  - shell
date: 2021-05-27 20:59:49
---



## shell基础


$()与``都是用来做命令替换的，当需要获取命令返回的结果时，比较常用，如RESULT=`cat main.swift`，就可以将main.swift中的内容作为字符串赋值给变量RESULT。

`$`与`${}`都是用来获取变量内容的

`$?` 来获取上条代码返回值，一般退出状态非0表示执行出错

```
-eq //equals等于
-ne //no equals不等于
-gt //greater than 大于
-lt //less than小于
-ge //greater equals大于等于
-le //less equals小于等于
```


## 代码段

判断上行代码是否执行成功，如果命令执行成功，则返回0，否则返回1
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

## sed命令

unix与linux在执行sed指令时有些许区别，当使用sed -i 命令时，需要在-i指令后面多加一段 "", 即 `sed -i "" "s/192.168.0.2/192.168.0.3/g" filePath`

## curl命令


## shell好用工具

- jq：shell下处理json神器，mac安装命令 `brew install jq`