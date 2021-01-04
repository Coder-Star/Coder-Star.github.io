---
title: npm常用命令
date: 2019-10-31 13:43:26
categories: [NodeJS]
tags: [npm]
---
npm是NodeJS的包管理工具

**安装**
```
npm install <模块名> -g 
全局安装

npm install <模块名>
本地安装

npm install <模块名> --save-dev 或 npm install gulp -S
项目中安装并将模块保存到package.json中devDependencies

npm install <模块名> --save  或 npm install gulp -D  
项目中安装并将模块保存到package.json中dependencies

npm install gulp --save-optional 或 npm install gulp -O
项目中安装并将模块保存到package.json中optionalDependencies

```

**安装淘宝镜像**

**`npm install -g cnpm --registry=https://registry.npm.taobao.org`**  
安装完毕之后就可以使用cnpm install这种方式了，提高下载速度

**其他情况**

* 删除node_modules文件夹  
  `npm install rimraf -g ` 先安装删除工具，再使用删除命令  
  `rimraf node_modules` 删除node_modules文件夹

* 出现类似 **Error: EACCES: permission denied** 这种错误
  原因是因为没有权限
  在命令前面 `sudo`

* 清除缓存  
  `npm cache clean --force`  在node 5版本以上，--force需强制加上，如果不加上会报错。

* 在更新包的过程中出现了file exist，can not remove这种类型的错误  
  先使用`rimraf node_modules` 删除node_modules文件夹，然后直接是使用`npm update <模块名>`命令更新模块