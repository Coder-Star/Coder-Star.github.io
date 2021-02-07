---
title: pod相关
date: 2019-10-12 14:01:14
categories: [iOS]
tags: [iOS]
---

## 一、发布自己的 pod 库

发布一个共享库需要必不可少的部分

- **共享文件夹**  
  文件夹存放着你要共享的内容, 也就是其他人 pod 得到的文件, .podspec 文件中的 source_files 需要指定此文件路径及文件类型

- **LICENSE 文件**  
  默认一般选择 MIT
- **库描述文件.podspec**  
  本库的各项信息描述, 需要提交给 CocoaPods, pod 通过这个文件查找到你共享的库, .podspec 文件的格式见第 3 点

### 1、准备工程文件

代码开发过程中，需要注意将暴露出的方法以及类的访问显示符调整为 open 或者是 public

1. 目录如下![工程目录.png](../../../img/工程目录.png) 2.建立 name.podspec 文件，用于描述工程，其中 name 为工程的名字，该文件随项目一起 push 到 github 上
   其中 podspec 文件格式如下

```Ruby
Pod::Spec.new do |s|
  s.name         = "LTXiOSUtils"        #名称
  s.version      = "0.0.1"              #版本号
  s.summary      = "Just testing"       #简短介绍
  s.homepage     = "http://aoto.io/"     #主页地址，可以是github主页
  # s.screenshots  = "www.example.com/screenshots_1.gif" //快照
  s.license      = { :type => "MIT", :file => "LICENSE" } #开源协议
  s.author       = { "linyi31" => "linyi@jd.com" }
  s.source       = { :git => "https://github.com/Coder-Star/LTXiOSUtils.git", :tag => s.version } #共享库源代码地址
  # s.source     = { :git => 'local', :tag => s.version} #当在开发阶段，使用本地代码时，可以使用这种写法，实际上参数不重要，只有给予s.source 参数就行，后面的是为了避免警告
  s.platform     = :ios, "9.0"            #支持的平台及最低版本
  s.source_files = "LTXiOSUtils/Classes/**/*.swift"   #OC可以使用类似这样"Source/Classes/**/*.{h,m}",**/*是一个正则，表示下面所有swift文件，这个路径是相对于podspec文件而言
  #s.exclude_files = "LTXiOSUtils/Classes/Exclude" #忽略提交的文件
  s.swift_version = "4.2"  #设置swift版本
  s.dependency "Alamofire", "~> 4.7"    #依赖关系，该项目所依赖的其他库，如果有多个可以写多个 s.dependency
  s.dependency 'SwiftyJSON', '~> 4.0'
  s.resources     = 'LTXiOSUtils/Resource/*.png'  #资源文件路径

  #文件分层，如果Classes下面只有子目录，没有文件，则上述的s.source_files可以不用写
  s.subspec 'Utils' do |ss1|
      ss1.source_files = 'LTXiOSUtils/Classes/Utils/*.swift'
  end
  s.subspec 'View' do |ss2|
      ss2.dependency 'LTXiOSUtils/Utils'
      ss2.source_files = 'LTXiOSUtils/Classes/View/*.swift'
      ss2.subspec 'Button' do |sss1|
        sss1.source_files = 'LTXiOSUtils/Classes/View/Button/*.swift'
      end
      ss2.subspec 'TextView' do |sss2|
        sss2.source_files = 'LTXiOSUtils/Classes/View/TextView/*.swift'
      end
  end

end
```

3.工程完成之后可以先使用 pod lib lint 命令验证一下描述文件是否正确，如果通过验证，就打上 tag 然后 push 上去，其中需要注意 tag 版本会成为 pod 库的版本

```git
git tag -m '描述' '1.0.0'
git push --tags //推送所有tag，也可以使用  git push origin '1.0.0' 推送指定tag到远程
```

### 2、pod 相关命令

#### 1、注册及查看等等

- **pod trunk register xxxx@qq.com 'name'**  
  pod 注册,包括邮箱以及用户名，会让邮箱发送一个链接，点击链接完成注册

- **pod trunk me**  
  查看自己的 pod 相关信息，包括发布过的库
- **pod lib lint**  
  验证 podspec，验证过程中可能会出现警告造成验证不成功（`The spec did not pass validation`）,可以使用 **pod lib lint --allow-warnings**忽略警告完成验证

- **pod search XXX**  
  搜索共享库信息

- **pod outdated**  
  查询依赖的三方库是否有更新

#### 2、创建

- **pod lib create XXX**  
  创建 pod 库的基本结构，在创建过程中会给出选项让你做选择，包括是否需要 Example 工程等等

- **pod spec create XXX**  
  创建库的描述文件，XXX 为依赖库的名字，使用这种方式创建不要添加.podspec 扩展名，这种方式只会创建描述文件

#### 3、公有库

- **pod trunk push XXX.podspec**  
  发布到公有库

- **pod trunk delete XXX 版本号**  
  删除某个 pod 的特定版本

- **pod trunk deprecate XXX**  
  放弃整个 pod 库(会弹出确认框提示确认)

#### 4、私有库

- **pod repo add 私有库名称 私有库地址**  
  添加私有库地址到本地 repo，这里的私有库地址可以是 gitee、gitlab、github 等 git 服务器地址

