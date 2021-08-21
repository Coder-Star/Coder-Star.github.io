---
title: 以 OTA 方式无线分发 iOS 安装包
date: 2020-03-26 19:49:05
categories:
  - iOS
  - 相关
tags: [分发]
---

## 安装原理

OTA 即`Over-the-Air`，是 Apple 在 iOS4 中新加的一项技术，目的是让开发者能够脱离 Appstore，实现从服务器下载并安装 iOS 应用，用户只需要在 iOS 的浏览器（WebKit）中，打开`itms-services://`协议链接，就可以直接安装 APP。iOS 浏览器内核会读取.plist 文件中的信息，包括应用的名称、版本、ipa 包下载地址等。

**这种方式只适合用来分发企业账号应用**

## 安装步骤

我们只要在浏览器中使用 a 标签打开下面地址或者在 iOS 应用中使用`UIApplication.shared.open()`来打开这个地址便可以开始进行下载了。

```
itms-services://?action=download-manifest&url=https://gitee.com/CoderStar/Test/manifest.plist
```

url 地址便是.plist 文件地址，苹果要求这个链接必须是 https 链接，我们可以有以下几种选择。

- 将 plist 放在代码托管平台上，如 github，gitlab 等，但因为访问速度的问题，我建议将其放在 gitee 上
- 自己搭建 https 网站，更加安全，不会将 plist 文件暴露出来从而暴露出 ipa 的下载地址

## 附录 manifest.plist

plist 文件格式必须为 UTF-8，中各部分参数解释如下

- URL：应用 (.ipa) 文件的完全限定 HTTPS URL，iOS 14 之前可以使用 HTTP，iOS 14 必须是 HTTPS
- display-image：57 x 57 像素的 PNG 图像，在下载和安装过程中显示。指定图像的完全限定 URL
- full-size-image：512 x 512 像素的 PNG 图像，表示 iTunes 中相应的应用
- bundle-identifier：应用的包标识符，与 Xcode 项目中指定的完全一样
- bundle-version：应用的包版本，在 Xcode 项目中指定
- title：下载和安装过程中显示的应用的名称

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>http://www.example.com/apps/app.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>url</key>
					<string>http://www.example.com/apps/image.57x57.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>url</key>
					<string>http://www.example.com/apps/image.512x512.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>com.example.test</string>
				<key>bundle-version</key>
				<string>1.0.0</string>
				<key>kind</key>
				<string>software</string>
				<key>platform-identifier</key>
				<string>com.apple.platform.iphoneos</string>
				<key>title</key>
				<string>测试APP</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>

```
