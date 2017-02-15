---
author: tangliangcheng
comments: true
date: 2017-02-15 22:10:34+00:00
layout: post
slug: learn-nodejs
title: 《7天学会NodeJs》笔记整理
excerpt: 最近阅读《7天学会NodeJs》，整理了一些知识点笔记。
categories:
- 技术分享
---

## 第一天 (认识node)

### 什么是NodeJs
js是脚本语言，脚本语言都需要一个解析器才能运行。对于写在html页面里的js，浏览器充当了解析器的角色。而对于需要独立运行的js，nodejs就是一个解析器。

### 模块(CMD模块系统)

将代码模块化:一般将代码合理拆分到不同的js文件中，每一个文件就是一个模块，而文件路径就是模块名

在编写每个模块时，都有require,exports,module三个预先定义好的变量可供使用

* require 加载和使用别的模块，传入一个模块名，返回一个模块导出对象(.js可省略，.json不可省略)
* exports 当前模块的导出对象(方法和属性),别的模块通过require时就得到了当前模块的exports对象
* module 访问当前模块的信息，最多用途是替换当前模块的导出对象

### 模块初始化

一个模块中的js代码仅在模块第一次被使用时执行一次，并在执行过程中初始化模块的导出对象。之后，缓存起来的导出对象被重复使用。

---

## 第二天 （模块和包）

### 模块路径解析规则

* 内置模块
	
    如果传递给require函数的是nodejs内置模块名称，不做路径解析，直接返回内部模块的导出对象，例如require('fs')
    
* node_modules目录

	nodejs定义了一个特殊的node_modules目录用于存放模块。例如执行模块的绝对路径是/home/user/hello.js，在该模块中使用require('foo/bar')方式加载模块时，node会依次尝试以下路径。
    
    	/home/user/node_modules/foo/bar
        /home/node_modules/foo/bar
        /node_modules/foo/bar
        
* NODE_PATH 环境变量 自定义模块搜索路径

### 包 (package)

多个js文件 -> 子模块 -> 包

### npm常用命令

* 在 `package.json` 所在目录下使用 `npm install . -g` 可先在本地安装当前命令行程序，可用于发布前的本地测试
* `npm update package` 可以把当前目录下node_modules子目录下对应模块更新至最新版本
* `npm cache clear` 可以清空npm本地缓存

---

## 第三天 （文件操作）

### 重要概念

* Buffer（数据块）
* Stream（数据流）  当内存中无法一次装下需要处理的数据时，或者一边读取一边处理更加高效时，就需要用到数据流
* File System（文件系统）
* Path（路径） NodeJS提供了path内置模块简化路径相关操作
* 文本编码 在读取不同编码的文本文件时，需要将文本内容转换为js使用的`utf8`编码字符串后才能正常使用

#### 文件操作

NodeJS通过fs内置模块提供对文件的操作。fs模块提供的API 可以分为以下三类:

* 文件属性读写 常用: `fs.stat`、`fs.chmod`、`fs.chown`
* 文件内容读写 常用: `fs.readFile`、`fs.readdir`、`fs.writeFile`、`fs.mkdir`
* 底层文件操作 常用: `fs.open`、`fs.read`、`fs.write`、`fs.close`

#### 路径操作

* `path.normalize` 将传入的路径转换为标准路径，win下为`\`，*nix下为`/`
* `path.join` 将多个路径拼接为标准路径
* `path.extname` 获取文件扩展名 

#### 遍历目录（一般使用递归算法）	

	function factorial(n) {
    	if (n === 1) {
        	return 1;
        } else {
        	return n * factorial(n - 1)
        }
    }
    
    性能更高的方式：
    
    	  A
         / \
        B   C
       / \   \
      D   E   F
      
    这棵树的遍历顺序是A > B > D > E > C > F。
    
#### 同步遍历（异步遍历与同步遍历远离相同）

	var fs = require('fs')
    var path = require('path')

    var dir = process.argv.slice(1)

    function travel(dir, callback) {
        fs.readdirSync(dir).forEach(function(file){
            var pathname = path.join(dir, file)

            if (fs.statSync(pathname).isDirectory()) {
                travel(pathname, callback)
            } else {
                callback(pathname)
            }
        })
    }

    travel(dir[1], function(pathname){
        console.log(pathname);
    })
    
 执行命令 
 
 	➜  fileAction node travel.js ./
    bigcopy.js
    copy.js
    database.sql
    demo.txt
    dst/databast-copy.spl
    dst/demo2.txt
    travel.js

	

### 例1: 小文件拷贝

	var fs = require('fs');
	
    //读取和写入文件
    function copy(src, dst) {
        fs.writeFileSync(dst, fs.readFileSync(src));
    }
	
    function main(argv) {
        copy(argv[0], argv[1]);
    }
	
    //获取命令参数(路径)
	main(process.argv.slice(2));
    
    
 执行命令 `node copy.js ./demo.txt ./dst/demo-copy.txt`
 
    
### 例2: 大文件拷贝

	var fs = require('fs');

    function copy(src, dst) {
        fs.writeFileSync(dst, fs.readFileSync(src));
    }

    function main(argv) {
        copy(argv[0], argv[1]);
    }

    main(process.argv.slice(2));
   
执行命令 `node bigcopy.js ./database.sql ./dst/database-copy.sql`


### 小结

* 学好文件编程，编写各种程序都不怕
* 如果不是很在意性能，`fs`模块的同步api能让生活更美好
* 需要对文件读写做到字节级别的精细控制时，请使用`fs`模块的文件底层操作api
* 不要使用拼接字符串的方式来处理路径，使用`path`模块
* 掌握好目录遍历和文件编码处理技巧，很实用

--- 
## 第四天 （网络操作）
 