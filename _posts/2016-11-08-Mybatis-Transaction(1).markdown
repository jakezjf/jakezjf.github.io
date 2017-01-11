---
layout:     post
title:      "Mybatis 事务管理的设计与实现(二)"
subtitle:   "Mybatis Transaction 源码解析"
date:       2016-11-08
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - Mybatis
---

> 本篇将介绍 Mybatis 事务管理的相关知识

## Mybatis 事务管理

### TransactionFactory 接口 
TransactionFactory 接口定义了抽象事务工厂生产方法。


	public interface TransactionFactory {
	
	  /**
	   * 设置事务工厂自定义属性
	   * @param props
	   */
	  void setProperties(Properties props);
	
	  /**
	   * 在数据库连接中创建事务
	   * @param conn Existing database connection
	   * @return Transaction
	   * @since 3.1.0
	   */
	  Transaction newTransaction(Connection conn);
	  
	  /**
	   * 创建事务，定义了JDBC源，事务隔离级别，提交事务方式
	   * @param dataSource DataSource to take the connection from
	   * @param level Desired isolation level
	   * @param autoCommit Desired autocommit
	   * @return Transaction
	   * @since 3.1.0
	   */
	  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);
	
	}

**TransactionFactory** 接口有两个实现类： **JdbcTransactionFactory** 、 **ManagedTransactionFactory** 。

### JdbcTransactionFactory 实现类
JdbcTransactionFactory 类实现了 TransactionFactory接口 、用于生产JdbcTransaction的工厂类。

##### setProperties() 方法



##### newTransaction(Connection conn) 方法

	  @Override
	  public Transaction newTransaction(Connection conn) {
	    return new JdbcTransaction(conn);
	  }

从获取的数据库连接中创建事务。

### ManagedTransactionFactory 实现类
**ManagedTransactionFactory** 类实现了 **TransactionFactory** 接口、用于生产ManagedTransaction的工厂类。

##### setProperties(Properties props) 方法

##### newTransaction(Connection conn) 方法