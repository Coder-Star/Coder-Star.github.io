---
title: iOS 自动打包
category:
  - iOS
  - 提效
tags:
  - iOS
  - 提效
date: 2021-08-19 20:27:18
---

## 前言

Hi Coder，我是 CoderStar！

测试阶段一般会发生这样的场景，测试拼命的提 Bug，开发拼命的改 Bug，改完重新打包发给测试进行复测，那这个过程中频繁的打包肯定是不可避免的。

如果使用 Xcode 打包，在打包期间我们是无法改剩余的 Bug 或进行其他模块的开发的，那这个时候我们能干什么呢？

哈哈哈，当然是去接杯咖啡或者泡杯茶了，不然还能干啥？

## 自动打包

好了，言归正传，其实这个打包过程我们可以脱离 Xcode，改用`xcodebuild`命令进行打包，相关核心命令包括：

* `xcodebuild clean`
* `xcodebuild archive`
* `xcodebuild -exportArchive`

详细脚本见下节。

> 如果有需要帮助的，可以通过公众号联系我。

一般自动打包都会专门使用一台 Mac 作为打包机（一般是 Mac Mini，大厂会有专门的打包集群），在打包机上安装 `jenkins` 用来做自动化构建，关于 `jenkins` 这块我就不展开讲了，有兴趣的可以去查阅相关资料。我简单讲下中间的流程：

1. 开发者提交代码到 `gitlab`；
2. 可在 `gitlab` 配置相应的触发条件，如 `push`、`tag` 等，满足触发条件会发送 `webhook` 消息到 `jenkins`（`webhook`地址是提前在 `jenkins` 中配置好的）；
3. `jenkins` 收到通知后，就会执行配置好的构建任务；
4. 构建任务内部拉取最新代码，进行一系列操作，如根据 `jenkins` 任务参数修改代码中的一些参数等，最后进行打包
5. 打包成功后，将安装包上传到分发平台（蒲公英等外部平台或者自研的内部平台），上传成功后便可以将下载链接等相关信息通过 `webhook` 发送到企业微信群、钉钉群等团队沟通工具中，通知相关人员打包成功。

![iOS自动化打包流程](../../../img/iOS/提效/ios%20package.jpeg)

这套体系搭建完成之后，对于开发人员而言打包就是修改 Bug，push 代码了。

> 上面只是一条简单的自动化打包流程，其实中间涉及的很多点没有展开，特别打包数量上了一定量级之后。

那有的读者所在公司可能没有专门的打包机，那这种情况我觉得就没有必要在自己的机器上再安装 jenkins 了，直接在终端执行脚本就可以了。步骤如下：

