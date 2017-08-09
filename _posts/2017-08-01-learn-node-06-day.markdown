---
author: tangliangcheng
comments: true
date: 2017-08-01 23:37:00+00:00
layout: post
slug: learn node 06 day
title: 《深入浅出NodeJs》学习笔记 05 day
excerpt: Node 网络编程
categories:
- 技术分享
---

### 构建TCP服务

#### TCP

全名为传输控制协议，在OSI模型（由七层组成，分别为物理层、数据链路层、网络传输层、会话层、表示层、应用层）中属于传输层协议。

其最显著的特征是在传输之间需要3次握手形成会话，只有会话形成之后，服务器端和客户端之间才能互相发送数据。在创建会话的过程中，服务器端和客户端分别提供一个`套接字`，这两个套接字共同形成一个链接。服务器端与客户端则通过套接字实现两者之间链接的操作。

```js
var net = require('net')
var server = net.createServer(function(socket) {
    socket.on('data', function(data) {
        socket.write('Hello')
    })

    socket.on('end', function() {
        console.log('连接断开')
    })

    socket.listen(8124, function() {
        console.log('server bound') //成功
    })
})
```

#### TCP服务的事件

在上述示例中，代码分为服务器事件和连接事件。

##### 服务器事件
对于通过`net.createServer()`创建的服务器，它是一个`EventEmitter`实例，它自定义事件有如下几种：

* listening: 在调用`server.listen()`绑定端口或者Domain Socket后触发，简洁写法为`server.list(port, listeningListener)`，通过第二个参数传入。
* connection: 每个客户端套接字连接到服务器端时触发，简洁写法为`net.createServer()`。
* close: 当服务器关闭时触发，在调用`server.close()`后，服务器将停止接受新的套接字连接，但保持当前连接，等所有连接断开后会触发该事件。
* error: 当服务器发生异常时。

##### 连接事件

服务器可以同时与多个客户端保持连接，对于每个连接而言就是典型的可写可读`Stream`对象。

`Stream`对象可以用于服务器端和客户端之间的通信，既可以通过`data`事件从一端读取另一端发来的数据，也可以通过`write()`方法从一端向另外一端发送数据。具有如下自定义事件：

* data: 当一端调用`write`发送数据时，另一端会触发`data`事件，事件传递的数据即是`write()`发送的数据。(**注意:**并不是每次write都会触发一次data事件,当关闭`Nagle`算法后，另一端可能收到多个小数据包合并，只触发一次data事件)
* end: 当连接中的任意一端发送了FIN数据时，将会触发该事件。
* connect: 该事件用于客户端，当套接字与服务器端连接成功时会被触发。
* drain: 当任意一端调用`write()`发送数据时，当前这端会触发该事件。
* error: 当异常发生时，触发该事件。
* close: 当套接字完全关闭时，触发该事件。
* timeout: 当一定事件后连接不再活跃时，该事件将会被触发，通知用户当前连接已经被闲置了。

**TCP套接字是可写可读的Stream对象，可以利用pip()方法巧妙实现管道操作**

### UDP

在UDP中，一个套接字可以与多个UDP服务通信，它虽然提供面向事务的简单不可靠信息传输服务，在网络差的情况下，存在丢包严重的问题，但是由于它无须连接，资源消耗低，处理快速灵活，所以常常应用在那种偶尔丢一两个数据包也不会产生重大影响的场景，比如：音频、视频等等。DNS服务也是基于它实现的。

#### 创建UDP套接字

一旦创建，既可以作为客户端发送数据，也可以作为服务器端接受数据。

```js
var dgram = require('dgram')
var socket = dgram.createScoket('udp4')
```

* 创建服务器端：调用`dgram.bing(port, [address])` 设置网卡和端口
* 创建客户端，调用`send()`发送消息到网络中

#### UDP套接字事件

UDP套接字事件只是一个EventEmitter的实例，具备如下自定义事件：

* message: 当UDP套接字侦听网卡端口后，接受到消息时触发该事件，触发携带的数据为消息的`Buffer`对象和一个远程地址信息
* listening: 当UDP套接字开始侦听时触发
* close: 调用`close()`方法时触发该事件，并不再触发`message`事件。如需再次触发`message`事件，可以重新绑定
* error: 当异常发生时触发

---

### 构建HTTP服务

#### HTTP

从协议的角度来说，现在的应用，如浏览器，其实是一个HTTP的代理，用户的行为将会通过它转化为HTTP请求报文发送给服务器端，服务器端在处理请求后，发送响应报文给代理，代理在解析报文后，将用户需要的内容呈现在界面上。

无论是HTTP请求报文还是HTTP响应报文，报文内容都包含两个部分：报文头和报文体。

**1、TCP的3次握手**
```shell
* Rebuilt URL to: www.baidu.com/
*   Trying 220.181.111.188...
* TCP_NODELAY set
* Connected to www.baidu.com (220.181.111.188) port 80 (#0)
```

**2、客户端想服务器端发送请求报文**
```shell
> GET / HTTP/1.1
> Host: www.baidu.com
> User-Agent: curl/7.54.0
> Accept: */*
```

**3、服务器端完成处理后，向客户端发送响应内容，包括响应头和响应体**
```shell
< HTTP/1.1 200 OK
< Server: bfe/1.0.8.18
< Date: Thu, 03 Aug 2017 05:20:56 GMT
< Content-Type: text/html
< Content-Length: 2381
< Last-Modified: Mon, 23 Jan 2017 13:27:36 GMT
< Connection: Keep-Alive
< ETag: "588604c8-94d"
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Pragma: no-cache
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
< Accept-Ranges: bytes
```

**4、最后部分为结束会话的信息**

#### http模块

**http模块即是将`connection`到`request`的过程进行了封装**

