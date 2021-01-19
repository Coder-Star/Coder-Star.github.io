---
title: Flutter环境搭建
date: 2019-10-12 13:36:22
categories: [Android]
tags: [Flutter]
---
## 1、下载Flutter

```git
git clone -b master https://github.com/flutter/flutter.git
```

## 2、配置环境变量

```vim
vim ~/.bash_profile

//加入内容
export PATH=/你的flutter文件夹所在位置/flutter/bin:$PATH
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

source ~/.bash_profile
flutter -h //显示相关帮助
```

## 3、flutter相关命令


* flutter doctor // 检查flutter相关环境
* flutter doctor --android-licenses

* flutter create xxapp // 创建Flutter Application
* flutter create -i swift -a kotlin xxapp //指定语言Android使用Kotlin，iOS使用Swift
* flutter create -t module xxapp_module //创建Flutter Module
* flutter create --org com.example --template=plugin --platforms=android,ios -a kotlin hello
 //创建Flutter Plugin
* flutter create --template=package xxapp_package // Flutter Package

* flutter create -i swift -a kotlin .  //如果在一个既存项目中运行这个命令，那么这将会修复当前项目，重新创建丢失的文件,注意最后面是一个`.`

## 4、Android Studio 安装相关插件

* Flutter
* Dart
* ...