- **pod repo remove xxxx** 删除某一个私有 Spec Repo

- **pod repo push 私有库名称 xxxx.podspec**  
  将描述文件上传到私有库

#### 5、安装相关命令

```Ruby
pod install --no-repo-update #安装跳过更新podspec索引，如果指定版本了，会安装指定版本，如果没有指定版本，则不会更新；pod install --verbose --no-repo-update 查看详细更新信息
pod update --no-repo-update #更新跳过更新podspec索引，不指定版本，会将版本更新到最新版； pod update --verbose --no-repo-update 查看更新过程中的详细信息
pod setup #生成本地spec仓库
pod spec lint #从本地以及远程验证pod是否能够通过验证
```

#### 6、其他

```Ruby
pod trunk add-owner 名称 邮箱 #如果你的pod是多人维护的，可以使用该命令添加写作者
pod cache clean --all #清理
pod deintegrate #移除pod库依赖
```

### 3、注意事项

- 将共享库上传到仓库地址后，一般使用`pod setup`以及`pod search` 查看自己的库，如果遇到上传很长时间还是无法查询到自己库的时候，可以先将`pod setup`成功后生成的~/Library/Caches/CocoaPods/search_index.json 文件删除后再进行`pod search`

  `rm ~/Library/Caches/CocoaPods/search_index.json`  
  如果还不成功，还可以使用`pod search xxx --simple`进行搜索。

- 当想在工程文件中使用私有库时候，需要在 Podfile 前面加上私有 Spec repo 的 git 地址，如`source https://github.com/Coder-Star/LTXSpecs.git`,如果还想使用公有库资源，再把公有库地址加上`source 'https://github.com/CocoaPods/Specs.git'`

- 当想删除私有库中某一个 podspec 时，可以直接在 Spec Repo 代码中将某个私有库 podspec 的文件删除，然后提交后远程就可以了

## 二、Podfile 文件样式

```Ruby
target 'BaseIOSProject' do
  source 'https://github.com/Coder-Star/LTXSpecs.git' #自己的私有库，在前面
  source 'https://github.com/CocoaPods/Specs.git' #公有库
  #source 'https://cdn.cocoapods.org/' # cocoapods 1.7.2 加入了cdn，用于替换https://github.com/CocoaPods/Specs.git共有源
  platform :ios, '10.0'
  use_frameworks!
  # use_modular_headers!

  pod 'Alamofire', '~> 4.7', :modular_headers => true #网络请求
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

#限制引入的库的swift语言版本
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

```Ruby
pod 'Alamofire' //不显式指定依赖库版本，表示每次都获取最新版本
pod 'Alamofire', '2.0' //只使用2.0版本
pod 'Alamofire', '> 2.0' //使用高于2.0的版本
pod 'Alamofire', '>= 2.0' //使用大于或等于2.0的版本
pod 'Alamofire', '< 2.0' //使用小于2.0的版本
pod 'Alamofire', '<= 2.0' //使用小于或等于2.0的版本
pod 'Alamofire', '~> 0.1.2' //使用大于等于0.1.2但小于0.2的版本
pod 'Alamofire', '~>0.1' //使用大于等于0.1但小于1.0的版本
pod 'Alamofire', '~>0' //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本

不从spec仓库中取，直接从github仓库上取
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :tag => '0.7.0'
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'develop'
pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :commit => '082f8319af'

pod 'Alamofire', :podspec => 'https://github.com/Alamofire/Alamofire/blob/master/Alamofire.podspec'

pod 'Alamofire',:path => '../' # 本地库，寻找对应的Alamofire.podspec文件
```

- 纯 OC 项目中，通过 cocoapods 导入 OC 库时，一般都不使用 use_frameworks!
- 纯 swift 项目中，通过 cocoapods 导入 swift 库时，必须使用 use_frameworks!
- 只要是通过 cocoapods 导入 swift 库时，都必须使用 use_frameworks!
- 使用动态链接库 dynamic frameworks 时，必须使用 use_frameworks!

## 三、关于项目引入 CocoaPods 的一些建议

### 1、CocoaPods 生成的 Pods 目录以及 Podfile.lock 等文件不加入到.gitignore 中去

- 这样做可以保证协作方 pull 完代码之后不需要执行 pod 命令，甚至可以不装 CocoaPods 就可以参与开发，避免因为 github 的问题导致 pod install 执行失败的问题出现
- 如果不带入 Podfile.lock，可能会导致 pod install 获得的库版本不同

### 2、Podfile 中依赖的第三方库使用精确版本号，不使用任何’~> 1.0', ‘>= 1.0'之类的版本号语法，只使用精确版本号，如'1.0.0'

这种做法可以保证三方库的可控性，避免 pod update 产生更新

### 3、当两个人同时修改 Podfile 文件，产生冲突

忽略其中一个人的包文件，只保留 Podfile 文件，合并完成之后，重新 install

## 打包

### framework

1. 安装 cocoapods-packager `gem "cocoapods-packager"`
2. 打包 `pod package XXX.podspec`

命令

```ruby
pod package XXX.podspec --force // --force表示强制覆盖之前存在的文件
pod package XXX.podspec --library // --library表示打包成a文件，如果不加就是打包成framework文件
```

## 相关资料

[用 CocoaPods 做 iOS 程序的依赖管理--唐巧](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)
