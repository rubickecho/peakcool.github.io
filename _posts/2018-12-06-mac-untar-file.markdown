---
author: tangliangcheng
comments: true
date: 2018-12-06 11:15:00+00:00
layout: post
slug: Mac解压rar文件
title: Mac解压rar文件
excerpt: HomeBrew unrar
categories:
- 技术分享
---

## Mac解压tar文件

客户呼啦一下，一言不合扔过来一堆rar压缩文件，自己现MAC上没有对应的解压软件

google一番，发现homebrew可以安装对应的工具命令行解压

```shell
$ brew install unrar
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 2 taps (homebrew/core and homebrew/cask).
==> Updated Formulae
babel                             jenkins                           jenkins-lts                       macvim                            taskell                           webpack

==> Downloading https://homebrew.bintray.com/bottles/unrar-5.6.8.mojave.bottle.tar.gz
######################################################################## 100.0%
```

安装成功后，执行

```
cd rarFileParentPath

unrar x rarFileName
```

