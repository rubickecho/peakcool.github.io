---
author: tangliangcheng
comments: true
date: 2016-11-07 02:00:34+00:00
layout: post
slug: redis-learn
title: Redis学习的一些总结
excerpt: Redis是基于内存的数据存储系统，数据以key=>value形式存储。value支持多种数据类型，包括string,hash,list,set,sorted set等等。
categories:
- 技术分享
---

Redis是基于内存的数据存储系统，数据以key=>value形式存储。value支持多种数据类型，包括string,hash,list,set,sorted set等等。

### 1.起步

[安装教程](http://www.redis.net.cn/tutorial/3503.html)

启动服务端: redis-server

启动客户端: redis-cli

验证是否成功:

	127.0.0.1:6379> PING
	PONG
	127.0.0.1:6379>
	
[学习教程](http://www.redis.net.cn/tutorial/3501.html)

### 2.配置文件  redis.conf

	CONFIG GET * //查看所有配置
	
* daemonize: 默认为no，yes则为启动redis-server时自动是后台运行方式

* port: 指定端口号

* bind: 绑定ip，只接受来自绑定ip的请求，更安全 

### 3.redis 数据持久化

RDB原理:

* Redis使用fork函数复制一份当前进程(父) 的副本(子)
* 父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件
* 当子进程写入完所有数据后会用该临时文件替换旧的rdb文件，到此一次快照操作完成。

AOF原理:

AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据。运作方式是不断地将命令追加到文件的末尾。

如何选择？

* 如果想达到像数据库一样的数据安全性能，应该同时使用两种
* 若要求不是特别高，可以只使用RDB持久化

### 4.redis 开启启动服务

1. /etc/init.d/redis 加入以下内容

		#!/bin/sh 
		##chkconfig: 2345 80 90 
		##description:auto_run 
		PATH=/usr/local/bin:/sbin:/usr/bin:/bin 

		REDISPORT=6379 
		EXEC=/usr/local/bin/redis-server 
		REDIS_CLI=/usr/local/bin/redis-cli 
		
		PIDFILE=/var/run/redis.pid 
		CONF="/etc/redis.conf" 
		
		case "$1" in 
		start)
			if [ -f $PIDFILE ] 
			then 
				echo "$PIDFILE exists, process is already running or crashed" 
			else 
				echo "Starting Redis server..." 
				$EXEC $CONF 
			fi 
			if [ "$?"="0" ] 
			then 
				echo "Redis is running..." 
				fi 
				;; 
			stop) 
			if 
			[ ! -f $PIDFILE ] 
			then 
				echo "$PIDFILE does not exist, process is not running" 
			else 
				PID=$(cat $PIDFILE) 
				echo "Stopping ..."
				$REDIS_CLI -p $REDISPORT SHUTDOWN 
				while [ -x ${PIDFILE} ] 
				do 
					echo "Waiting for Redis to shutdown ..." 
					sleep 1 
				done 
					echo "Redis stopped" 
				fi 
				;; 
			restartforce-reload) 
				${0} stop 
				${0} start 
				;; 
			*) 
			echo "Usage: /etc/init.d/redis {startstoprestartforce-reload}" >&2 
				exit 1 
		esac 
		#######

2. 设置权限: chmod +x /etc/init.d/redis
3. 加入开机启动服务

	* linux: sudo chkconfig redis on
	* ubuntu: sudo sysv-rc-conf redis on
	
4. 检查是否加入服务: service redis start

### 5.Redis 6种过期策略

* Volatile-lru : 只针对设置了过期时间的key
* AllKeys-lru: 删除lru算法的key
* Volatile-random: 随机删除即将过期的key
* Allkeys-random: 随机删除
* Volatile-ttl: 删除即将过期的key
* noeviction: 永不过期

### Redis 事务

事务是一个单独的隔离，事务中的所有命令都会序列化，按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

事务是一个原子操作，事务中的命令要么全部被执行(commit)，要么全部不执行，可以回滚(rollback)。

一个事务从开始到执行会经历三个阶段

* 开始事务
* 命令入队
* 执行事务


|   序号    | 命令及描述 	  |
|   -----  |:------: |
|1|DISCARD 取消事务，放弃执行事务块内的所有命令。|
|2|	EXEC 执行所有事务块内的命令。|
|3|	MULTI 标记一个事务块的开始。|
|4|UNWATCH 取消 WATCH 命令对所有 key 的监视。|
|5|WATCH key [key ...] 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。|


### Redis 手册

[Redis中文手册PHP版](http://www.cnblogs.com/ikodota/archive/2012/03/05/php_redis_cn.html)

[Redis手册](http://www.redis.net.cn/order/)

[Redis命令参考](http://doc.redisfans.com/)

### Redis 精文

[缓存更新套路--陈皓](http://coolshell.cn/articles/17416.html)

[20分钟快速了解Redis--手插口袋_](http://www.imooc.com/article/3585)

[Redis进阶：数据持久化，安全，在PHP中使用--atwal](http://www.imooc.com/article/11205)


