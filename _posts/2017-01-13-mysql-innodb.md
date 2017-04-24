---
layout:     post
title:      "学习InnoDB的实现"
subtitle:   "InnoDB的实现原理"
date:       2017-01-13
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - MySQL
---

> 学习MySQL InnoDB引擎的实现


# Master Thread
主线程 Master Thread  是MySQL 中最核心的线程，负责将缓存中的数据刷新回内存，保证数据的一致性，，包括脏读的刷新、合并插入缓冲、UNDO页的回收等等。


# IO Thread
IO Thread 用于处理IO请求，可以极大提升数据库的性能。MySQL的IO Thread 线程的个数在Windows平台下是可以设置的，在Linux平台下是不能更改的。

在MySQL中通过命令查询read、write IO Thread：

	
	mysql> show variables like 'innodb_%_io_threads';
	+-------------------------+-------+
	| Variable_name           | Value |
	+-------------------------+-------+
	| innodb_read_io_threads  | 4     |
	| innodb_write_io_threads | 4     |
	+-------------------------+-------+
	2 rows in set (0.00 sec)
	
默认情况下，MySQL开启四个 read IO Thread、四个 write IO Thread。


通过命令 show ENGINE INNODB STATUS\G; 可以查看IO线程的状态

	
	FILE I/O
	--------
	I/O thread 0 state: waiting for i/o request (insert buffer thread)
	I/O thread 1 state: waiting for i/o request (log thread)
	I/O thread 2 state: waiting for i/o request (read thread)
	I/O thread 3 state: waiting for i/o request (read thread)
	I/O thread 4 state: waiting for i/o request (read thread)
	I/O thread 5 state: waiting for i/o request (read thread)
	I/O thread 6 state: waiting for i/o request (write thread)
	I/O thread 7 state: waiting for i/o request (write thread)
	I/O thread 8 state: waiting for i/o request (write thread)
	I/O thread 9 state: waiting for i/o request (write thread)
	
### Purge Thread
Purge Thread 主要用来回收已经使用并分配的undo页，在InnoDB 1.1 前一直都是由 Master Thread 负责完成的，从 InnoDB 1.1 版本开始，purge 操作交由 Purge Thread 来完成，降低 Master Thread 的负担，提高CPU的使用率。在 InnoDB 1.1 版本中，可以开启一个 Purge Thread ，从 InnoDB 1.2 版本后可以开启多个 Purge Thread 。


	
	mysql> show variables like "innodb_purge_threads";
	+----------------------+-------+
	| Variable_name        | Value |
	+----------------------+-------+
	| innodb_purge_threads | 4     |
	+----------------------+-------+
	1 row in set (0.00 sec)
	
	
# 总结

InnoDB 引擎使用多线程处理数据，各功能都分配有不同线程，线程的设置影响着数据库的整体性能。




























