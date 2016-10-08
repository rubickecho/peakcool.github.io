---
author: tangliangcheng
comments: true
date: 2016-03-03 13:56:11+00:00
layout: post
link: http://www.rmogo.com/wordpress/2016/03/03/mac%e4%b8%8bphp%e5%ae%89%e8%a3%85%e6%8b%93%e5%b1%95phpreids/
slug: mac-install-phpredis
title: mac下PHP安装拓展PHPRedis
excerpt : mac下PHP安装mamp拓展phpredis方法介绍
wordpress_id: 156
categories:
- 技术分享
---

### 一.mac自带php安装phpredis

{% highlight shell %}
$ brew install php56-redis  #56为php版本5.6.10
{% endhighlight %}

### 二.mac下mamp安装phpredis

    1. 进入**/Applications/MAMP/bin/php/php5.6.10/** 下载phpredis **_https://github.com/phpredis/phpredis_**

    2. 进入phpredis文件夹：

    {% highlight shell %}
    $ cd phpredis
    $ sudo phpize
    $ sudo ./configure --with-php-config=/Applications/MAMP/bin/php/php5.6.10/bin/php-config  #配置phpredis
    $ sudo make  #编译make
    $ sudo cp -p modules/redis.so /Applications/MAMP/bin/php/php5.6.10/lib/php/extensions/no-debug-non-zts-20131226 #将编译完成的模块，复制到MAMP 中的PHP 扩展中
    $ sudo vi /Applications/MAMP/bin/php/php5.6.10/conf/php.ini
    $ extension=redis.so    #修改 php.ini 文件在增加 redis.so 扩展
    {% endhighlight %}
    3. 最后完成，重启MAMP

![reids模块]({{ site.url }}/images/posts/phpredis.png)
