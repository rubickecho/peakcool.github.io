---
author: tangliangcheng
comments: true
date: 2015-12-29 12:11:53+00:00
layout: post
link: http://www.rmogo.com/wordpress/2015/12/29/chrome%e8%b0%83%e8%af%95%e5%b0%8f%e6%8a%80%e5%b7%a7/
slug: chrome-debug-tips
title: MAC Chrome调试小技巧【转载】
wordpress_id: 429
wordpress_id: 40
excerpt: 除了我们日常使用的调试方法，在Chrome中，其含有一些有意思的方法，有助于提高我们的开发调试效率。
categories:
- 转载收藏
---

**前言：**

除了我们日常使用的调试方法，在Chrome中，其含有一些有意思的方法，有助于提高我们的开发调试效率。



* * *





**Sources页**

**`command + p` 文件跳转**

使用Sublime的人或习惯用`command + p` 进行文件的跳转，在chrome dev tools中其实也有类似的跳转方法。

    
    <code><span class="pln">command </span><span class="pun">+</span><span class="pln"> p  
    
    command </span><span class="pun">+</span><span class="pln"> p </span><span class="pun">+</span> <span class="pun">文件名</span> <span class="pun">+</span> <span class="pun">:</span> <span class="pun">+</span> <span class="pun">数字</span></code>


**`command + shift + o` 任意方法跳转**

Sublime中使用`command +R` 进行方法跳转，而在dev tools中，可以使用`command + shift + o` 进行任意方法的跳转。

    
    <code><span class="pln">command </span><span class="pun">+</span><span class="pln"> shift </span><span class="pun">+</span><span class="pln"> o  </span><span class="com">// 跳转到任意方法</span></code>


**注：** 查找某文件中的方法，使用`command + p` 和 `command + shift + o` 更配哦~

**Elements页**



	
  * 使用方向键快，上下键导航，左右键收起展开；

	
  * H键快速隐藏dom（效果相当于给DOM加入visibility: hidden属性，有别于display:none）

	
  * Enter进行快速编辑属性；

	
  * 鼠标右击使用各类方法…


**Console页**

**`$_` 表示上次的计算结果**

举个栗子

    
    <code><span class="lit">15</span> <span class="pun">*</span> <span class="lit">15</span><span class="pln">  
    
    $_ </span><span class="pun">*</span> <span class="lit">10</span> </code>


**`$0` 获取当前选中的DOM**

选中DOM之后，在控制台输入`$0`。

    
    <code><span class="pln">$0</span></code>


![图片描述](http://img.mukewang.com/55aef87f0001fd7105880240.png)
注：`$1 $2 $3` 是获取前几次选的dom，不常用

**`$(selector)` 与 `$$(selector)` 获取当前选中的DOM**
当页面没有引入jQuery等类库的时候，这是我们一般会用。
`document.querySelector()` 或是 `document.querySelectorAll()` 来作用dom选择器。
而在Chrome调试中我们可以使用是`$(selector)` 与 `$$(selector)`来作为选择器，省去大串代码，如下。

    
    <code><span class="pln">$</span><span class="pun">(</span><span class="str">'body'</span><span class="pun">)</span><span class="pln">
    
    $$</span><span class="pun">(</span><span class="str">'body'</span><span class="pun">)</span></code>


由上图实际结果看出，`$()`和`$$()`获取得到的都是满足选中条件元素的一个集合，相当于`document.querySelectorAll()`
注: 实验所用chrome版本：40.0.2214.111 (64-bit)

**copy(Object) 拷贝对象**

    
    <code><span class="pln">copy</span><span class="pun">(</span><span class="pln">document</span><span class="pun">.</span><span class="pln">body</span><span class="pun">)</span><span class="pln">
    
    copy</span><span class="pun">(</span><span class="pln">$0</span><span class="pun">)</span></code>


注: 可搭配`$0`来拷贝当前选择的dom，记得手动黏贴。

**console.time & console.timeEnd 计算耗时**

对代码执行的耗时情况进行测试时，处理手工在代码中创建前后两个时间戳进行对比，在dev tools中，我们可以使用`console.time`与 `console.timeEnd`实现。

    
    <code><span class="pln">console</span><span class="pun">.</span><span class="pln">time</span><span class="pun">(</span><span class="str">"测试用时"</span><span class="pun">);</span>
    <span class="kwd">var</span><span class="pln"> array</span><span class="pun">=</span> <span class="kwd">new</span> <span class="typ">Array</span><span class="pun">(</span><span class="lit">1000000</span><span class="pun">);</span>
    <span class="kwd">for</span> <span class="pun">(</span><span class="kwd">var</span><span class="pln"> i </span><span class="pun">=</span><span class="pln"> array</span><span class="pun">.</span><span class="pln">length </span><span class="pun">-</span> <span class="lit">1</span><span class="pun">;</span><span class="pln"> i </span><span class="pun">>=</span> <span class="lit">0</span><span class="pun">;</span><span class="pln"> i</span><span class="pun">--)</span> <span class="pun">{</span><span class="pln">
    array</span><span class="pun">[</span><span class="pln">i</span><span class="pun">]</span> <span class="pun">=</span> <span class="kwd">new</span> <span class="typ">Object</span><span class="pun">();</span>
    <span class="pun">};</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">timeEnd</span><span class="pun">(</span><span class="str">"测试用时"</span><span class="pun">);</span></code>


**关闭Console界面**

ESC…



* * *





原文源自：AlloyTeam——[http://www.alloyteam.com/2015/06/chrome-diao-shi-ji-qiao/](http://www.alloyteam.com/2015/06/chrome-diao-shi-ji-qiao/)
