---
layout:     post
title:      "MySQL InnoDB引擎学习(一)"
subtitle:   "学习 MySQL InnoDB引擎"
date:       2016-12-08
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - MySQL
---

> 本篇将介绍 MySQL InnoDB 存储引擎的相关知识

# MySQL InnoDB 存储引擎
MySQL 数据库区别于其他数据库的最重要的一个特点是 MySQL 有表插件式表存储引擎，插件式存储引擎的种类繁多，能够在各种不同的应用场景可以使用不同的存储引擎。对于开发人员来说，存储引擎是对其透明的，每个存储引擎有各自的特点，可供开发人员根据不同的表结构选用合适的存储引擎。

InnoDB 是 MySQL 数据库 OLTP (Online Transaction Processing) 在线事务处理应用中最广泛的。 InnoDB 存储引擎支持事务处理，它的使用场景主要在 OLTP 在线事务处理中，同时 InnoDB 的特点是行锁设计、支持外键、多版本并发控制 MVCC 、支持非锁定读。


## InnoDB 后台线程


###### Master Thread
从字面就能看出， **Master Thread** 是 MySQL 中十分重要的线程，该线程主要负责将缓冲池的数据刷新到内存中，确保数据的一致性。

