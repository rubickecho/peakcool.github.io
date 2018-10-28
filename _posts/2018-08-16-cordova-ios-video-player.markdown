---
author: tangliangcheng
comments: true
date: 2018-10-28 22:33:00+00:00
layout: post
slug: cordova ios video player
title: Cordova iOS 多媒体播放器允许行内播放
excerpt: Cordova iOS 多媒体播放器允许行内播放
categories:
- 技术分享
---

## Cordova iOS 多媒体播放器允许行内播放

最近在cordova app上实现多媒体播放的功能时，遇到一个问题

```ios 默认不支持video行内播放```

google一番，搜集了关于这个问题的信息

1. h5 video标签禁止全屏播放
    ```在video标签中设置webkit-playsinline="true" playsinline="true"等值```

2. cordova-plugin-wkwebview-engine 上README提到

    ```The AllowInlineMediaPlayback preference will not work because of this Apple bug. This bug has been fixed in iOS 10.```


3. Apple 为了多媒体播放更好的用户体验，默认不允许行内播放，所以需要手动自己开启

    ```
    在cordova/config.xml中，新增
    <preference name="AllowInlineMediaPlayback" value="true" />
    ```