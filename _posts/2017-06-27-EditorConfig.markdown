---
author: tangliangcheng
comments: true
date: 2017-06-27 23:00:34+00:00
layout: post
slug: EditorConfig
title: EditorConfig 规范编码
excerpt: vue-cli构建项目，修改editorconfig文件，规范编码
categories:
- 技术分享
---


[官方地址](http://editorconfig.org/)

官方描述：
> EditorConfig helps developers define and maintain consistent coding styles between different editors and IDEs. The EditorConfig project consists of a file format for defining coding styles and a collection of text editor plugins that enable editors to read the file format and adhere to defined styles. 

### 常用

{% highlight shell %}
#Unix风格换行符，作用于每个文件
end_of_line = lf 
insert_final_newline = true

#设置编码
charset = utf-8

#设置缩进
indent_style = tab
indent_size = 4

#设置换行符前面的空格
trim_trailing_whitespace = true
{% endhighlight %}

### 支持IDE

![IDE](http://ohg6w8k1d.bkt.clouddn.com/editorconfig-support)