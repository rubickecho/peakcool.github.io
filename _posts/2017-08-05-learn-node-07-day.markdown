---
author: tangliangcheng
comments: true
date: 2017-08-05 21:03:00+00:00
layout: post
slug: learn node 07 day
title: 《深入浅出NodeJs》学习笔记 07 day
excerpt: Node 构建基础Web应用
categories:
- 技术分享
---

## 构建Web应用

从http模块中服务器端的`request`事件开始分析。`request`事件发生于网络连接的建立，客户端向服务器端发送报文，服务器端解析报文，发送HTTP请求的报头时.

在具体的业务中，一般会有如下需求:

* 请求方法的判断
* URL的路径解析
* URL中查询字符串解析
* Cookie的解析
* Basic认证
* 表单数据的解析
* 任意格式文件的上传处理

---

### Cookie

Cookie的处理分为如下几步:

* 服务器向客户端发送Cookie
* 浏览器将Cookie保存
* 之后每次刷新浏览器都会将Cookie发向服务器端

#### Cookie参数

* `path`表示这个Cookie影响到的路径，当前访问的路径不满足该匹配时，浏览器不发送这个Cookie
* `Expires`和`Max-Age`是用来告知浏览器这个Cookie何时过期的，如果不设置该选项，在关闭浏览器时会丢失掉这个Cookie。如果设置了过期时间，浏览器将会把Cookie内容写入到磁盘中保存，下次打开浏览器依然有效。`Expires`的值是一个UTC格式的时间字符串，告知浏览器此Cookie何时过期。`Max-Age`则告知浏览器多久后过期，当两个值同时存在时，且同时被支持时，`Max-Age`会覆盖`Expires`
* `HttpOnly`告知浏览器不允许通过脚本`document.cookie`去更改这个Cookie值，事实上，设置`HttpOnly`之后，这个值在`document.cookie`中不可见，但是在HTTP请求的过程中，依然会发送这个Cookie到服务器端
* `Secure` 当Secure值为true时，在HTTP中是无效的，在HTTPS中才有效，很难被窃听到

#### Cookie的性能影响

* 减少Cookie的大小
* 为静态组件使用不同的域名
* 减少DNS查询

---

### Session

#### Session与内存

为了解决性能问题和`Session`数据无法跨进程共享的问题，常用的方案是将Session集中化，将原本可能分散在多个进程里的数据，统一转移到集中的数据存储中。目前常用的工具是`redies`、`memcached`等。通过这些高效的缓存，node进程无须在内部维护数据对象，垃圾回收问题和内存限制问题都迎刃而解

---

#### Session与安全

典型的`XXS`(跨站脚本攻击 Cross Site Scripting)拿到用户口令，实现伪造，通常都是由网站开发者决定哪些脚本可以执行在浏览器端，不过`XXS`漏洞会让别的脚本执行。它的主要形成原因多数是用户的输入没有被转义，而被直接执行。

---

### 缓存

#### 使用缓存流程图

![使用缓存流程图](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E6%9E%84%E5%BB%BA%E5%9F%BA%E7%A1%80web%E5%BA%94%E7%94%A8/%E4%BD%BF%E7%94%A8%E7%BC%93%E5%AD%98%E7%9A%84%E6%B5%81%E7%A8%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

#### 缓存设置

#### 清除缓存

知晓了如何设置缓存，以达到节省网络带宽的目的，但是缓存一旦设定，当服务器端意外更新内容时，却无法通知客户端更新。这使得我们在使用缓存时也要为其设定版本号，所幸浏览器根据URL进行缓存，那么一旦内容更新时，我们就让浏览器发起新的url请求，使新内容能够被客户端更新。一般的更新机制有如下两种：

* 每次发布，路径中跟随web应用的版本号： http://url.com/?v=20170801
* 每次发布，路径中跟随该文件内容的hash值： http://url.com/?hash=afadsad11sad

一般来说根据文件内容的hash值进行缓存淘汰更加高效，因为文件内容不一定跟随web应用的版本而更新，而内容没有更新时，版本号的改动导致更新毫无意义，因此以文件内容形成的hash值更精确

---

### 数据上传

Node的http模块只对HTTP报文的头部进行了解析，然后触发request事件。如果请求中还带有内部部分（如POST请求，它具有报头和内容），内容部分需要用户自行接收和解析。通过设置报头的`Transfer-Encoding`或`Content-Length`即可判断请求中是否带有内容。

#### 表单数据

默认的表单提交，请求头中的`Content-Type`字段值为`application/x-www-form-urlencoded`

#### 其他格式

常见的提交还有JSON和XML文件等，判断和解析他们的原理都比较相似，都是依据`Content-Type`的值决定，其中JSON类型的值为application/json,XML的值为application/xml

需要注意的是，在Content-Type中可能还会附带`charset=utf-8`的编码信息。

---

### 中间件与性能

#### 中间件的工作模型

![中间件的工作模型](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E6%9E%84%E5%BB%BA%E5%9F%BA%E7%A1%80web%E5%BA%94%E7%94%A8/%E4%B8%AD%E9%97%B4%E4%BB%B6%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%9E%8B.png?raw=true)

主要能提升性能的点：

* 编写高效的中间件
    提升单个中间件的处理速度，以尽早调用next()执行后续逻辑。需要知道的是，一旦中间件被匹配，那么每个请求都会使该中间件执行一次，哪怕它只浪费了1毫秒的执行时间都会让我们QPS显著下降，常见的优化方法有几种。

    * 使用高效的方法，必要时通过jsperf.com测试基准性能
    * 缓存需要重复计算的接口

* 合理利用路由，避免不必要的中间件执行

---

## 页面渲染

### MIME

`MIME`的全称是`Multipurpose Internet Mail Extensions`,从名字可以看出，它最早用于电子邮件，后来也应用到浏览器中。不同的文件类型具有不同的`MIME`值，如JSON文件的值为`application/json`，XML文件的值为`application/xml`，PDF文件的值为`application/pdf`。

Node社区中`mime`模块可以判断文件类型。

```js
var mime = require('mime')

mime.lookup('/path/to/file.txt') //=> 'text/plain'
mime.lookup('file.txt') //=> 'text/plain'
mime.lookup('.TXT') //=> 'text/plain'
mime.lookup('htm') //=> 'text/html'
```

`Content-Disposition`: 无论响应的内容是什么样的`MIME`值，不要求客户端去打开它，只需弹出并下载它。
