---
title: CocoaPods相关
category:
  - iOS
  - CocoaPods
tags:
  - iOS
  - CocoaPods
date: 2021-03-22 08:47:15
---


## 基本概念

CocoaPods 是一个 ruby 工具，Mac 下优秀的第三方包管理工具, 帮助管理和集成, 自动更新网络上的第三方类库. 方便了配置。CocoaPods 默认只能管理基于 git 管理的代码，如果要使用 svn 或者 mercurial 管理代码，则需要安装一些插件 (cocoapods-repo-svn). 我们通过 git 上的插件把私有代码通过 svn 下载到本地的私有仓库. 这样委托 pod 来管理.

远程索引库，本地索引库，远程代码库、本地代码库相关概念如下：
![CocoaPods概念图](../../../../img/iOS/架构设计/CocoaPods/CocoaPods概念图.png)

远程索引库地址为`https://github.com/CocoaPods/Specs.git`

本地索引库存储位置 `~/.cocoapods`，缓存存储位置：`~/Library/Caches/CocoaPods`

缓存存储目录结构如下：
![缓存存储目录结构](../../../../img/iOS/架构设计/CocoaPods/缓存存储目录.png)

**搜索并安装 Cocoapods 上的某一个库，大致执行流程为：**

- 从 search_index.json 中找到该库对应版本的 podspec 文件并将其下载到 Specs/Release 文件夹中
- 根据该库的 podspec 文件中的 source 中的 git 地址，将对应版本的库下载到 Pods/Release 文件夹中

**将本地自己开发的库推送到 Cocoapods 仓库时，大致执行流程为：**

- 执行 pod spec lint XXX.podspec 时，如果没有依赖其他库时，会在 Specs/External 文件中该库生成对应版本的 podsepc 文件，并对其文件名进行 hash
- 将该 podspec 文件中对应的库下载到 Pods/External 文件夹中
- 如果发布的库依赖其他 Cocoapods 上发布的库时，会将依赖库的 podspec 文件下载到 Specs/Release 文件夹中，依赖的库下载到 Pods/Release 文件夹中

Cocoapods 1.7.2 就可以使用 CDN 的方式下载索引库，需要在 Podfile 文件里加上`source 'https://cdn.cocoapods.org/'`，而到 Cocoapods 1.8.0 时，直接将 CDN 作为默认仓库来源。使用 CDN 分发，直接找到第三方库的 spec 地址，直接下载，不需要全量下载远程索引库到本地。

## 原理

当所有的依赖库都下载完后，Cocoapods 会将所有的依赖库都放到另一个名为 Pods 的项目中，然后让主项目依赖 Pods 项目。这样，源码管理工作都从主目录移到了 Pods 项目中。

动态库形式，工程变动如下：

- 在 General -> Linked Frameworks and Libraries 中添加了 Pods_test.framework 的依赖
- Build Settings -> Other Linker Flags 中添加了 -framework "AFNetworking"
- Build Settings -> RunPath Search Paths 中添加了 @executable_path/Frameworks
- Build Settings -> Other C Flags 添加 -iquote "$CONFIGURATION_BUILD_DIR/AFNetworking.framework/Headers" 和 Other C++ Flags 添加 $(OTHER_CFLAGS)，也就是说，跟随 Other C Flags 配置
- Build Settings -> Preprocessor Macros 添加 COCOAPODS=1
- Build Settings -> User-Defined -> MTL_ENABLE_DEBUG_INFO PODS_FRAMEWORK_BUILD_PATH PODS_ROOT 三个变量
- Build Phases -> Embed Pods Frameworks 添加 "${SRCROOT}/Pods/Target Support Files/Pods-test/Pods-test-frameworks.sh" // 静态库不会生成该文件
- Build Phases -> Copy Pods Resources 添加 "${SRCROOT}/Pods/Target Support Files/Pods-test/Pods-test-resources.sh" // 如果三方库中没有资源文件，则不会生成该文件

两个 sh 文件实际上就是将封装通用架构动态库文件和将动态库复制到 bundle 指定目录下。

如果是静态库方式引入，Pods 项目最终会编译成为一个名为 libPods- 项目名称.a 的文件，主项目只要依赖这个.a 文件即可；

