---
author: tangliangcheng
comments: true
date: 2017-08-10 22:33:00+00:00
layout: post
slug: learn node 08 day
title: 《深入浅出NodeJs》学习笔记 08 day
excerpt: Node 玩转进程
categories:
- 技术分享
---

## 玩转进程

### 多进程架构

#### Master-Worker模式（主从模式）

图中进程分为两种：主进程和工作进程。这是典型的分布式架构中用于并行处理业务的模式，具备较好的可伸缩性和稳定性。主进程不负责具体的业务处理，而是负责调度或者管理工作进程，它是趋向稳定的。工作进程负责具体的业务处理。

![Master-Worker](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/Master-Worder%E6%A8%A1%E5%BC%8F.png?raw=true)

#### 创建子进程

`child_process`模块提供了4个方法用于创建子进程:

* spawn(): 启动一个子进程来执行命令
* exec(): 启动一个子进程来执行命令，与spawn()不同的是其接口不同，它有一个回调函数获知子进程的状况
* execFile(): 启动该一个子进程来执行可执行文件
* fork(): 与spawn()类似，不同点在于它创建Node的子进程只需指定要执行的javascript文件模块即可

spawn()与exec()、execFile()不同的是，后两者创建的时间可以指定`timeout`属性设置超时时间，一旦创建的进程运行超过设定的时间将会被杀死。

**4种方法的差别**

| 类型 | 回调/异常 | 进程类型 | 执行类型 | 可设置超时 |
| - | - | - | - | - |
| spawn() | no | 任意 | 命令 | no |
| exec() | yes | 任意 | 命令 | yes |
| execFile() | yes | 任意 | 可执行文件 | yes |
| fork() | no | Node | JavaScript文件 | no |

#### 进程间通信

主进程与工作线程之间通过`onmessage()`和`postMessage()`进行通信，子进程对象则由`send()`方法实现主进程向子进程发送数据,`message`事件实现收听子进程发来的数据。

#### 进程间通信原理

IPC的全称是Inter-Process Communication,即进程间通信。进程间通信的目的是为了让不同的进程能够互相访问资源并协调工作。实现进程间通信的结束有很多，如命名管道、匿名管道、socket、信号量、共享内存、消息队列、Domain Socket等。Node中实现IPC通道的是管道(pipe)技术。但是此管道非彼管道，在Node中管道是个抽象层面的称呼，具体细节实现由libuv提供，在Windows下由命名管道( named pipe ) 实现， *nix系统则采用Unix Domain Socket实现。表现在应用层上的进程间通信只有简单的message事件和send()方法。

父进程在实际创建子进程之前，会创建IPC通道去监听它，然后才真正创建出子进程，并通过环境变量(NODE_CHANNEL_FD)告诉子进程这个IPC通道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个已经存在的IPC通道，从而完成父子进程之间的连接。

**IPC创建和实现示意图**

