---
title: npm常用命令
date: 2019-10-31 13:43:26
categories: [NodeJS]
tags: [npm]
---

npm 是 NodeJS 的包管理工具

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
安装完毕之后就可以使用 cnpm install 这种方式了，提高下载速度

**其他情况**

- 删除 node_modules 文件夹  
  `npm install rimraf -g ` 先安装删除工具，再使用删除命令  
  `rimraf node_modules` 删除 node_modules 文件夹

- 出现类似 **Error: EACCES: permission denied** 这种错误
  原因是因为没有权限
  在命令前面 `sudo`

- 清除缓存  
  `npm cache clean --force` 在 node 5 版本以上，--force 需强制加上，如果不加上会报错。

- 在更新包的过程中出现了 file exist，can not remove 这种类型的错误  
  先使用`rimraf node_modules` 删除 node_modules 文件夹，然后直接是使用`npm update <模块名>`命令更新模块

```json
{
  "dependencies" :{
    "foo" : "1.0.0 - 2.9999.9999", // 指定版本范围
    "bar" : ">=1.0.2 <2.1.2",
    "baz" : ">1.0.2 <=2.3.4",
    "boo" : "2.0.1", // 指定版本
    "qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
    "asd" : "http://asdf.com/asdf.tar.gz", // 指定包地址
    "til" : "~1.2",  // 最近可用版本
    "elf" : "~1.2.3",
    "elf" : "^1.2.3", // 兼容版本
    "two" : "2.x", // 2.1、2.2、...、2.9皆可用
    "thr" : "*",  // 任意版本
    "thr2": "", // 任意版本
    "lat" : "latest", // 当前最新
    "dyl" : "file:../dyl", // 本地地址
    "xyz" : "git+ssh://git@github.com:npm/npm.git#v1.0.27", // git 地址
    "fir" : "git+ssh://git@github.com:npm/npm#semver:^5.0",
    "wdy" : "git+https://isaacs@github.com/npm/npm.git",
    "xxy" : "git://github.com/npm/npm.git#v1.0.27",
  }
}
```
