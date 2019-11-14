---
title: pod相关
date: 2019-10-12 14:01:14
categories: [iOS]
tags: [iOS]
---
## 一、发布自己的pod库

发布一个共享库需要必不可少的部分

* **共享文件夹**  
  文件夹存放着你要共享的内容, 也就是其他人pod得到的文件, .podspec文件中的source_files需要指定此文件路径及文件类型

* **LICENSE文件**  
  默认一般选择MIT
  
* **库描述文件.podspec**  
  本库的各项信息描述, 需要提交给CocoaPods, pod通过这个文件查找到你共享的库, .podspec文件的格式见第3点

### 1、准备工程文件

代码开发过程中，需要注意将暴露出的方法以及类的访问显示符调整为open或者是public

1. 目录如下![工程目录.png](/pod相关/工程目录.png)
2.建立 name.podspec文件，用于描述工程，其中name为工程的名字，该文件随项目一起push到github上
其中podspec文件格式如下

```pod
Pod::Spec.new do |s|
  s.name         = "O2View"             #名称
  s.version      = "0.0.1"              #版本号
  s.summary      = "Just testing"       #简短介绍
  s.homepage     = "http://aoto.io/"     #主页地址，可以是github主页
  # s.screenshots  = "www.example.com/screenshots_1.gif" //快照
  s.license      = { :type => "MIT", :file => "LICENSE" } #开源协议
  s.author       = { "linyi31" => "linyi@jd.com" }
  s.source       = { :git => "https://github.com/Coder-Star/LTXiOSUtils.git", :tag => s.version } #共享库源代码地址
  s.platform     = :ios, "9.0"            #支持的平台及版本，这里我们用swift，直接上9.0
  s.source_files = "Source/Classes/**/*.swift"   #OC可以使用类似这样"Source/Classes/**/*.{h,m}",**/*是一个正则，表示下面所有swift文件，这个路径是相对于podspec文件而言
  s.swift_version = "4.2"  #设置swift版本
  s.dependency "Alamofire", "~> 4.7"    #依赖关系，该项目所依赖的其他库，如果有多个可以写多个 s.dependency
  s.dependency 'SwiftyJSON', '~> 4.0'
  s.resources     = 'Source/Resource/*.png'  #资源文件路径

  #文件分层，如果Classes下面只有子目录，没有文件，则上述的s.source_files可以不用写
  s.subspec 'Utils' do |ss1|
      ss1.source_files = 'Source/Classes/Utils/*.swift'
  end
  s.subspec 'View' do |ss2|
      ss1.source_files = 'Source/Classes/View/*.swift'
      s.subspec 'Button' do |sss1|
        sss1.source_files = 'Source/Classes/View/Button/*.swift'
      end
      s.subspec 'TextView' do |sss2|
        sss1.source_files = 'Source/Classes/View/TextView/*.swift'
      end
  end

end
```

3.工程完成之后可以先使用pod lib lint命令验证一下描述文件是否正确，如果通过验证，就打上tag然后push上去，其中需要注意tag版本会成为pod库的版本

``` git
git tag -m '描述' '1.0.0'
git push --tags //推送所有tag，也可以使用  git push origin '1.0.0' 推送指定tag到远程
```

### 2、pod处理

#### 1、注册及查看等等

* **pod trunk register xxxx@qq.com 'name'**  
  pod注册,包括邮箱以及用户名

* **pod trunk me**  
  查看自己的pod相关信息，包括发布过的库
  
* **pod lib lint**  
  验证podspec，验证过程中可能会出现警告造成验证不成功（`The spec did not pass validation`）,可以使用 **pod lib lint --allow-warnings**忽略警告完成验证

* **pod search XXX**  
  搜索共享库信息

#### 2、创建

* **pod lib create XXX**  
  创建pod库的基本结构，在创建过程中会给出选项让你做选择，包括是否需要Example工程等等

* **pod spec create XXX**  
  创建库的描述文件，XXX为依赖库的名字，使用这种方式创建不要添加.podspec扩展名，这种方式只会创建描述文件

#### 3、公有库

* **pod trunk push XXX.podspec**  
  //发布到公有库

#### 4、私有库

* **pod repo add 私有库名称 私有库地址**  
  添加私有库地址到本地repo，这里的私有库地址可以是gitee、gitlab、github等git服务器地址

* **pod repo push 私有库名称 xxxx.podspec**  
  将描述文件上传到私有库

### 3、注意事项

* 将共享库上传到仓库地址后，一般使用`pod setup`以及`pod search` 查看自己的库，如果遇到上传很长时间还是无法查询到自己库的时候，可以先将`pod setup`成功后生成的~/Library/Caches/CocoaPods/search_index.json文件删除后再进行`pod search`
`rm ~/Library/Caches/CocoaPods/search_index.json`

* 当想在工程文件中使用私有库时候，需要在Podfile前面加上私有Spec repo的git地址，如`source https://github.com/Coder-Star/LTXSpecs.git`,如果还想使用公有库资源，再把公有库地址加上`source 'https://github.com/CocoaPods/Specs.git'`

## 二、profile文件样式

```pod
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

```pod
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

## 三、Pod相关命令