![IPC创建和实现示意图](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/IPC%E5%88%9B%E5%BB%BA%E5%92%8C%E5%AE%9E%E7%8E%B0%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

**创建IPC管道的步骤示意图**

![](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/%E5%88%9B%E5%BB%BAIPC%E7%AE%A1%E9%81%93%E7%9A%84%E6%AD%A5%E9%AA%A4%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

**注意：只有启动的子进程是Node进程时**

#### 句柄传递

只有一个进程可以监听到一个端口上，其余的进程在监听的过程中都会抛出`EADDRINUSE`异常，这是端口被占用的情况，新的进程不能继续监听该端口。这个问题破坏我们将多个进程监听同一个端口的想法。要解决这个问题，通常的做法是让每个进程监听不同的端口，其中主进程监听主端口（如80），主进程对外接收所有的网络请求，再将这些请求分别代理到不同的端口进程上。

![主进程接收、分配网络请求示意图](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/%E4%B8%BB%E8%BF%9B%E7%A8%8B%E6%8E%A5%E6%94%B6%E3%80%81%E5%88%86%E9%85%8D%E7%BD%91%E7%BB%9C%E8%AF%B7%E6%B1%82%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

通过代理，可以避免端口不能重复监听的问题，甚至可以在代理进程上适当的负载均衡，使得每个子进程可以较为均衡地执行任务。由于进程每接收到一个连接，将会用掉一个文件描述符，因此代理方案中客户端连接到代理进程，代理进程连接到工作进程的过程中需要用掉2个文件描述符。操作系统的文件描述符是有限的，代理方案浪费掉一倍数量的文件描述符的做法影响了系统的扩展能力。

为了解决上述这样的问题，Node在版本v0.5.9引入了进程间发送句柄的功能。send()方法除了能通过IPC发送数据外，还能发送句柄，第二个可选参数就是句柄，如下：

```js
child.send(message, (sendHandle))
```

什么是句柄呢？**句柄是一种可以用来标识资源的引用**，它的内部包含了指向对象的文件描述符。比如句柄可以用来标识一个服务器端socket对象，一个客户端socket对象、一个UDP套接字、一个管道等。

发送句柄意味着什么？在前一个问题中，我们可以去掉代理这种方案，使主进程接收到socket请求后，将这个socket直接发送给工作进程，而不是重新与工作进程间建立新的socket连接来转发数据。文件描述符浪费的问题可以通过这种方式轻松解决。

![](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/%E4%B8%BB%E8%BF%9B%E7%A8%8B%E5%B0%86%E8%AF%B7%E6%B1%82%E5%8F%91%E9%80%81%E7%BB%99%E5%B7%A5%E4%BD%9C%E8%BF%9B%E7%A8%8B%202.png?raw=true)


主进程代码:

```js
var child = require('child_process').fork('child.js')

var server = require('net').createServer()
server.on('connection', function(socket) {
    socket.end('handle by parent')
})

server.listen(1337, function(){
    child.send('server', server)
})
```

子进程代码:

```js
process.on('message', function(m, server){
    if ( m === 'server' ) {
        server.on('connection', function(socket) {
            socket.on('handle by child')
        })
    }
})
```

再优化，将服务器句柄发送给子进程之后，关闭服务器监听，让子进程处理请求。

![句柄的发送与还原示意图](https://github.com/peakcool/learn-nodejs-notes/blob/master/%E7%A4%BA%E6%84%8F%E5%9B%BE/%E7%8E%A9%E8%BD%AC%E8%BF%9B%E7%A8%8B/%E5%8F%A5%E6%9F%84%E7%9A%84%E5%8F%91%E9%80%81%E4%B8%8E%E8%BF%98%E5%8E%9F%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

#### 句柄发送与还原

目前子进程对象send()方法可以发送的句柄类型包括：

* net.Socket TCP套接字
* net.Server TCP服务器
* net.Navtive C++层面的TCP套接字或者IPC管道
* dgram.Socket UDP套接字
* dgram.Native C++层面的UDP套接字

send()方法在将消息发送到IPC管道前，将消息组装成两个对象，一个参数是`handle`，另一个是`message`。`message`参数如下：

```js
{
    cmd : 'NODE_HANDLE',
    type : 'net.Server',
    msg : message
}
```
发送到IPC管道中的实际是我们要发送的句柄文件描述符，文件描述符实际上是一个整数值。这个`message`对象在写入到IPC管道时也会通过JSON.stringify()进行序列化。所以最终发送到IPC通道中的信息都是字符串，send()方法能发送消息和句柄并不意味着它能发送任意对象。

多个应用监听相同端口时，文件描述符同一时间只能被某个进程所用，换言之就是网络请求向服务器端发送请求时，只有一个幸运的进程能够抢到连接，也就是说只有它能为这个请求进行服务。这些进程服务是**抢占式**的。

### 负载均衡

在进程间监听相同的端口，使得用户请求能够分散到多个进程上进行处理，这带来的好处是可以将CPU资源都调用起来。

比如：饭店将客人的点单分发给多个厨师进行点餐点制作。既然涉及多个厨师共同处理所有菜单，那么保证每个厨师的工作量是一门学问，既不能让一些厨师忙不过来，也不能让一些厨师闲着，这个保证多个处理单元工作量公平的策略叫做**负载均衡**。

Node默认提供的机制是**抢占式**策略。所谓的抢占式就是在一堆工作进程中，闲着的进程对到来的请求进行争抢，谁抢到谁服务。

为此Node在v0.11中提供了一种新的策略使得负载均衡更合理，这种新的策略叫做**Round-Robin 伦叫调度**。
