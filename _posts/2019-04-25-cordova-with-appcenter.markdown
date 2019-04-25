---
author: tangliangcheng
comments: true
date: 2019-04-25 21:17:00+00:00
layout: post
slug: cordova with appcenter
title: Cordova Appcenter 应用整理
excerpt: cordova with appcenter
categories:
- 技术分享
---

## Cordova Appcenter 应用整理

Cordova codepush 借助appcenter平台，以下是一些备忘整理。

### Appcenter

1. 安装appcenter
    ```shell
    npm install -g appcenter-cli 
    ```
2. https://appcenter.com/ 注册账号
3. appcenter cli 登录(粘贴浏览器中的字符串)
    ```shell
    appcenter login
    ```
4. 在appcenter中创建应用
5. 创建deployment key
    ```shell
    appcenter codepush deployment add -a <groupName>/<appname> Staging //测试环境

    appcenter codepush deployment add -a <groupName>/<appname >Production //开发环境
    ```

### Cordova

1. 创建cordova 项目
2. cd cordova project
3. 添加android平台
4. 添加codepush、appcenter相关插件
    ```shell
    cordova plugin add cordova-plugin-code-push@latest
    cordova plugin add cordova-plugin-appcenter-analytics
    cordova plugin add cordova-plugin-appcenter-crashes
    ```
5. config.xml中添加
    ```xml
    <platform name="android"> 
        <preference name="APP_SECRET" value="90a4efdb-547c-475e-92f0-4a20a1526852" />
        <preference name="CodePushDeploymentKey" value="YOUR-ANDROID-DEPLOYMENT-KEY" />
        <preference name="AndroidPersistentFileLocation" value="Compatibility" />
    </platform>
    <platform name="android">
    ```
6. www/index.js onDeviceReady 中添加

    ```js
    codePush.sync();
    ```
7. 打包apk应用安装
    ```
    cordova build android
    ```
8. 启动应用如图
9. 修改index.html
    ```html
    <div class="app">
        <h1>Apache Cordova-5</h1>
        <h2>Hello World</h2>
        <div id="deviceready" class="blink">
            <p class="event listening">Connecting to Device</p>
            <p class="event received">Device is Ready</p>
        </div>
    </div>
    ```
10. 发布到appcenter， 如图
    ```shell
    appcenter codepush release-cordova -a <groupName>/<appname> -t <version>
    ```
    
    ![](https://ws3.sinaimg.cn/large/006tNc79gy1g2f8dnnde1j31s40u0adt.jpg)

## 最终效果

启动应用，开启remote debuge模式，可以看到更新日志等

相关代码在 https://github.com/peakcool/cordova-codepush-example


### 其他命令

```
appcenter codepush deployment list -a <groupName>/<appname> //查看keys

appcenter codepush rollback <groupName>/<appname> <deploymentName> //回滚版本

appcenter codepush patch -a <groupName>/appname Production -m

appcenter codepush patch -a <groupName>/appname Production v23 -rollout 50%
```

### 参考网址

https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cordova

https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cli

https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cli
