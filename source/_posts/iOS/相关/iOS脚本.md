---
title: iOS 脚本
date: 2020-02-18 18:53:07
categories: [iOS]
tags: [iOS]
---

## 1、Xcode 编译时获取版本号并展示到 storyboard 中

```swift
# 注意：Xcode 11与Xcode 10环境变量有变化
# Xcode 11
version=${MARKETING_VERSION}
# Xcode 10
if [ -z ${version} ]; then
    version=$(/usr/libexec/PlistBuddy -c "Print :CFBundleShortVersionString" "${PROJECT_DIR}/${INFOPLIST_FILE}")
fi
sed -i bak -e "/userLabel=\"versionLb\"/s/text=\"[^\"]*\"/text=\"版本号：$version\"/" $PROJECT_DIR/$PROJECT_NAME/Base.lproj/LaunchScreen.storyboard
```
