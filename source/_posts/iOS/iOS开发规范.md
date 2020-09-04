---
title: iOS开发规范
date: 2020-08-31 10:50:18
categories: [iOS]
tags: [Swift,iOS,规范]
---

## Swift语言规范

### 命名规约

* 代码中的命名

* 代码中的命名严禁使用拼音及英文混合的方式，更不允许直接出现中文的方式

* class、struct、enum、protocol命名统一使用UpperCamelCase风格

* 方法名、参数名、成员变量、局部变量统一使用lowerCamelCase风格
  
* 常量命名使用UpperCamelCase风格
  

### 格式规约

* 类、函数左大括号不另起一行，与名称之间留有空格

* 无论是终止或者隔开声明都使用分号
  
* 代码中的空格出现地点
  * 

### 长度尺寸规约

* 每行代码长度应小于100个字符，或者阅读时候不应该需要滚动屏幕，在正常范围内可以看到完整代码
* 

### 简略规约

* Swift会被结构体按照自身的成员自动生成一个非public的初始化方法，如果这个初始化方法刚好适合，不要自己再声明
  
* 类及结构体初始化方法不要直接调用.init，直接直接省略，使用()

* 如果只有一个get的计算属性，忽略get

* 数据定义时，尽量使用字面量形式进行自动推断，如果上下文不足以推断字面量类型时，需要声明赋值类型

* 省略默认的访问权限（internal）



### 注释规约

### 其他

* 图形化的字面量，`#colorLiteral(...)`, `#imageLiteral(...)`只能用在playground当做自我练习使用，禁止在项目工程中使用

* 避免强制解包以及强制类型映射，尽量使用`if let` 或 `guard let`进行解包

* 避免判断语句嵌套层次太深，使用guard提前返回

* 如果for循环在函数体中只有一个if判断，使用for where进行替换
```
✅
for item in collection where item.hasProperty {

}

❌
for item in collection {
  if item.hasProperty {

  }
}
```

* 


### 工具

利用SwiftLint工具对Swift代码规范进行显示，[点击查看Realm地址](https://github.com/realm/SwiftLint)


### 资源文件命名及使用

### 基本组件

### UI布局

### 数据存储