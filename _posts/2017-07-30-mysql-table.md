---
layout:     post
title:      "学习数据库分库分表"
subtitle:   "分库分表的场景"
date:       2017-07-30
author:     "JianFeng"
header-img: "img/blog/2017-07-30-mysql.jpg"
catalog: true
tags:
    - MySQL
---

> 学习数据库分库分表

# 单库单表
在学校时所做的项目可能用户量并不是很大，使用单库就能支撑我们的项目业务，是一种最常见的数据库设计，所需的数据可以直接在一个库中读出，一个库存放了项目所有数据。

# 单库多表
当数据量上来了，单表单库不能很好的支持业务，当一张表的数据达到一定数量级，性能会明显下降，如果使用的是InnoDB引擎，可能还好些，如果使用的是MyISAM 引擎那么在高并发情景下，可能会有很多慢SQL的出现。InnoDB在进行写操作时，会对需要操作的行加锁也就是行锁；而MyISAM采用的表锁策略，当对表中数据进行写操作时，会对表进行加锁也就是表锁。严重的可能会拖垮我们整个应用程序。

单库多表可以缓解这时候的高并发场景，像一个库中的user表可以有多张，起名 user_0000、user_0001、user_0002...user_0128，类似这样的表，通过分表规则对分别对128张表进行操作，这能够大大提高我们的数据库读写能力。特别是对锁表的MyISAM，效果尤甚。


# 多库多表
当数据量再上一个量级，存储可能又存在问题了，一个MySQL存储数据的数量是有限制的，数据集中会给性能带来影响。
多库多表可以缓解存储量的问题。 单台数据库服务器没法支撑业务发展，这个时候可以再对数据库进行水平区分。 

# 分库分表规则
使用什么规则来进行分库分表呢？

##### id分库分表
使用id来分库分表是最常用的策略，阿里内部使用的TDDL数据库中间件也是推荐采用这种形式，在进行初次分库分表时会犯一个错误，id使用自增的形式，自增只能确保单表内的id自增，不能保证整个数据库集群id自增，通常我们需要一些插件进行辅助，像uuid生成算法、阿里内部sequence插件。


##### 用户纬度分库分表
电商场景下，用户购买商品，需要保存用户的交易记录，如果按照用户的纬度分表，则每个用户的交易记录都保存在同一表中，所以很快很方便的查找到某用户的购买情况。但是不同商品可能存放在多个不同的表中，查询也是很费劲的。
反之，假如我按照商品来分库分表，这能很方便查找到购买该商品的用户，但是查询用户又会比较繁琐。


##### 时间纬度分库分表
按照时间纬度来分库分表能够很快的了解到一定时间段的数据，比如按时间纬度存储用户购买交易数据，可以让我们很方便查找到某个时间段内用户购买交易信息。但这也会给数据库带来很大的压力，相当于单库单表，会对单台服务器巨大压力。适合数据量小和实时性分析高的场景。

# 分库分表注意点

##### 避免跨库事务
跨库跨表事务是分库分表最常见的场景，操作起来不仅复杂，而且对效率会有一定影响。 

##### 相关数据放在同一个DB服务器上
机器出现故障是不可避免的，假如db0的数据依赖db1的数据，db1挂了那么这次查询就废了。放在同一个DB影响会小一些。

##### 避免联合查询
联合查询和跨库事务一样会有复杂的操作，跨库查询RT也会增大。

##### 尽量使用Long、int取模分库分表
之前做需求，想使用String进行hash后取模，这会造成很大性能消耗，所以建议使用Long、int分库分表。

根据业务场景确定切分字段；业务中根据什么字段去查询，就用什么字段去分表。

##### 分库分表宜多不宜少
要根据自己的业务需求判断现在和未来会产生的数据量，尽量避免后期可能遇到的二次拆分，数据迁移应该很麻烦。


# 总结
用好分库分表对我们应用的成长起至关重要的作用，合理的设计能方便以后系统的更新迭代。