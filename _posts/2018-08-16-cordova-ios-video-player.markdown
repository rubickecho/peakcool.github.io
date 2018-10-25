---
author: tangliangcheng
comments: true
date: 2018-10-16 22:33:00+00:00
layout: post
slug: learn node 08 day
title: Cordova Android 扫码返回按钮路由返回上一级
excerpt: Cordova Android phonegap-plugin-barcodescanner 插件扫码返回按钮路由返回上一级
categories:
- 技术分享
---

## Cordova Android 扫码返回按钮路由回退

最近在实现cordova 扫码需求时，用到了`phonegap-plugin-barcodescanner`库

开启相机后，在android上，没有`取消扫码`按钮，只能点击`返回`按钮才能取消扫码

这时会有一个问题，点击返回按钮时，webview路由会`go(-1)`并且取消扫码关闭相机

而实际期望的是关闭相机，路由不返回上一级

在vue项目中解决思路是：

```
1. vuex中创建一个值 state.unBack = false作为临时保存扫码开启状态
2. 开启扫码时，设置 state.unBack = true
3. 在当前页路由勾子 beforeRouteLeave 中，判断unBack是否会false, false时正常路由行为， true时next(false)阻止路由跳转
```