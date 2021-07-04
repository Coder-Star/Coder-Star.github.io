---
title: Mac 效率软件
category:
  - 装机必备
tags:
  - 装机必备
date: 2021-01-09 08:51:20
---

## 前言

使用 Mac 开发也有几个年头了，积累了一些效率工具和开发工具，今天整理了一下并分享给大家，也期待大家有更多好的工具推荐给我，我补充上去。

> 点击软件名称跳转后软件相关链接，因为这些软件大部分都可以通过标题背后的相关链接都能找到安装教程，本文就不涉及安装教程的描述了。如果大家没有找到软件的相应链接，也可以私信我。

## 包管理器

### [Homebrew](https://brew.sh/)

Homebrew 是一款 Mac OS 平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。算是 Mac 系统的必备环境了。

有了它，比如你要下载下面提到的 node 环境，你根本不用考虑 node 去哪个地方下，只需要执行`brew install node`命令就好。

### [Npm](https://nodejs.org/en/)

Npm 其实是 Node.js 的包管理工具，安装 Node 后就会有 npm 环境了。有很多 npm 包是很好的工具，以我经常用的一个举例吧

#### [anywhere](https://www.npmjs.com/package/anywhere)

它可以随时随地将你的当前目录变成一个静态文件服务器的根目录，只需要你在当前目前下执行一个`anywhere`命令。

这样就实现了**一个局域网**下，文件互传的功能，我经常使用它来和同事之间传递文件，毕竟内网传递速度就是快。

### [Gem](https://rubygems.org/)

Gem 是 Ruby 模块的包管理器。如果你是 iOS 开发者，对这个一定不会陌生，因为 CocoaPods 本身就是一个 ruby 模块，我们可以通过 gem 来安装 CocoaPods，当然还可以通过 Homebrew 来安装。

## 日常工具

### [Snipaste](https://zh.snipaste.com/)

最好用的截图工具，我要向大家强烈安利它，不仅有正常的截图、编辑等功能，还有一个其他软件都没有而且我经常用的功能 -- 贴图，可以直接将图片像便签一样贴在桌面上。

![MWeb](../../../img/杂项/效率软件/Snipaste.png)

### [MWeb](https://zh.mweb.im/)

专业的 Markdown 写作、记笔记、静态博客生成软件，用起来真的比较方便，其实还有会朋友推荐 Typora 这款软件，但是我不太喜欢那种预览区和编辑区在一起的方式，如果对 Typora 有兴趣的，也可以去看看。

![MWeb](../../../img/杂项/效率软件/MWeb.png)

### [Go2Shell](https://zipzapmac.com/Go2Shell)

Go2Shell 可以让 Finder 中打开一个指向当前目录的终端窗口。

![MWeb](../../../img/杂项/效率软件/Go2Shell.png)

### [Parallel Desktop](https://www.parallels.cn/)

Mac 上的虚拟机软件，有的软件没有 Windows 版本，或多或少需要一个虚拟机安装其他系统。

我有的时候会通过这种方式从 Mac 电脑向 Mac 不支持写的硬盘中拷贝文件。

### [Mircrosoft Remote Desktop](https://www.microsoft.com/en-us/download/details.aspx?id=50042)

微软官方免费远程桌面控制 Windows 的软件，我之所以用这款软件，是因为我上家公司服务器系统是 Windows Server 的，如果也有类似需求或者需要远程 Windows 系统的读者，可以看看这款软件。

### [Remote Desktop - VNC](https://apps.apple.com/cn/app/remote-desktop-vnc/id472995993?mt=12)

远程连接 Mac 的工具。我只所以用这款软件，是因为我前不久需要连接 Mac Mini 做一些 iOS 自动化打包的事情，有类似需求的读者，可以看看这款软件。

### [Stretchly](https://github.com/hovancik/stretchly)

这是一款休息时间提醒应用，非常适合我们程序员这类写 Bug 时聚精会神，忘记起来活动活动的职业。
![Stretchly](../../../img/杂项/效率软件/Stretchly.png)

### [Alfred](https://www.alfredapp.com/)

