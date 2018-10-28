---
author: tangliangcheng
comments: true
date: 2018-10-27 16:33:00+00:00
layout: post
slug: flutter gradle build
title: Flutter Gradle Run Build Failed
excerpt: Could not resolve all files for configuration 'classpath'.
categories:
- 技术分享
---
<!-- TOC -->

- [Flutter Gradle编译失败](#flutter-gradle编译失败)
    - [错误日志](#错误日志)
    - [解决办法](#解决办法)

## Flutter Gradle编译失败

### 错误日志

```
Launching lib/main.dart on Android SDK built for x86 in debug mode...
Initializing gradle...
Resolving dependencies...
* Error running Gradle:
Exit code 1 from: /Users/tangliangcheng/AndroidStudioProjects/flutter_app_demo01/android/gradlew app:properties:
Starting a Gradle Daemon (subsequent builds will be faster)

Project evaluation failed including an error in afterEvaluate {}. Run with --stacktrace for details of the afterEvaluate {} error.

FAILURE: Build failed with an exception.

* Where:
Build file '/Users/tangliangcheng/AndroidStudioProjects/flutter_app_demo01/android/app/build.gradle' line: 25

* What went wrong:
A problem occurred evaluating project ':app'.
> Could not resolve all files for configuration 'classpath'.
   > Could not find lint-gradle-api.jar (com.android.tools.lint:lint-gradle-api:26.1.2).
     Searched in the following locations:
         https://jcenter.bintray.com/com/android/tools/lint/lint-gradle-api/26.1.2/lint-gradle-api-26.1.2.jar

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 10s

Finished with error: Please review your Gradle project setup in the android/ folder.

```

### 解决办法

flutter⁩ ▸ ⁨packages⁩ ▸ ⁨flutter_tools⁩ ▸ ⁨gradle⁩
```
buildscript {
    repositories {
	google() //添加
        jcenter()
        maven {
            url 'https://dl.google.com/dl/android/maven2'
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.2'
    }
}
```