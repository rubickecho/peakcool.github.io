---
author: tangliangcheng
comments: true
date: 2018-10-28 21:40:00+00:00
layout: post
slug: gem install bundle faile
title: gem安装依赖失败
excerpt: gem安装依赖失败
categories:
- 技术分享
---
<!-- TOC -->

- [gem安装依赖失败](#gem安装依赖失败)

<!-- /TOC -->
## gem安装依赖失败

很久没有写博客了，最近也新换了电脑，重新把博客从github上拉下来。

安装了`jekyll`后，满心欢喜的执行`jekyll serve` 终端里得来的却是无情的红色报错，心里顿时一凉，好不容易想写文章的热情也被浇灭。

终于2天后，还是静下心来把这些问题解决。

1. run `gem install bundle`

2. remove `Gemfile.lock`

3. run `sudo bundle install`

这时再启动，发现报错

```shell
/Library/Ruby/Gems/2.3.0/gems/bundler-1.17.1/lib/bundler/runtime.rb:319:in `check_for_activated_spec!': You have already activated i18n 1.1.1, but your Gemfile requires i18n 0.9.5. Prepending `bundle exec` to your command may solve this. (Gem::LoadError)
	from /Library/Ruby/Gems/2.3.0/gems/bundler-1.17.1/lib/bundler/runtime.rb:31:in `block in setup'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/forwardable.rb:202:in `each'
	from /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/forwardable.rb:202:in `each'
	from /Library/Ruby/Gems/2.3.0/gems/bundler-1.17.1/lib/bundler/runtime.rb:26:in `map'
	from /Library/Ruby/Gems/2.3.0/gems/bundler-1.17.1/lib/bundler/runtime.rb:26:in `setup'
	from /Library/Ruby/Gems/2.3.0/gems/bundler-1.17.1/lib/bundler.rb:107:in `setup'
	from /Library/Ruby/Gems/2.3.0/gems/jekyll-3.8.4/lib/jekyll/plugin_manager.rb:50:in `require_from_bundler'
	from /Library/Ruby/Gems/2.3.0/gems/jekyll-3.8.4/exe/jekyll:11:in `<top (required)>'
	from /usr/local/bin/jekyll:22:in `load'
	from /usr/local/bin/jekyll:22:in `<main>'
```

这时再执行 `gem unintall i18n`，选择移除`1.1.1`版本

大功告成！
