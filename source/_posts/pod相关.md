---
title: pod相关
date: 2019-10-12 14:01:14
categories: [iOS]
tags: [iOS]
---
# 一、发布自己的pod库
## 1、准备工程文件
代码开发过程中，需要注意将暴露出的方法以及类的访问显示符调整为open或者是public
1. 目录如下
![工程目录.png](/pod相关/工程目录.png)
2.建立 name.podspec文件，用于描述工程，其中name为工程的名字，该文件随项目一起push到github上
其中podspec文件格式如下
```
Pod::Spec.new do |s|
  s.name         = "O2View"             #名称
  s.version      = "0.0.1"              #版本号
  s.summary      = "Just testing"       #简短介绍
  s.description  =  <<-DESC
  SwifterSwift is a collection of over 500 native Swift extensions, with handy methods, syntactic sugar, and performance improvements for wide range of primitive data types, UIKit and Cocoa classes –over 500 in 1– for iOS, macOS, tvOS and watchOS.
                   DESC
  s.homepage     = "http://aoto.io/"
  # s.screenshots  = "www.example.com/screenshots_1.gif" //快照
  s.license      = "MIT"                #开源协议
  s.author       = { "linyi31" => "linyi@jd.com" }
  s.source       = { :git => "https://github.com/marklin2012/O2View.git" }
  ## 这里不支持ssh的地址，只支持HTTP和HTTPS，最好使用HTTPS
  ## 正常情况下我们会使用稳定的tag版本来访问，如果是在开发测试的时候，不需要发布release版本，直接指向git地址使用
  ## 待测试通过完成后我们再发布指定release版本，使用如下方式
  #s.source      = { :git => "http://EXAMPLE/O2View.git", :tag => version }
  s.platform      = :ios, "9.0"          #支持的平台及版本，这里我们用swift，直接上9.0
  s.requires_arc = true                 #是否使用ARC
  s.source_files  = "O2View/*.swift"    #OC可以使用类似这样"Classes/**/*.{h,m}"
  s.frameworks = 'UIKit', 'QuartzCore', 'Foundation'    #所需的framework,多个用逗号隔开
  s.module_name = 'O2View'              #模块名称
  # s.dependency "JSONKit", "~> 1.4"    #依赖关系，该项目所依赖的其他库，如果有多个可以写多个 s.dependency
  if s.respond_to? 'swift_version'  #设置swift版本
    s.swift_version = "4.2"
  end
end
```
3.工程完成之后可以先使用pod lib lint命令验证一下描述文件是否正确，如果通过验证，就打上tag然后push上去，其中需要注意tag版本会成为pod库的版本
```
git tag '1.0.0'
git push --tags //推送所有tag，也可以使用  git push origin '1.0.0' 推送指定tag到远程
```
## 2、pod处理
* **pod trunk register xxxx@qq.com "name"**
  //pod注册,包括邮箱以及用户名
* **pod trunk me**
  //查看自己的pod相关信息
* **pod spec create name**       
//创建库的描述文件，name为依赖库的名字，使用这种方式创建不要添加.podspec扩展名
* **pod lib lint**
//验证podspec，验证过程中可能会出现警告造成验证不成功（`The spec did not pass validation`）,可以使用 **pod lib lint --allow-warnings**忽略警告完成验证
* **pod trunk push name.podspec**
//发布pod

# 二、profile文件样式
```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'BaseIOSProject' do
  source 'https://github.com/CocoaPods/Specs.git'
  platform :ios, '10.0'
  use_frameworks!

  pod 'Alamofire', '~> 4.7' #网络请求
  pod 'MJRefresh' #上拉刷新、下拉下载

  target 'BaseIOSProjectTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'BaseIOSProjectUITests' do
    inherit! :search_paths
    # Pods for testing
  end
end

#设计引入的库的swift语言版本
swift_42 = ['Alamofire','MJRefresh']
post_install do |installer|
  installer.pods_project.targets.each do |target|
    swift_version = nil
    
    if swift_42.include?(target.name)
      swift_version = '4.2'
    end
    
    if swift_version
      target.build_configurations.each do |config|
        config.build_settings['SWIFT_VERSION'] = swift_version
      end
    end
  end
end

```
```
pod 'Alamofire' //不显式指定依赖库版本，表示每次都获取最新版本
pod 'Alamofire', '2.0' //只使用2.0版本
pod 'Alamofire', '> 2.0' //使用高于2.0的版本
pod 'Alamofire', '>= 2.0' //使用大于或等于2.0的版本
pod 'Alamofire', '< 2.0' //使用小于2.0的版本
pod 'Alamofire', '<= 2.0' //使用小于或等于2.0的版本
pod 'Alamofire', '~> 0.1.2' //使用大于等于0.1.2但小于0.2的版本
pod 'Alamofire', '~>0.1' //使用大于等于0.1但小于1.0的版本
pod 'Alamofire', '~>0' //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本
```
