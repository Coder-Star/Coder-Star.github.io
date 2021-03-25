---
title: CocoaPods-项目使用
category:
  - iOS
  - CocoaPods
tags:
  - iOS
  - CocoaPods
date: 2021-03-22 10:58:25
---

## 使用

1. 进入到项目根目录，执行`pod init`命令，该命令会生成 Podfile 文件，开发者就可以编辑该文件内容了
2. 编辑 Podfile 文件后执行`pod install`命令，命令执行完成后，会生成 ProjectName.xcworkspace、Podfile.lock、Pods 等文件。
3. 不再通过 ProjectName.xcodeproj 打开工程，而是使用 ProjectName.xcworkspace

**关于 Podfile.lock 与 Manifest.lock 文件的作用**

**Podfile.lock 文件：** 该文件会记录三方库第一次 install 时的版本，当其他协作者同步了你的 Podfile.lock 文件后，他执行 pod install 会安装 Podfile.lock 指定版本的依赖库，这样就可以防止大家的依赖库不一致而造成问题，因此，CocoaPods 官方强烈推荐把 Podfile.lock 纳入版本控制之下。当修改了 Podfile 文件里使用的依赖库或者运行 pod update 命令时，就会生成新的 Podfile.lock 文件。

**Manifest.lock 文件：** 该文件相当于 Podfile.lock 文件的副本，程序在 Build 的时候会执行 Build Phases 中的[CP] Check Pods Manifest.lock 脚本检查一下 Podfile.lock 和 Manifest.lock 是否一致,如果不一致就会报错。

**有了 Podfile.lock 那为啥还要有 Manifest.lock 的原因**：Pods 目录并不总是被放到版本控制之下，有了这个检查机制就能保证开发团队的各个小伙伴能在运行项目前更新他们的依赖库，并保持这些依赖库的版本一致，从而防止在依赖库的版本不统一造成程序在一些不明显的地方编译失败或运行崩溃。

**关于要不要将 Pods 目录交给 git 管理**

- 好处是协作方 pull 完代码之后不需要执行 pod 命令，甚至可以不装 CocoaPods 就可以参与开发，避免因为 github 的问题导致 pod install 执行失败的问题出现。也可以避免协作者之间因 CocoaPods 版本不同出现问题。
- 坏处是可能会出现 Pods 目录下的冲突，Pod 库会比较大，影响 git 传输。

**小 Tips：**

- Podfile 中依赖的第三方库使用精确版本号，不使用任何’~> 1.0', ‘>= 1.0'之类的版本号语法，只使用精确版本号，如'1.0.0'；这种做法可以保证不同的协作者依赖的三方库版本一致；

## Podfile 文件样式

```Ruby
target 'BaseIOSProject' do
  source 'https://github.com/Coder-Star/LTXSpecs.git' #自己的私有库，在前面
  source 'https://github.com/CocoaPods/Specs.git' #公有库
  #source 'https://cdn.cocoapods.org/' # cocoapods 1.7.2 加入了cdn，用于替换https://github.com/CocoaPods/Specs.git共有源

  platform :ios, '10.0'

  install! 'cocoapods', :clean  #全局

  # 动态库形式
  use_frameworks!

  # 使所有pod可以使用模块导入的方式
  # use_modular_headers!

  # 静态库形式
  # use_frameworks! :linkage => :static

  # 不显示所有pod的警告
  inhibit_all_warnings!
  # 控制单个pod库是否显示警告
  #,:inhibit_warnings => false

  # 明确指定继承于父层的所有 pod，默认就是继承的
  # inherit! :search_paths

  pod 'Alamofire', '~> 4.7', :modular_headers => true #网络请求
  pod 'MJRefresh' #上拉刷新、下拉下载

  pod 'QueryKit/Attribute' # 子模块
  pod 'QueryKit', :subspecs => ['Attribute', 'QuerySet'] # 多个子模块

  pod 'LTXiOSUtils',:path => '../' # 本地pod库，会寻找该路径下LTXiOSUtils.podspec文件

  pod 'LookinServer', :configurations => ['Debug'] # 查看UI，只编译进debug版本， :configurations => ['Debug','Test']

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
