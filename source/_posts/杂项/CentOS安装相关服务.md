---
title: CentOS安装相关服务
date: 2019-12-10 19:30:46
categories: [杂项]
tags: [CentOS]
---

## 1、NodeJs

- `curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -`
- `sudo yum -y install nodejs`
- `node -v`
- `npm -v`

## 2、JDK

[参考教程地址](https://blog.csdn.net/u010590120/article/details/94736800)

### 1、下载地址[点击跳转到下载地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

根据自己的系统版本选择合适的安装包，后缀要.tar.gz

## 3、Mysql

[参考教程地址](https://blog.csdn.net/wyhh_0101/article/details/88666649)

### 1、最新 RPM 地址(http://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)

从 mysql5.7 版本之后，默认采用了 caching_sha2_password 验证方式，使用下面命令将验证方式还是改为之前的
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';

### 4、wftpserver

[参考教程地址](https://bbs.wftpserver.com/viewtopic.php?f=2&t=49)

## 其他

- 查看 CentOS 系统版本 **cat /etc/redhat-release**
