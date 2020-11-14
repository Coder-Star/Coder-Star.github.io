---
title: iOS证书相关
date: 2019-10-21 19:52:42
categories: [iOS]
tags: [iOS证书]
---

#### 检查 App 证书过期时间信息

```ios
unzip -q XXXXXXXXX.ipa
codesign -d --extract-certificates Payload/*.app
openssl x509 -inform DER -in codesign0 -noout -nameopt -oneline -dates //获取签名有效日期

codesign -dv --verbose=4 Payload/*.app //获取签名详细信息
```

#### mobileprovision 文件位置

/Users/XXXXXXXXX/Library/MobileDevice/Provisioning Profiles
