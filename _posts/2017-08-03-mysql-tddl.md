---
layout:     post
title:      "TDDL 数据库中间件学习"
subtitle:   "TDDL 的学习和使用"
date:       2017-08-03
author:     "JianFeng"
header-img: "img/blog/2017-07-30-mysql.png"
catalog: true
tags:
    - MySQL
---

> TDDL 的学习和使用

# TDDL 产生背景
 
单一的数据库无法满足性能的需求，分库分表可以有效的缓解数据量大的压力，数据切分、读写分离、系统容灾和动态数据源可以让我们很好的管理数据，使我们的开发精力都放在业务开发上。这些功能能给我们的性能带来提升，但是访问数据库的操作却很困难。我们需要一个中间件，能以一定的规则进行访问数据库，TDDL（Taobao Distributed Data Layer）就是一个用于解决这个问题的工具。
 
# 分表分库
 
多库多表可以缓解存储量的问题。 单台数据库服务器没法支撑业务发展，这个时候可以再对数据库进行水平区分。
 
# 读写分离
 
对单一的数据库进行大量的读写操作，会带来很大的性能压力，每台DB所能承受的QPS是有限的，将读写的压力分担到不同的机器，一台master、两台salve，master主要负责数据库的写操作，salve负责数据库的读操作。
 
读写分离同样存在一些问题
 
## 怎么判断读写
 
读操作和写操作在不同的数据库执行，我们怎么判断是读操作还是写操作呢？我们只能在程序上判断是写操作还是读操作，然后再指定数据库执行。
 
数据之间的复制
 
读写分离后，数据都写到了 Master ，Salve的数据需要从 master 上同步，Master 和 Salve 之间数据的同步延时是没办法保证的，短在一秒内，长则几秒和几十秒都有可能。这段延时时间内，应用程序如果从 Salve 上取数据，往往达不到预期的效果。
 
## 解决方案
 
1.在 Master 进行写操作后，insert 和 update 后，强制sleep 几秒钟，这是非常简单粗暴的方式，对于中小型应用可以使用这种方式，但对大型应用程序和实时性要求很高的应用程序，这样是行不通的。
 
2.在应用程序和数据库之间加一个缓存，像 Redis、Mongodb 这类的缓存性能很高，Redis 每秒你能进行11万次的读操作，8万次的写操作，这对于一些普通应用是绰绰有余的。
 
# TDDL规则实例
 
<?xml version="1.0" encoding="gb2312"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
<beans>
  <!-- 这个bean配置为TDDL规则总配置 -->
  <bean id="vtabroot" class="com.taobao.tddl.interact.rule.VirtualTableRoot" init-method="init">
    <!-- 没有被配置在tableRules的逻辑表都将在这个group里，以单表形式执行 -->
    <!-- 注意，这里的YOUR_GROUP_KEY只是示例，使用时要填应用实际使用的GroupKey -->
    <property name="defaultDbIndex" value="YOUR_GROUP_KEY"/>  
    <!-- 数据库类型,默认是mysql -->
    <property name="dbType" value="MYSQL"></property> 
    <!-- 该map配置有分表的逻辑表，有几个表有分表就配置几个键值对(该事例表示只有三个表需要分表)-->
    <property name="tableRules">
      <map> 
        <!-- key是逻辑表名，value指的是对应具体配置的id -->
        <entry key="user" value-ref="user_bean"></entry> 
        <!-- 逻辑表名为admin，具体的分表规则在id="admin_bean"的配置中 -->
        <entry key="admin" value-ref="admin_bean"></entry> 
        <!-- 这张表是单表，可以配置在这，不配置的话默认走defaultDbIndex -->
        <entry key="picture" value-ref="picture_bean"></entry> 
      </map>
    </property>
  </bean>
 
配置该应用所有表的信息，这里配置了 user、admin、picture 三个表，这是整体的配置
 
我们还要对单表进行详细配置，例如配置user表：
 
  <bean id="user_bean" class="com.taobao.tddl.interact.rule.TableRule">
    <!-- groupKey格式框架，{}中的数将会被dbRuleArray的值替代，并保持位数 -->
    <property name="dbNamePattern" value="TDDL_{0000}_GROUP"/> 
    <!-- 具体的分库规则 -->
    <property name="dbRuleArray"> 
      <!-- 按照user_id取模划分64张表，结果除以32后分到两个库中 -->
      <value>(#user_id,1,64#.longValue() % 64).intdiv(32)</value> 
    </property>
    <!-- 具体表名格式框架，{}中的数将会被tbRuleArray的值替代，并保持位数 -->
    <property name="tbNamePattern" value="user_{0000}"></property> 
    <!-- 具体的分表规则 -->
    <property name="tbRuleArray"> 
      <!-- 按照user_id取模划分64张表 -->
      <value>#user_id,1,64#.longValue() % 64</value> 
    </property>
    <!-- 全表扫描开关，默认关闭，是否允许应用端在没有给定分表键值的情况下查询全表 -->
    <property name="allowFullTableScan" value="true"/> 
  </bean>
 
分了两个库和64张表
 
<value>(#user_id,1,64#.longValue() % 64).intdiv(32)</value> 
 
user_0000 - user_0031 分到了 TDDL_0000_GROUP 这个库
 
user_0032 - user_0063 分到了 TDDL_0001_GROUP 这个库
 
如果存在不分库分表的情况，可按照如下配置：
 
  <bean id="picture_bean" class="com.taobao.tddl.interact.rule.TableRule">
    <property name="dbNamePattern" value="YOUR_GROUP_KEY"/> 
    <property name="tbNamePattern" value="picture"></property>
  </bean>