![`connection`到`request`的过程进行封装](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%60connection%60%E5%88%B0%60request%60%E7%9A%84%E8%BF%87%E7%A8%8B%E8%BF%9B%E8%A1%8C%E5%B0%81%E8%A3%85.png?raw=true)

除此之外，http模块将连接所用套接字的读写抽象为`ServerRequest`和`ServerResponse`对象，它们分别对应请求和响应操作。在请求产生的过程中，http模块拿到连接中传来的数据，调用二进制模块`http_parser`进行解析，解析完请求报文的报头后，触发`request`事件，调用用户业务逻辑。

![http模块产生请求的流程](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/http%E6%A8%A1%E5%9D%97%E4%BA%A7%E7%94%9F%E8%AF%B7%E6%B1%82%E7%9A%84%E6%B5%81%E7%A8%8B.png?raw=true)

##### http请求

请求报文头第一行`GET/HTTP/1.1`解析分为如下：

* req.method属性： 值为GET,是为http请求方法
* req.url属性：值为/
* req.httpVersion属性，值为1.1

其余报头很规律的`Key: Value`格式，被解析后放置在`req.headers`属性上传递给业务逻辑调用。
报文体部分则抽象为一个只读对象，如果业务逻辑需要读取报文体中的数据，则要在这个数据流结束后才能操作。

```js
function (req, res) {
    var buffers = []
    req.on('data', function(trunk) {
        buffers.push(trunk)
    }).on('end', function() {
        var buffer = Buffer.concat(buffers)
        res.end('Hello World')
    })
}
```

##### http响应

http响应封装了对底层连接的写操作，可以将其看成一个可写的流对象。它影响响应报文头部消息的API为`setHeader()`和`writeHead()`两个步骤，实际生成如下报文：

```shell
< HTTP/1.1 200 OK
< Content-Type: text/html
```

可以调用`setHeader`进行多次设置，但只有调用`writeHead`后，报头才会被写入到连接中。
报文体部分则是调用`res.write()`和`res.end()`方法实现，后者和前者的差别在于`res.end()`会先调用`write()`发送数据，然后发送信号告知服务器这次响应结束。

##### http服务的事件

服务器也是一个`EventEmitter`实例：

* connection事件
* request事件
* close事件
* checkContinue事件
* connect事件
* upgrade事件
* clientError事件

#### http客户端

与http服务器基本相似，提供一个`http.request(options, connect)` 用于构造HTTP客户端。

##### options选项

* host: 服务器的域名或者IP地址, 默认为`localhost`
* hostname: 服务器名称
* port: 服务器端口
* localAddress: 建立网络连接的本地网卡
* socketPath: Domain套接字路径
* method: HTTP请求方法，默认为GET
* path: 请求路径，默认为/
* headers: 请求头对象
* auth: `Basic`认证，这个值将被计算为请求头中的`Authorization`部分

##### http响应

HTTP客户端的响应对象与服务器相似，在`ClientRequest`对象中，它的事件叫做`response`

##### http代理

**HTTP代理对服务器端创建的连接进行管理**
![HTTP代理对服务器端创建的连接进行管理](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/HTTP%E4%BB%A3%E7%90%86%E5%AF%B9%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E5%88%9B%E5%BB%BA%E7%9A%84%E8%BF%9E%E6%8E%A5%E8%BF%9B%E8%A1%8C%E7%AE%A1%E7%90%86.png?raw=true)

调用HTTP客户端同时对一个服务器发起10次HTTP请求时，其实质只有5个请求处于并发状态，后续的请求需要等待某个请求完成服务后才真正发出。这与浏览器对同一个域名下有下载连接数的限制是相同的行为。如需改变，可以在`option`中传递`agent`选项。

---

### 构建WebScoket服务

WebScoket与传统HTTP有以下好处：

* 客户端与服务器端只建立一个TCP连接，可以使用更少的连接
* WebScoket服务器端可以推送数据到客户端，这远比HTTP请求响应模式更灵活高效
* 有更轻量的协议头，减少数据传输

WebScoket协议主要分为两个部分：**握手** 和 **数据传输**

### 网络服务与安全

SSL作为一种安全协议，它在传输层提供对网络连接加密的功能。对于应用层而言，它是透明的，数据在传输到应用层之前就已经完成了加密和解密的过程。最初的SSL应用在web上，被服务器端和浏览器端同时支持，随后IETF将其标准化，称为TLS(Transport Layer Security, 安全传输层协议)。

#### TLS/SSL

TLS/SSL是一个公钥/私钥的结构，是一个非堆成的结构，每个服务器端和客户端都有自己的公私钥。公钥用来加密要传输的数据，私钥用来解密接受到的数据。公钥和私钥是配对的，通过公钥加密的数据，只有通过私钥才能解密，所以建立连接安全传输之前，客户端和服务器端互换公钥。客户端发送数据时要通过服务器端的公钥加密，服务器端发送数据时则需要通过客户端的公钥进行加密。

Node底层采用的是`openssl`实现的TLS/SSL,为此要生成公钥和私钥可以通过`openssl`实现。

```shell
// 服务器端私钥
openssl genrsa -out server.key 1024

// 客户端私钥
openssl genrsa --out client.key 1024
```

但是这样网络中依然可能存在窃听情况，典型的例子是中间人攻击。为了解决这个问题，引入了**数字证书**。与直接公钥不同，数字证书中包含了服务器的名称和主机名，服务器的公钥、签名颁发机构的名称、来自签名颁发机构的签名。在连接建立前，会通过证书中的签名确认收到的公钥是来自目标服务器的，从而产生信任关系。

#### HTTPS

HTTPS服务就是工作在TLS/SSL上的HTTP。

创建HTTPS服务如下：

* 准备证书
* 创建HTTPS服务
