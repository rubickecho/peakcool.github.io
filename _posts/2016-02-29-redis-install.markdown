---
author: tangliangcheng
comments: true
date: 2016-02-29 02:23:55+00:00
layout: post
link: http://www.rmogo.com/wordpress/2016/02/29/redis%e5%ad%a6%e4%b9%a0%e4%b9%8b%e5%ae%89%e8%a3%85/
slug: redis-install
title: Redis学习之安装
wordpress_id: 152
excerpt: redis起步之安装，Redis的版本选取目前的稳定版本是2.8.9。客户端选用了Redis的java版本jedis2.4.2.
categories:
- 技术分享
---

### 安装redis


**系统环境和版本说明**

Redis的版本选取目前的稳定版本是2.8.9。客户端选用了Redis的java版本jedis2.4.2。

**安装步骤**

a.进入root目录，并下载redis的安装包

{% highlight shell %}
$ wget http://labfile.oss.aliyuncs.com/files0422/redis-2.8.9.tar.gz
{% endhighlight %}

b.在目录下，解压按照包生成的目录redis-2.8.9

{% highlight shell %}
$ tar xvfz redis-2.8.9.tar.gz   
{% endhighlight %}

c.进入解压之后的目录进行编译

{% highlight shell %}
$ cd redis-2.8.9
$ make
{% endhighlight %}

d.进入src文件夹

{% highlight shell %}
$ cp redis-server /usr/local/bin/
$ cp redis-cli /usr/local/bin/
{% endhighlight %}

**测试redis的功能是否正常**

{% highlight shell %}
$ make test
{% endhighlight %}

在redis安装完成后，注意一些重要的文件。


  * 服务端: src/redis-server

	
  * 客户端: src/redis-cls

	
  * 默认配置文件: redis.conf


**启动redis-server**

{% highlight shell %}
$ redis-server   //服务器端
$ redis-cli      //客户端
{% endhighlight %}

注意，启动redis-server后不要关闭终端，另外开一个终端窗口执行redis-cli。

![reids启动服务]({{ site.url }}/images/posts/redis-serve.png)

由图可知，启动的端口为缺省的6379。