如果是以动态库形式引入三方库，项目会依赖一个名为 Pods\_项目名称.framework 的文件，该文件是一个静态库，只是起到一个集合的作用，实际的二进制代码还是放置在各个动态库之中。

## 使用

1. 进入到项目根目录，执行`pod init`命令，该命令会生成 Podfile 文件，开发者就可以编辑该文件内容了
2. 编辑 Podfile 文件后执行`pod install`命令，命令执行完成后，会生成 ProjectName.xcworkspace、Podfile.lock、Pods 等文件。
3. 不再通过 ProjectName.xcodeproj 打开工程，而是使用 ProjectName.xcworkspace

**关于 Podfile.lock 与 Manifest.lock 文件的作用**

**Podfile.lock 文件：** 该文件会记录三方库第一次 install 时的版本，当其他协作者同步了你的 Podfile.lock 文件后，他执行 pod install 会安装 Podfile.lock 指定版本的依赖库，这样就可以防止大家的依赖库不一致而造成问题，因此，CocoaPods 官方强烈推荐把 Podfile.lock 纳入版本控制之下。当修改了 Podfile 文件里使用的依赖库或者运行 pod update 命令时，就会生成新的 Podfile.lock 文件。

**Manifest.lock 文件：** 该文件相当于 Podfile.lock 文件的副本，程序在 Build 的时候会执行 Build Phases 中的 [CP] Check Pods Manifest.lock 脚本检查一下 Podfile.lock 和 Manifest.lock 是否一致, 如果不一致就会报错。

**有了 Podfile.lock 那为啥还要有 Manifest.lock 的原因**：Pods 目录并不总是被放到版本控制之下，有了这个检查机制就能保证开发团队的各个小伙伴能在运行项目前更新他们的依赖库，并保持这些依赖库的版本一致，从而防止在依赖库的版本不统一造成程序在一些不明显的地方编译失败或运行崩溃。

**关于要不要将 Pods 目录交给 git 管理**

- 好处是协作方 pull 完代码之后不需要执行 pod 命令，甚至可以不装 CocoaPods 就可以参与开发，避免因为 github 的问题导致 pod install 执行失败的问题出现。也可以避免协作者之间因 CocoaPods 版本不同出现问题。
- 坏处是可能会出现 Pods 目录下的冲突，Pod 库会比较大，影响 git 传输。

**小 Tips：**

- Podfile 中依赖的第三方库使用精确版本号，不使用任何'~> 1.0', '>= 1.0'之类的版本号语法，只使用精确版本号，如'1.0.0'；这种做法可以保证不同的协作者依赖的三方库版本一致；

## Podfile 文件样式

