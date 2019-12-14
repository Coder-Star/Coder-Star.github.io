---
title: npm常用命令
date: 2019-10-31 13:43:26
categories: [NodeJS]
tags: [npm]
---
npm是NodeJS的包管理工具

## 1、安装

### 1、npm install

1. **npm install <模块名> -g**  
   全局安装
2. **npm install <模块名>**  
   本地安装
3. **npm install <模块名> --save-dev**  
   项目中安装并将模块保存到package.json中

### 2、安装淘宝镜像

**npm install -g cnpm --registry=https://registry.npm.taobao.org**  
安装完毕之后就可以使用cnpm install这种方式了
