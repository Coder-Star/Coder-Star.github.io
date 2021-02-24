---
title: ES相关
date: 2020-10-30 09:00:21
tags:
---

##

### 布尔值

ECMAScript 中所有类型都有与 true 或 false 这两个值等价的值，要将一个值转换为其对应的布尔
你可以使用!!操作符将 truthy 或 falsy 值转换为布尔值

- !!"" // false
- !!0 // false
- !!null // false
- !!undefined // false
- !!NaN // false
- !!"hello" // true
- !!1 // true
- !!{} // true
- !![] // true

### 深拷贝

1. 使用 JSON 的形式
   `JSON.parse(JSON.stringify(obj))`，最常见的深拷贝就是这种方式了，但是这种方法的弊端就是`undefined`、`function`、`symbol` 会在转换过程中被忽略
2. 递归方式，一层层的拷贝
3. 第三方插件，如 lodash 的 cloneDeep 方法

## ES6
