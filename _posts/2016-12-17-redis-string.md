---
layout:     post
title:      "Redis 中 string 的实现原理"
subtitle:   "学习和探索 Redis 中 string 的实现原理"
date:       2017-01-06
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Redis
---

> 本篇将介绍 Redis 中 string 的实现原理 的相关知识


# Redis

redis 数据库中每个键值对 (key－value) 都是一个对象 (Object) 。

redis 数据库中的键都为 string 类型。redis 支持五种数据结构类型，string 字符串、list 链表、hash 哈希、set 集合、sort set 有序的集合。




### redis 字符串
在 redis 数据库中，最常使用那就是字符串了，在c语言的字符串只会做为字符串字面量，不能对其进行修改。而 redis 对字符串的实现在底层由 **SDS** ，SDS 是 Simple Dynamic String 的缩写(简单动态字符串)。



#### SDS 在 redis 中的定义
在 redis 源码 sds.h 定义了结构表示 SDS 。
	
	struct __attribute__ ((__packed__)) sdshdr5 {
	    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
	    char buf[];
	};
	struct __attribute__ ((__packed__)) sdshdr8 {
	    uint8_t len; /* used */
	    uint8_t alloc; /* excluding the header and null terminator */
	    unsigned char flags; /* 3 lsb of type, 5 unused bits */
	    char buf[];
	};
	struct __attribute__ ((__packed__)) sdshdr16 {
	    uint16_t len; /* used */
	    uint16_t alloc; /* excluding the header and null terminator */
	    unsigned char flags; /* 3 lsb of type, 5 unused bits */
	    char buf[];
	};

- len 表示字符串真实长度
- alloc 表示该字符串最大的容量(不包括最后的"\0")
- flags 总是占用一个字节。其中的最低3个bit用来表示header的类型。header的类型共有5种，在sds.h中有常量定义。

		 #define SDS_TYPE_5  0
		 #define SDS_TYPE_8  1
		 #define SDS_TYPE_16 2
		 #define SDS_TYPE_32 3
		 #define SDS_TYPE_64 4 

- buf[] buf是一个 char 数组

这样做有什么好处呢？

在c语言中，我们要获取一个字符串的长度，需要遍历整个字符串，对遇到的每一个字符进行计数，直到遇到字符串的结尾标记“\0”，遍历结束，获取字符串长度的时间复杂度为O(n)，是不是很低效？

redis 定义的 SDS 本身记录了字符串的当前长度以及 buf 字符串数组的剩余容量，获取字符串长度和获取剩余字符串数组容量的时间复杂度为O(1)，是不是高效了很多。


#### set 命令
set 命令：set key value [EX seconds] [PX milliseconds] [NX|XX]

- EX seconds – 设置键key的过期时间，单位时秒
- PX milliseconds – 设置键key的过期时间，单位时毫秒
- NX – 只有键key不存在的时候才会设置key的值
- XX – 只有键key存在的时候才会设置key的值

	127.0.0.1:6379> set name "zhongjianfeng"
	OK
	127.0.0.1:6379> get name
	"zhongjianfeng"

设置 EX seconds ，可以使键值对对象在有效期内保存，当过了有效期将其销毁。在集群下使用 redis 做为 session管理，可以使用这个参数实现 session 过期机制。


#### append 命令
append 命令可以在指定key对应的 value 后添加字符串

append 命令：append key value

	127.0.0.1:6379> get name
	"zhongjianfeng"
	127.0.0.1:6379> append name "01"
	(integer) 15
	127.0.0.1:6379> get name
	"zhongjianfeng01"
	
在 key 为 name 的 value 后添加 "01"


# 总结

redis 在设计字符串时精心设计，考虑到字符串的使用场景较多，字符串的键值对也经常变化，特地设计了 SDS ，典型的内存换性能方式。


