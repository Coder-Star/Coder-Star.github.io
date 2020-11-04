---
title: ES相关
date: 2020-10-30 09:00:21
tags:
---

## 

ECMAScript中所有类型都有与true或false这两个值等价的值，要将一个值转换为其对应的布尔
你可以使用!!操作符将truthy或falsy值转换为布尔值

* !!"" // false
* !!0 // false
* !!null // false
* !!undefined // false
* !!NaN // false
* !!"hello" // true
* !!1 // true
* !!{} // true
* !![] // true