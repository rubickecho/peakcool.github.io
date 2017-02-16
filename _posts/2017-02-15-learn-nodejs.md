---
author: tangliangcheng
comments: true
date: 2017-02-15 22:10:34+00:00
layout: post
slug: learn-nodejs
title: 《7天学会NodeJs》笔（zou）记（ma）整（guan）理（hua）
excerpt: 最近阅读《7天学会NodeJs》，整理了一些知识点笔记。
categories:
- 技术分享
---

## 第一天 (认识node)

### 什么是NodeJs
js是脚本语言，脚本语言都需要一个解析器才能运行。对于写在html页面里的js，浏览器充当了解析器的角色。而对于需要独立运行的js，nodejs就是一个解析器。

### 模块(CMD模块系统)

将代码模块化:一般将代码合理拆分到不同的js文件中，每一个文件就是一个模块，而文件路径就是模块名

在编写每个模块时，都有`require`,`exports`,`module`三个预先定义好的变量可供使用

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

    nodejs定义了一个特殊的node_modules目录用于存放模块。例如执行模块的绝对路径是`/home/user/hello.js`，在该模块中使用`require('foo/bar')`方式加载模块时，node会依次尝试以下路径。
    
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
 
 在 *nix系统下，监听1024以下的端口需要root权限，因此，如果想要监听80或443端口的话吗需要使用sudo命令启动程序。
 
 简单例子:
 
    var http = require('http');
        http.createServer(function (request, response) {
        response.writeHead(200, { 'Content-Type': 'text-plain' });
        response.end('Hello World\n');
     }).listen(8124);
 
 
 ### HTTP
 
 'http'模块提供两种使用方式：
 
 1. 作为服务端使用时，创建一个HTTP服务器，监听HTTP客户端请求并返回响应
 2. 作为客户端使用时，发起一个HTTP客户端请求，获取服务端响应

    #### 服务端
    
    * createdServer 创建一个服务器
    * listen 监听端口

    HTTP请求本质上是一个数据流，由请求头(headers)和请求体(body)组成
    HTTP响应本质上是一个数据流，由响应头(headers)和响应体(body)组成
    
### HTTPS

`https` 模块与`http`模块极为类似，区别在于`https`模块需要额外处理SSL证书

### URL

处理http请求`url`模块使用率超高，该模块允许解析url，生成url,以及拼接url

url的各组成部分：

                               href
     -----------------------------------------------------------------
                                host              path
                          --------------- ----------------------------
     http: // user:pass @ host.com : 8080 /p/a/t/h ?query=string #hash
     -----    ---------   --------   ---- -------- ------------- -----
    protocol     auth     hostname   port pathname     search     hash
                                                    ------------
                                                       query
                                                       
#### 常用

* parse 将url字符串转换为对象
* format 将url对象转换为url字符串
* resolve 拼接url
* querystring 模块用于实现url参数字符串与参数对象的相互转换
* zlib模块提供了数据压缩和解压的功能
* net模块可用于创建socket服务器或socket客户端

### 小结

* `http` 和 `https`模块支持服务端模式和客户端模式两种使用方式
* `request` 和 `response` 对象除了用于读写头数据外，都可以当作数据流来操作
* `url.parse`方法加上`request.url`属性是处理http请求时的固定搭配
* 使用`zlib`模块可以减少使用http协议时的数据传输量
* 通过`net` 模块的socket服务器与客户端可对http协议做底层操作

---


## 第五天 （进程管理）

#### process

任何一个进程都有启动进程时使用的命令行参数，有标准输入和标准输出，有运行权限，有运行环境和运行状态。在nodejs中，可以通过`process`对象感知和控制node自身进程的方方面面。另外，`process`不是内置模块，而是一个全局对象，因此在任何地方都可以直接使用。

* 获取命令行参数: `process.argv`，node执行程序路径和主模块文件路径占据了`argv[0]`和`argv[1]`两个位置，所以第一个命令行参数从`argv[2]`开始。

        function main(argv) {
            //...
        }
        main(process.argv.slice(2))
  
* 退出程序: `process.exit()`
* 降权：在*nix系统下，使用root权限才能监听1024以下端口，一旦完成监听后，继续上程序运行在root权限下存在安全隐患，所以需要降权。
* 创建进程 `spawn(exec, args, options) ` 

    1. exec 执行文件路径
    2. args 每个成员按顺序对应一个命令行参数
    3. options 可选，配置进程的执行环境和行为
    
* 访问进程

    1. stdout
    2. stderr

### 总结

* 使用process对象管理自身
* 使用child_process模块创建和管理子进程

---


## 第六天 （异步编程）

### 回调

在代码中，异步编程的直接体现就是回调，异步编程依托于回调来实现，但不能说使用了回调后程序就异步化了。

JS本身是单线程运行的，不可能在一段代码还没结束时去运行别的代码，因此不存在异步执行的概念。但是如果某一个函数做的事情就是创建一个别的线程或进程，并与js主线并行地做一些事情，并在事情结束后通知js主线程，那么情况就是异步了。

js在执行完一段代码之前无法执行包括回调函数在内的别的代码，也就是说，即使平行线程完成工作了，通知js主线程执行回调函数了，回调函数也要等到主线程空闲时才能执行。

把单独一个回调函数理解为同步函数，使用try catch冒泡捕获异常并抛出，这样也就解释了为什么node.js 异步函数回调函数的第一个参数都是err


### 小结

* 不掌握异步编程就不算学会了nodejs
* 异步编程依托于回调来实现，而使用回调不一定是异步编程
* 异步编程下的函数间数据传递、数组遍历和异常处理与同步编程有很大区别
* 使用`domain`模块简化异步代码的异常处理，并小心陷阱
    
