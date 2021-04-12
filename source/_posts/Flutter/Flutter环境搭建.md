---
title: Flutter环境搭建
date: 2019-10-12 13:36:22
categories: [Android]
tags: [Flutter]
---

## 1、下载 Flutter

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

## 3、flutter 相关命令

- flutter doctor // 检查 flutter 相关环境
- flutter doctor --android-licenses

- flutter create xxapp // 创建 Flutter Application
- flutter create -i swift -a kotlin xxapp //指定语言 Android 使用 Kotlin，iOS 使用 Swift
- flutter create -t module xxapp_module //创建 Flutter Module
- flutter create  --template=plugin --platforms=android,ios -i swift -a kotlin xxapp_plugin
  //创建 Flutter Plugin
- flutter create --template=package xxapp_package // Flutter Package

- flutter create -i swift -a kotlin . //如果在一个既存项目中运行这个命令，那么这将会修复当前项目，重新创建丢失的文件,注意最后面是一个`.`

## 4、Android Studio 安装相关插件

- Flutter
- Dart
- ...


## 5、使用fvm灵活切换flutter版本

```
// 添加homebrew tap
brew tap xinfeng-tech/fvm

// 安装 fvm
brew install fvm

// 安装flutter版本
fvm install 1.17.1

// 修改环境变量（~/.bash_profile文件中）
export FVM_DIR="$HOME/.fvm"
source "/usr/local/opt/fvm/init.sh"

并修改flutter的路径
export PATH=/你的flutter文件夹所在位置/flutter/bin:$PATH
修改为
/Users/用户名/.fvm/current/bin:$PATH

// 切换flutter版本
fvm use 1.17.1

// 查看flutter相关版本
fvm list
```

Tips：换成fvm管理flutter版本后第一次执行flutter命令会出现警告，可以项目中先执行一下 `flutter pub get` 然后再执行一下`flutter clean`，问题就会得到解决。