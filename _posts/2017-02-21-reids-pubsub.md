---
layout:     post
title:      "Redis 发布与订阅"
subtitle:   "学习 Redis 发布与订阅功能"
date:       2017-02-09
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Redis
---

> 学习 Redis 发布与订阅功能



# Redis 发布与订阅功能
发布与订阅 (pub/sub) 是一种消息同通信模式，是一种常用的设计模式(观察者模式)，主要的目的是解耦消息发布者和消息订阅者的耦合。redis 实现发布与订阅功能的代码级相互解耦，也解决两者在物理部署上的耦合，redis 客户端可以通过 subscribe 和 psubscribe 命令向 redis 服务端订阅相关的信息。

下图是发布与订阅的示意图：

下图展示的是一个 channel 和三个 client，三个 client 向 channel 订阅了信息。

![](/img/blog/redis-pub0.png)

下图展示的是信息通过 channel 发布给订阅者。


![](/img/blog/redis-pub1.png)



# subscribe 命令

subscribe 命令是用来订阅消息的，subscribe 后加名称，可订阅相应的消息。

在客户端中输入：SUBSCRIBE test

订阅 test 相关的消息

	127.0.0.1:6379> SUBSCRIBE test
	Reading messages... (press Ctrl-C to quit)
	1) "subscribe"
	2) "test"
	3) (integer) 1
	
	
	
# publish 命令

publish 用于发布消息

在另一个客户端输入： publish test “Hello world”

在 test 频道发布 Hello world 的消息
	
	127.0.0.1:6379> PUBLISH test "Hello world"
	(integer) 1

在订阅 test 频道的客户端可以收到如下消息：

	
	127.0.0.1:6379> SUBSCRIBE test
	Reading messages... (press Ctrl-C to quit)
	1) "subscribe"
	2) "test"
	3) (integer) 1
	1) "message"
	2) "test"
	3) "Hello world"	

# psubscribe 命令


psubscribe 命令用来订阅一个或多个符合给定模式的频道。

	127.0.0.1:6379> PSUBSCRIBE pattern [pattern ...]


psubscribe 可以匹配多个频道。


	127.0.0.1:6379> PSUBSCRIBE test*
	Reading messages... (press Ctrl-C to quit)
	1) "psubscribe"
	2) "test*"
	3) (integer) 1
	
匹配以 test 开头的频道，例如 test1、test.test、test.test1 等等。

注：订阅多个频道，例如同时订阅 test* 和 test1 ，向 test1 频道发送消息会导致订阅者多次。









 