这个我觉得根本无需介绍，神器，使用 macOS 的同学应该都知道。一句话来说就是，Alfred 是 macOS 上神级的效率应用，能够在实际操作中大幅提升工作效率。

## 开发工具

### [Sourcetree](https://www.sourcetreeapp.com/)

Sourcetree 是 Git

### [Charles]()

### [Navicat Premium]()

 数据库管理工具

### [Yummy FTP Pro]()

### [iTerm2]()

### [Postman]()

### [FinalShell]()

## iOS 工具

### [JSONConverter](https://github.com/iosyaowei/JSONConverter)

JSONConverter 是 MAC 上 iOS/Flutter 开发的辅助工具，可以快速的格式化 JSON 数据并转换生成对应的模型类属性，目前支持 Objective-C、Swift、Flutter 以及目前流行的第三方库：SwiftyJSON、HandyJSON，ObjectMapper, 可以灵活选择构建 class/struct，并支持配置类名前缀等，省去手敲模型的麻烦，借此提高开发效率。

![JSONConvert](../../../img/杂项/效率软件/JSONConvert.png)

### [LSUnusedResources](https://github.com/tinymind/LSUnusedResources)

用于在 Xcode 项目中查找未使用的图像和资源。
![LSUnusedResourcesExample](../../../img/杂项/效率软件/LSUnusedResourcesExample.gif)

### [BuildTimeAnalyzer](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)

展示 Swift 编译构建时间。
![BuildTimeAnalayer](../../../img/杂项/效率软件/BuildTimeAnalayer.png)

### [ImageOptim](https://imageoptim.com/mac):

压缩图片
![MWeb](../../../img/杂项/效率软件/ImageOptim.png)

### [Lookin](https://lookin.work/)

Lookin 可以查看与修改 iOS App 里的 UI 对象，类似于 Xcode 自带的 UI Inspector 工具，或另一款叫做 Reveal 的软件。但借助于“控制台”和“方法监听”功能，Lookin 还可以进行 UI 之外的调试。此外，虽然 Lookin 主体是一款 macOS 程序，它亦可嵌入你的 iOS App 而单独运行在 iPhone 或 iPad 上。最后，Lookin 完全免费。
![Lookin](../../../img/杂项/效率软件/Lookin.jpeg)

### [LinkMap](https://github.com/huanxsd/LinkMap)

这个工具是专为用来分析项目的 LinkMap 文件，得出每个类或者库所占用的空间大小（代码段 + 数据段），方便开发者快速定位需要优化的类或静态库。
![LinkMap](../../../img/杂项/效率软件/LinkMap.png)

### [SwiftFormat For Xcode](https://github.com/nicklockwood/SwiftFormat)

SwiftFormat 是一个代码库和命令行工具，用于在 macOS 或 Linux 上重新格式化 Swift 代码。

### [Hopper](https://www.hopperapp.com/)

逆向工程工具，可让您反汇编、反编译和调试应用程序。

![Hopper](../../../img/杂项/效率软件/Hopper.jpeg)

### [iTools](https://pro.itools.cn/pro_mac/)

这个只要是做 iOS 开发的应该都不知道，我就不过多介绍了。

### [Network Link Conditioner](https://developer.apple.com/downloads/?q=Hardware%20IO%20Tools)

### [XSimulatorMngr](https://github.com/xndrs/XSimulatorMngr)

这是一个来自苹果官方的工具，它可以模拟任何网络环境，如3G，Edge等等，也可以重新定义当前的网络环境，如网络延迟、带宽或丢包率。

![XSimulatorMngr](../../../img/杂项/效率软件/XSimulatorMngr.png)

## 在线工具

### [JSON](https://www.json.cn/)

JSON 解析，用来格式化 JSON

### [tinypng](https://tinypng.com/)

在线压缩图片

### [tableconvert](https://tableconvert.com/)

将表格转成 md，excel 等各种形式，我经常会用来写一些表格用来转成 md

### [DownGit](https://minhaskamal.github.io/DownGit/#/home)

下载 Github 指定文件

## Mac 软件网站

* [xclient](https://xclient.info/)
* [macbl](https://www.macbl.com/)
* [xxmac](https://www.xxmac.com/)
