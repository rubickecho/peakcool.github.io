---
author: tangliangcheng
comments: true
date: 2020-10-27 13:20:00+00:00
layout: post
slug: flutter升级后问题：Bad state Insecure HTTP is not allowed by platform
title: flutter升级后问题：Bad state Insecure HTTP is not allowed by platform
excerpt: 最近升级了flutter后，旧项目在build后，出现了Bad state Insecure HTTP is not allowed by platform问题，解决方案如下
categories:
- 技术分享
---

# Bad state Insecure HTTP is not allowed by platform

最近升级了flutter后，旧项目在build后，出现了Bad state Insecure HTTP is not allowed by platform问题，解决方案如下:

## Android平台

andorid/app/src/main/res/xml/network_security_config.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
	<base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

andorid/app/src/main/AndroidManifest.xml

```xml

add "android:networkSecurityConfig="@xml/network_security_config"

<application
        android:name="io.flutter.app.FlutterApplication"
        android:label="Project_name"
		android:networkSecurityConfig="@xml/network_security_config"
        android:icon="@mipmap/ic_launcher">
        <activity
            android:name=".MainActivity"
            android:launchMode="singleTop"
```

## iOS平台

ios/Runner/info.plist
```swift
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

## 为什么会出现这个问题?

详见文章:
[Solving the new HTTPS requirements in Flutter](https://medium.com/flutter-community/solving-the-new-https-requirements-in-flutter-7abe240fbf23)

特别注意，

## 其他参考

[github issues](https://github.com/flutter/flutter/issues/66275)