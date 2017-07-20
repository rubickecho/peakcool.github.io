---
author: tangliangcheng
comments: true
date: 2017-07-16 23:00:34+00:00
layout: post
slug: learn node 01 day
title: 《深入浅出NodeJs》学习笔记 01 day
excerpt: 包与NPM、AMD,CMD规范介绍
categories:
- 技术分享
---

## 包与NPM

### 包结构

**完全符合CommonJs规范的包目录应该包含如下文件：**

* package.json: 包描述文件
* bin: 用于存放可执行二进制文件的目录
* lib: 用于存放JavaScript代码的目录
* doc: 用于存放文档的目录
* test: 用于存放单元测试用例的代码

**CommonJs package.json必须字段：**

* name: 包名
* description: 包简介
* vsersion: 版本号，一般为`major.minor.revision`格式
* keywords: 关键词数组
* maintainers: 包维护者列表
* contributors: 贡献者列表
* bugs: 一个可以反馈bug的地址
* licenses: 许可证列表
* repositories: 托管源代码的位置列表
* dependencied: 当前包所需依赖的列表
* homepage: 当前包的网站地址
* os: 操作系统支持列表
* cpu: cpu架构支持列表
* engine: 支持的javascript引擎列表
* builtin: 标志当前包是否是内建在底层系统的标准组件
* directores: 包目录说明
* implements: 实现规范的列表
* scripts: 让包安装或者卸载等过程中提供钩子机制
***npm 补充：*** author, bin, main, devDependencies

### 发布包

```shell
1.创建文件夹 npm-package-dev
mkdir npm-package-dev

2.进入文件夹
cd npm-package-dev

3.初始化npm包
npm init

4.注册仓库账号
npm adduser

5.上传包
npm publish

6.执行 npm install <name> 安装包，其中name为npm init时填写的name字段
```

### 如何选择好的包

* 具备良好的测试
* 具备良好的文档
* 具备良好的测试覆盖率
* 具备良好的编码规范
* 更多条件

## 前后端公用模块

* AMD规范

> 定义： define(id?, dependencies?, factory)

id 和 dependencies可选，factory为模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。如果是对象，此对象应该为模块的输出值。

示例：

```js
define(function() {
    var exports = {}
    exports.sayHello = function() {
        console.log('Hello, World')
    }

    return exports
})
```

* CMD规范

> 定义： define(factory)

与AMD相比，AMD需要在声明时指定所有依赖，CMD支持动态引入

示例：

```js
define(function( require, exports, modules) {
    // The module code goes here
})
```
