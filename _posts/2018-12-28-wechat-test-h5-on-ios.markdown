---
author: tangliangcheng
comments: true
date: 2018-12-28 11:17:00+00:00
layout: post
slug: 微信网页ios兼容性调试
title: 微信网页ios兼容性调试
excerpt: Wechat ios compatibility test
categories:
- 技术分享
---

## 微信h5网页ios兼容性测试

微信公众号中，h5网页ios兼容性测试，主要借助`ios模拟器`和`safari浏览器`开发者模式。

#### step1 确认出现ui bug 的browser信息和硬件设备系统信息

![sentry平台日志监控]({{ site.url }}/images/posts/sentry_device_info.jpg)

由browser可以看到，ios上微信中打开网页浏览器内核默认是用的 `Safari UI/WKWebView`

那么也就是说，我们可以借助对应版本的ios模拟器，然后访问h5网页来做兼容性测试。

#### step2 安装并启动对应版本ios模拟器

1. 打开`Xcode`, 进入`Preferences -> Components`选项，安装上步获取到的对应os版本

![xcode安装模拟器系统]({{ site.url }}/images/posts/xcode_install_os.png)

2. 打开`Simulator`, 启动对应型号和系统的模拟器

3. 模拟器打开Safari，访问H5界面


#### step3 Mac Safari Debug

MAC打开Safari, `开发选项 -> 模拟器 -> JSContext` 开始调试，复现问题

![safari web debug]({{ site.url }}/images/posts/safari_debug.png)