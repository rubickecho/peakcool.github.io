---
author: tangliangcheng
comments: true
date: 2016-01-22 13:00:35+00:00
layout: post
link: http://www.rmogo.com/wordpress/2016/01/22/142/
slug: fis3-setting
title: FIS3初学习
excerpt: 初接触fis3，记录的一些基本配置.
wordpress_id: 142
categories:
- 技术分享
---

今天开始按照着fis3的官方文档学习起来，一点一点地配置。


<blockquote>fis3官网地址 [http://fis.baidu.com/fis3/index.html](http://fis.baidu.com/fis3/index.html)</blockquote>


fis3刚开始学习的时候最感兴趣的是浏览器自动刷新，文件的压缩，下面是一些配置代码:

  * js压缩

{% highlight shell %}
fis.match('*.js', {
    optimizer: fis.plugin('uglify-js')
});
{% endhighlight %}

  * css压缩

{% highlight shell %}
fis.match('*.css', {
    useSprite: true,
    optimizer: fis.plugin('clean-css')
});
{% endhighlight %}

  * png压缩

{% highlight shell %}
fis.match('*.png', {
    optimizer: fis.plugin('png-compressor')
});
{% endhighlight %}
	
  * 输出到指定目录
    
{% highlight shell %}
fis3 release -d <path>
{% endhighlight %}

  * 对css图片进行合并

{% highlight shell %}
fis.match('*.css', {
    useSprite: true
});
{% endhighlight %}

  * 浏览器自动刷新

{% highlight shell %}
fis3 server start -p [8081]
fis3 release -wL
{% endhighlight %}


注:因为我自己电脑的server port是8080,所以我将fis3内置服务器port改成了8081


***在执行命令时，需进入到项目文件夹内***


* * *




