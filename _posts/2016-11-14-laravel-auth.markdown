---
author: tangliangcheng
comments: true
date: 2016-11-14 23:00:34+00:00
layout: post
slug: laravel-auth
title: Laravel Auth深入探究
excerpt: 今天在写Laravel登录的时候，根据需求也不想用laravel自带的auth登录认证，便在搜索了一番，开始了auth的深入探究之旅。
categories:
- 技术分享
---

今天在写Laravel登录的时候，根据需求也不想用laravel自带的auth登录认证，便在搜索了一番，开始了auth的深入探究之旅。

刚开始根据官方文档，[用户认证](https://laravel-china.org/docs/5.1/authentication) 里的手动认证用户描述，实现了我的简单登录逻辑代码，如下：

	public function login(Request $request)
    {
        if (Auth::attempt(['name' => $request->name, 'password' => $request->password])) {
            //认证通过
            $rt = array('status' => 200, 'msg' => '登陆成功', 'data' => array('isLogin' => true));
            return response()->json($rt);
        } else {
            $rt = array('status' => 400, 'msg' => '登陆失败', 'data' => array('isLogin' => false));
        }

        return response()->json($rt);
    }
    

然后顺着laravel Auth的逻辑，找到在 ***Illuminate\Auth\Guard.php*** 中的 **attempt** 方法，如下图：

![attempt]({{ site.url }}/images/posts/auth-1.png)

根据代码的实现，最关键的点应该是以下两句代码:

	$this->lastAttempted = $user = $this->provider->retrieveByCredentials($credentials);
	
	$this->hasValidCredentials($user, $credentials)
	
我们一句一句来，顺着 **retrieveByCredentials** 方法，这里要说明的是，因为在 ***config/auth.php*** 中我选择的是 eloquent 认证驱动，所以找到 ***Illuminate\AuthEloquentUserProvider*** 下的 **retrieveByCredentials**方法，如下图：

![retrieveByCredentials]({{ site.url }}/images/posts/auth-2.png)

retrieveByCredentials 方法实现了根据用户登录输入的信息(除密码外)，查询到登录用户的信息并返回。

然后执行 **$this->hasValidCredentials($user, $credentials)**, 最终定位到 ***Illuminate\AuthEloquentUserProvider*** 中的 **validateCredentials**方法，如下图:

![validateCredentials]({{ site.url }}/images/posts/auth-3.png)

最终Laravel 通过 [Hash facade](https://laravel-china.org/docs/5.1/hashing) 提供 Bcrypt 加密来验证用户密码是否一致。(注意:数据库密码字段名为 **password**，若不是需在模型方法中重新指定)

看到了这里，也有了一些疑问，如果我不用hash来加密密码呢，使用md5 + "盐" 的方式，那就只有改写这个验证逻辑了。（"盐" 在数据库中字段名为 **salt**）

在 Illuminate\Contracts\Auth\Authenticatable 中加入方法:

	public function getAuthSalt();
	

在 Illuminate\Auth\Authenticatable 加入方法:

	public function getAuthSalt()
    {
       return $this->salt;
    }
    
改写上文提到的 **retrieveByCredentials** 方法

	public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['password'];
        $authPassword = $user->getAuthPassword();
        $authSalt = $user->getAuthSalt();
        return $authPassword === md5($plain.$authSalt);
    }
    
到这里基本就结束了，需要注意的是，使用Auth认证默认是 ***App\User.php***,我根据需求更改了 ***config/auth.php***，改成了 ***App\Models\User.php*** 因为该文件我是执行命令 ***php artisan make:model Models/User*** 生成的，便有了一点小坑的地方，它没有 use Auth 认证的类，所以此时将 ***app/User.php*** 下的复制过来即可。

----

以下是我在学习时，找到的两篇关于laravel auth 的文章,对我理解auth有很大的帮助

* [laravel身份验证-Auth的使用](https://my.oschina.net/cxz001/blog/347225)
* [Laravel框架初识：扩展Auth功能](http://blueve.me/archives/898)

----