全部格式请见 [官方文档](https://guides.cocoapods.org/syntax/podfile.html)

```Ruby
def rn_dependency
  pod "Yoga", :path => "node_modules/react-native/ReactCommon/yoga"
end

target 'BaseIOSProject' do
  
  source 'https://github.com/Coder-Star/LTXSpecs.git' #自己的私有库，在前面
  source 'https://github.com/CocoaPods/Specs.git' #公有库
  
  #source 'https://cdn.cocoapods.org/' # cocoapods 1.7.2 加入了cdn，用于替换https://github.com/CocoaPods/Specs.git共有源

  platform :ios, '10.0'

  #全局
  install! 'cocoapods', :clean, :disable_input_output_paths => true

  # disable_input_output_paths设置完在 copy pods resource 脚本上不会再有input file以及outfile，这种可以保证每次编译都重新编译，不会存在代码不生效的情况，但是会出现编译速度变的很慢的现象，所以最好不要打开，此还能解决的case是，子pod里面含有assets，而且是以resource的方式引入的。

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

  # 公有pod使用
  rn_dependency

  target 'BaseIOSProjectTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'BaseIOSProjectUITests' do
    inherit! :search_paths
    # Pods for testing
  end
end

pre_install do |installer|

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


# 替换某个文件内容，起到紧急修复Bug的目的
post_install do |installer|
    find_and_replace("xx.Swift", bugCode, fixCode)
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

pod 'Alamofire',:path => '../' # 本地库，寻找对应的 Alamofire.podspec文件
```
## 准备自己的 Pod 库

布一个共享库一般包含三部分

- **库源码及资源文件** 你想要发布出去的相关资源文件
- **LICENSE 文件** 默认一般选择 MIT
- **podspec 文件** 本库的各项信息描述, 需要提交给 CocoaPods, pod 通过这个文件查找到你共享的库

目录如下
![工程目录.png](../../../../img/iOS/架构设计/工程目录.png)

生成这样的目录结构可以利用 CocoaPods 的命令自动生成

```
- **pod lib create XXX**
  创建 pod 库的基本结构，在创建过程中会给出选项让你做选择，包括是否需要 Example 工程等等

- **pod spec create XXX**
  创建库的描述文件，XXX 为依赖库的名字，使用这种方式创建不要添加.podspec 扩展名，这种方式只会创建描述文件
```

**podspec 文件格式**

全部配置请见 [官方文档](https://guides.cocoapods.org/syntax/podspec.html#specification)

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
  s.platform     = :ios, "9.0" # 支持的平台及最低版本
  # s.ios.deployment_target = '7.0' # 或者这种写法，这种写法可以设置多平台
  s.requires_arc  = true # arc和mrc选项，true一定不要加单引号
  s.swift_version = "4.2"  #设置swift版本
  s.static_framework  =  true # 设置库为静态库，mach-o type会是static，设置之后即使项目的 Podfile 中使用了 use_frameworks! ，使用 该pod 也会以静态库使用

  s.source_files = "LTXiOSUtils/Classes/**/*.swift"   #OC可以使用类似这样"Source/Classes/**/*.{h,m}",**/*是一个正则，表示下面所有swift文件，这个路径是相对于podspec文件而言
  # s.exclude_files = "LTXiOSUtils/Classes/Exclude" #忽略提交的文件


  s.dependency "Alamofire", "~> 4.7"    #依赖关系，该项目所依赖的其他库，如果有多个可以写多个 s.dependency
  s.dependency 'SwiftyJSON', '~> 4.0'


  # resources及resource_bundle都是设置资源的方式

  s.resources     = 'LTXiOSUtils/Resource/*.png'  #资源文件路径
  # cocoapods 官方推荐使用 resource_bundles，因为 resources  指定的资源会被直接拷贝到目标应用中，因此不会被 Xcode 优化，在编译生成 product 时，与目标应用的图片资源以及其他同样使用 resources 的 Pod 的图片一起打包为一个 Assets.car 文件。这样全部混杂在一起，就使得资源文件容易产生命名冲突。而 resource_bundles 指定的资源，会被编译到独立的 bundle 中，bundle 名就是你的 pod 名，这样就很大程度上减小了资源名冲突问题，并且 Xcode 会对 bundle 进行优化。一个 bundle 包含一个 Assets.car，获取图片的时候要严格指定 bundle 的位置，很好的隔离了各个库或者一个库下的资源文件。
  resources.resource_bundle = { "LTXiOSUtils" => "LTXiOSUtils/Resources/Resource/*" } # LTXiOSUtil是bundle的名称。

  s.vendored_libraries = 'YJDemoSDK/Classes/libWeChatSDK.a' # 表示依赖第三方/自己的 .a / 静态库，依赖的第三方的或者自己的静态库文件必须以lib为前缀进行命名，否则会出现找不到的情况，这一点非常重要
  s.public_header_files = 'YJDemoSDK/Classes/YJDemoSDK.h'   #需要对外开放的头文件，如果在swift工程中，这个头文件会被放置到umbrella-header中
  s.module_map = 'source/module.modulemap' # 自定义modulemap

  s.private_header_files = 'xxxx.h' #私有h文件，该文件放置库中使用的第三方的头文件

  s.vendored_frameworks = "frameworks/Test.framework" # 第三方framework目录，当库不想公布源码时可以使用这种方式
  s.frameworks = "UIKit","Foundation" #需要引入的系统frameworks
  s.libraries  = 'z', 'sqlite3' #表示依赖的系统类库，比如libz.dylib等

# 模块化，假如项目中有OC代码，需要模块化，就需要进行开启，并且配合public_header_files使用
#  s.pod_target_xcconfig = {
#    'DEFINES_MODULE' => 'YES'
#  }

  s.default_subspec = "Utils"

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

**小 Tips**

- 代码开发过程中，需要注意将暴露出的方法以及类的访问显示符调整为 open 或者是 public，使代码可以被外部正常访问到
- 本地库调试时，Demo 工程 Podfile 文件引用本地库的方式使用`pod 'LTXiOSUtils',:path => '../'`这种形式，如果调试过程中出现修改不生效的问题，请及时 clean。

## 发布 Pod 库

### 正式发布前准备

podspec 文件关联的源代码地址是以 tag 为准，所以发布 pod 库之前还要保证远程仓库中已经存在本地发布依赖的 tag，tag 设置相关命令如下：

```git
git tag -m '描述' '1.0.0'

// 推送所有tag，也可以使用  git push origin '1.0.0' 推送指定tag到远程
git push --tags
```

库发布之前需要使用命令验证库的格式及代码是否有问题，命令如下：

```
// 验证 本地podspec，验证过程中可能会出现警告造成验证不成功（The spec did not pass validation）,可以使用pod lib lint --allow-warnings忽略警告完成验证
// 也可以添加 `--skip-import-validation` 忽略导入验证
// 当出现 ERROR | [iOS] xcodebuild: Returned an unsuccessful exit code 就可以使用
pod lib lint

// 验证本地podspec及远程podspec文件
pod spec lint
```

在发布 Pod 之前需要先进行注册 CocoaPods 账号，注册命令如下：

```
// pod 注册,包括邮箱以及用户名，会让邮箱发送一个链接，点击链接完成注册
pod trunk register xxxx@qq.com 'name'
```

### 发布

发布 Pod 库有两种方式，一种方式是发布到 CocoaPods 的中心仓库，这样是开源出去的，另外一种是发布到私有索引库，用于不开源，公司内部使用。

#### 公有索引库

```
// 发布到公有库
pod trunk push XXX.podspec

// 删除pod库的的某一个版本
pod trunk delete XXX 版本号

// 放弃整个 pod 库(会弹出确认框提示确认)
pod trunk deprecate XXX
```

#### 私有索引库

```
// 添加私有索引库地址到本地 repo，地址可以是 gitee、gitlab、github 等 git 地址
pod repo add 私有库名称 私有库地址

// 删除某一个私有 Spec Repo
pod repo remove xxxx

// 将描述文件上传到私有索引库，这时私有索引库中就会多出提交的podspec文件
// 其实就是将库发布到私有源
pod repo push 私有索引库名称 xxxx.podspec
```

外部使用私有索引库里面的 pod 库时需要在 Podfile 文件中加上私有源，最好放在其他源的前面

```
source 'https://github.com/Coder-Star/LTXSpecs.git'
```

将共享库上传到仓库地址后，一般使用`pod setup`以及`pod search` 查看自己的库，如果遇到上传很长时间还是无法查询到自己库的时候，可以先将`pod setup`成功后生成的~/Library/Caches/CocoaPods/search_index（`rm ~/Library/Caches/CocoaPods/search_index.json`），json 文件删除后再进行`pod search`。

如果还不成功，还可以使用`pod search xxx --simple`进行搜索。

## 其他

```
// 查看自己的 pod 相关信息，包括发布过的库
pod trunk me

```


## 工具链

ruby：`brew install rbenv`
rvm/rbenv：使用这个安装指定 ruby 版本。`rbenv install 2.7.0`
gem：ruby 环境下的包
Bundler：gem 包管理器，`bundle init`生成一个`Gemfile`文件，描述 gem 及其版本，`Cocoapods`就是其下一个普通的 gem。写完之后使用`bundle install`
Cocoapods：不再使用`pod install`，而是使用`bundle exec pod install`

推荐阅读冬瓜写的[Cocoapods历险记](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1477103239887142918&__biz=MzA5MTM1NTc2Ng==#wechat_redirect)

根据 `podspec` 文件打包成framework
- [cocoapods-pack](https://github.com/square/cocoapods-pack)
- [cocoapods-xcframework](https://github.com/TyrantDante/cocoapods-xcframework)
