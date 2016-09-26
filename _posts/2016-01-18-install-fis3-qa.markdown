---
author: tangliangcheng
comments: true
date: 2016-01-18 13:54:53+00:00
layout: post
link: http://www.rmogo.com/wordpress/2016/01/18/%e5%ae%89%e8%a3%85fis3%e6%97%b6%e9%81%87%e5%88%b0%e7%9a%84%e4%b8%80%e4%ba%9b%e9%97%ae%e9%a2%98/
slug: install-fis3-qa
title: 安装FIS3时遇到的一些问题
wordpress_id: 134
categories:
- 技术分享
---

最近想学习点新东西，发现前端的构建工具fis比较有趣，便开始按照文档安装起来，第一步便遇到了问题:




<blockquote>error1：npm ERR! Error: EACCES, mkdir

解决办法：权限问题，加个sudo再来一次</blockquote>


加了权限执行以后，满以为就能顺顺利利的撸fis3了，结果一个难题出现(如下图):

![2](http://115.28.108.2/wordpress/wp-content/uploads/2016/01/2.png)


<blockquote>解决办法:因为天朝的墙太厚了，fis的有些组件不能正常安装，这是好就需要换国内的源，个人推荐淘宝

>     
>     <code>  sudo npm install -g fis --disturl=http://registry.npm.taobao.org/mirrors/node --registry=http://registry.npm.taobao.org</code>
> 
> 
然后再执行

>     
>     <code>  npm install -g fis3</code>
> 
> 
最后执行

>     
>     <code>  npm -v</code>
> 
> 
出现下图，代表着终于能够开始撸fis3了。</blockquote>


![1](http://115.28.108.2/wordpress/wp-content/uploads/2016/01/1.png)