1. 建立新的打包目录，其中包含源代码、打包脚本以及打包生成文件等目录；(不要直接使用开发工程目录，否则打包的时候还是不可以修改代码）
2. 代码提交后，执行打包目录下的打包脚本，脚本内部需要添加拉取最新代码操作，拉取之后进行打包。

## 附录

该脚本只包含了 iOS 通用的打包步骤，大家可根据业务需求进行调整，如`git pull`拉取最新代码等操作。

如果复制不方便，也可以直接从[打包脚本地址](https://github.com/Coder-Star/CSPubicFile/blob/main/iOS/%E8%84%9A%E6%9C%AC/%E6%89%93%E5%8C%85%E8%84%9A%E6%9C%AC/package.sh)进行下载。

```shell
#!/bin/sh

# 注意事项
# 1、如果提示permission denied: ./package.sh ， 则先附加权限，命令如下：chmod 777 package.sh
# 2、请根据自己项目的情况选择使用 workspace 还是 project 形式，目前默认为 workspace 形式

### 需要根据自己项目的情况进行修改，XXX都是需要进行修改的，可搜索进行修改 ###

# Project名称
PROJECT_NAME=XXX

## Scheme名
SCHEME_NAME=XXX

## 编译类型 Debug/Release二选一
BUILD_TYPE=XXX

## 项目根路径，xcodeproj/xcworkspace所在路径
PROJECT_ROOT_PATH=XXX

## 打包生成路径
PRODUCT_PATH=XXX

## ExportOptions.plist文件的存放路径，该文件描述了导出ipa文件所需要的配置
## 如果不知道如何配置该plist，可直接使用xcode打包ipa结果文件夹的ExportOptions.plist文件
EXPORTOPTIONSPLIST_PATH=XXX


## workspace路径
WORKSPACE_PATH=${PROJECT_ROOT_PATH}/${PROJECT_NAME}.xcworkspace

## project路径
PROJECT_PATH=${PROJECT_ROOT_PATH}/${PROJECT_NAME}.xcodeproj

### 编译打包过程 ###

echo "============Build Clean Begin============"

## 清理缓存

## project形式
# xcodebuild clean -project ${PROJECT_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE} || exit

## workspace形式
xcodebuild clean -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -configuration ${BUILD_TYPE} || exit

echo "============Build Clean End============"

#获取Version
VERSION_NUMBER=`sed -n '/MARKETING_VERSION = /{s/MARKETING_VERSION = //;s/;//;s/^[[:space:]]*//;p;q;}' ${PROJECT_PATH}/project.pbxproj`
# 获取build
BUILD_NUMBER=`sed -n '/CURRENT_PROJECT_VERSION = /{s/CURRENT_PROJECT_VERSION = //;s/;//;s/^[[:space:]]*//;p;q;}' ${PROJECT_PATH}/project.pbxproj`

## 编译开始时间,注意不可以使用标点符号和空格
BUILD_START_DATE="$(date +'%Y-%m-%d_%H-%M')"

## IPA所在目录路径
IPA_DIR_NAME=${VERSION_NUMBER}_${BUILD_NUMBER}_${BUILD_START_DATE}

##xcarchive文件的存放路径
ARCHIVE_PATH=${PRODUCT_PATH}/IPA/${IPA_DIR_NAME}/${SCHEME_NAME}.xcarchive
## ipa文件的存放路径
IPA_PATH=${PRODUCT_PATH}/IPA/${IPA_DIR_NAME}

# 解锁钥匙串 -p后跟为电脑密码
security unlock-keychain -p XXX

echo  "============Build Archive Begin============"
## 导出archive包

## project形式
# xcodebuild archive -project ${PROJECT_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH}

## workspace形式
xcodebuild archive -workspace ${WORKSPACE_PATH} -scheme ${SCHEME_NAME} -archivePath ${ARCHIVE_PATH} -quiet || exit

echo "============Build Archive Success============"


echo "============Export IPA Begin============"
## 导出IPA包
xcodebuild -exportArchive -archivePath $ARCHIVE_PATH -exportPath ${IPA_PATH} -exportOptionsPlist ${EXPORTOPTIONSPLIST_PATH} -quiet || exit

if [ -e ${IPA_PATH}/${SCHEME_NAME}.ipa ];
    then
    echo "============Export IPA SUCCESS============"
    open ${IPA_PATH}
    else
    echo "============Export IPA FAIL============"
fi


# 删除Archive文件，可根据各自情况选择是否保留
# rm -r ${ARCHIVE_PATH}


### 上传过程 ###

## 上传app store

ALTOOL_PATH="/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool"

# 将-u 后面的XXX替换成自己的AppleID的账号，-p后面的XXX替换成自己的密码
# "$ALTOOL_PATH" --validate-app -f ${IPA_PATH}/${SCHEME_NAME}.ipa -u XXX -p XXX -t ios --output-format xml

# "$ALTOOL_PATH" --upload-app -f ${IPA_PATH}/${SCHEME_NAME}.ipa -u  XXX -p XXX -t ios --output-format xml

## 上传到蒲公英
echo "============Upload PGYER Begin============"

## 具体参数可见 http://www.pgyer.com/doc/view/api#uploadApp
PGYER_UPLOAD_RESULT=`curl \
-F "file=@${IPA_PATH}/${SCHEME_NAME}.ipa" \
-F "buildInstallType=XXX" \
-F "buildPassword=XXX" \
-F "buildUpdateDescription=XXX" \
-F "_api_key=XXX" \
https://www.pgyer.com/apiv2/app/upload`

echo "============Upload PGYER SUCCESS============"
## 返回结果码，其中0为成功上传，因为返回结果中带回来的有中文显示乱码，无法利用jq解析


## 如需上传到fim，可查阅 https://www.betaqr.com/docs/publish 文档


### 如需脚本在执行过程中给用户提供选择，可使用以下Demo ###

#echo "Place enter the number you want to export? [ 1:app-store 2:ad-hoc] "
#read number
#    while([[ $number != 1 ]] && [[ $number != 2 ]])
#    do
#        echo "Error! Should enter 1 or 2"
#        echo "Place enter the number you want to export ? [ 1:app-store 2:ad-hoc] "
#        read number
#    done
#if [ $number == 1 ];
#    then
#    ####
#    else
#    ####
#fi

```

## 最后

新的一周要更加努力呀！

Let's be CoderStar!
