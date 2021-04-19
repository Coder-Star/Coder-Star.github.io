---
title: iOS持久化方式-keychain
category:
  - iOS
  - 基础原理
tags:
  - iOS
  - 持久化
date: 2021-02-24 22:43:50
---

iOS 系统的 Keychain 是一个安全的存储容器，它本质上就是一个 sqllite 数据库，它的位置存储在/private/var/Keychains/keychain-2.db。其特性就是存储的所有数据内部都会进行加密，可以用来存储比较敏感的信息，比如密码等。

这种存储方式是独立于 APP 沙盒之外的，所以即使 APP 被删除后，Keychain 里面存储的信息依然存在。

Keychain 用于 App 间通信的一个典型场景也和 app 的登录相关，就是统一账户登录平台。使用同一个账号平台的多个 app，只要其中一个 app 用户进行了登录，其他 app 就可以实现自动登录不需要用户多次输入账号和密码。一般开放平台都会提供登录 SDK，在这个 SDK 内部就可以把登录相关的信息都写到 keychain 中，这样如果多个 app 都集成了这个 SDK，那么就可以实现统一账户登录了。
