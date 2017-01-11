---
layout:     post
title:      "Mybatis 事务管理的设计与实现(一)"
subtitle:   "Mybatis Transaction 源码解析"
date:       2016-11-07
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - Mybatis
---

> 本篇将介绍 Mybatis 事务管理的相关知识

# Mybatis 事务

事务是数据库重要的组成部分，事务必须服从 **ACID** 原则，ACID 是指原子性(atomicity)、一致性(consistency)、隔离性(isolation)、持久性(durability)。

Mybatis 是java web常用的 ORM 框架，当然也实现了事务管理的功能。Mybatis 实现事务主要有两种方法： **JdbcTransaction**、 **ManagedTransaction**。

### Transaction 接口
**Transaction** 接口定义了 Mybatis 事务管理的功能，它实现类就是**JdbcTransaction**、 **ManagedTransaction**。看看它的接口定义了哪些功能。

	public interface Transaction {
	
	  /**
	   * 获取数据库连接
	   * @return DataBase connection
	   * @throws SQLException
	   */
	  Connection getConnection() throws SQLException;
	
	  /**
	   * 提交到数据库
	   * @throws SQLException
	   */
	  void commit() throws SQLException;
	
	  /**
	   * 回滚数据库
	   * @throws SQLException
	   */
	  void rollback() throws SQLException;
	
	  /**
	   * 关闭数据库连接
	   * @throws SQLException
	   */
	  void close() throws SQLException;
	
	  /**
	   * 设置事务超时时间
	   * @throws SQLException
	   */
	  Integer getTimeout() throws SQLException;
	  
	}

#### JdbcTransaction 实现类
JdbcTransaction 所做的工作主要是对数据库连接Connection的一个封装，内部管理数据库事务还是调用Connection的提交、回滚等事务操作方法。

##### 成员变量及构造方法


	  private static final Log log = LogFactory.getLog(JdbcTransaction.class);
	  //日志
	  protected Connection connection;
	  //数据库连接
	  protected DataSource dataSource;
	  //获取JDBC数据源
	  protected TransactionIsolationLevel level;
	  //设置事物隔离级别
	  protected boolean autoCommmit;
      //是否自动提交事务
	
	  public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
	    dataSource = ds;
	    level = desiredLevel;
	    autoCommmit = desiredAutoCommit;
	  }
	
	  public JdbcTransaction(Connection connection) {
	    this.connection = connection;
	  }

##### getConnection() 方法
返回数据库连接，如果连接为空调用 openConnection() 方法。

	  @Override
	  public Connection getConnection() throws SQLException {
	    if (connection == null) {
	      openConnection();
	    }
	    return connection;
	  }

##### openConnection() 方法

	  protected void openConnection() throws SQLException {
	    if (log.isDebugEnabled()) {
	      log.debug("Opening JDBC Connection");
	    }
	    connection = dataSource.getConnection();
	    if (level != null) {
	      connection.setTransactionIsolation(level.getLevel());
	    }
	    setDesiredAutoCommit(autoCommmit);
	  }

获取数据库连接，并且设置是否为自动提交。

##### setDesiredAutoCommit() 方法
此方法用来设置是否让事务自动提交，在实际开发中 Mybatis 是默认自动提交事务的。
	
	  protected void setDesiredAutoCommit(boolean desiredAutoCommit) {
	    try {
 		  //判断当前事务提交方式和期望的提交方式
	      if (connection.getAutoCommit() != desiredAutoCommit) {
	        if (log.isDebugEnabled()) {
	          log.debug("Setting autocommit to " + desiredAutoCommit + " on JDBC Connection [" + connection + "]");
	        }
			//设置事务提交方式为期望提交方式
	        connection.setAutoCommit(desiredAutoCommit);
	      }
	    } catch (SQLException e) {
	      throw new TransactionException("Error configuring AutoCommit.  "
	          + "Your driver may not support getAutoCommit() or setAutoCommit(). "
	          + "Requested setting: " + desiredAutoCommit + ".  Cause: " + e, e);
	    }
	  }

##### resetAutoCommit() 方法
重置事务提交方式，如果不是自动提交 (**true**)，则设置为自动提交 (**true**)。

	  protected void resetAutoCommit() {
	    try {
	      if (!connection.getAutoCommit()) {
	        if (log.isDebugEnabled()) {
	          log.debug("Resetting autocommit to true on JDBC Connection [" + connection + "]");
	        }
	        connection.setAutoCommit(true);
	      }
	    } catch (SQLException e) {
	      if (log.isDebugEnabled()) {
	        log.debug("Error resetting autocommit to true "
	          + "before closing the connection.  Cause: " + e);
	      }
	    }
	  }

##### TransactionIsolationLevel 隔离级别
前面提到过事务的隔离级别，有四种：

- 脏读
- 不可重复读
- 可重复读
- 串行

TransactionIsolationLevel 是枚举类型

	public enum TransactionIsolationLevel {
	  NONE(Connection.TRANSACTION_NONE),
	  READ_COMMITTED(Connection.TRANSACTION_READ_COMMITTED),
	  READ_UNCOMMITTED(Connection.TRANSACTION_READ_UNCOMMITTED),
	  REPEATABLE_READ(Connection.TRANSACTION_REPEATABLE_READ),
	  SERIALIZABLE(Connection.TRANSACTION_SERIALIZABLE);
	
	  private final int level;
	
	  private TransactionIsolationLevel(int level) {
	    this.level = level;
	  }
	
	  public int getLevel() {
	    return level;
	  }
	}

定义了四种级别及一个 NONE 。

#### ManagedTransaction 实现类
ManagedTransaction 类让容器来管理事务 Transaction 的整个周期。

##### 成员变量及构造方法
	  private static final Log log = LogFactory.getLog(ManagedTransaction.class);
	  //日志
	  private DataSource dataSource;
	  //JDBC数据源
	  private TransactionIsolationLevel level;
	  //设置隔离级别
	  private Connection connection;
	  //数据库连接
	  private boolean closeConnection;
	
	  public ManagedTransaction(Connection connection, boolean closeConnection) {
	    this.connection = connection;
	    this.closeConnection = closeConnection;
	  }
	
	  public ManagedTransaction(DataSource ds, TransactionIsolationLevel level, boolean closeConnection) {
	    this.dataSource = ds;
	    this.level = level;
	    this.closeConnection = closeConnection;
	  }

##### commit() 方法
	  @Override
	  public void commit() throws SQLException {
	    // Does nothing
	  }

什么也没有做，交给容器管理 Transaction 。

##### rollback() 方法

	  @Override
	  public void rollback() throws SQLException {
	    // Does nothing
	  }

什么也没有做，交给容器管理 Transaction 。

##### getConnection() 方法

	  @Override
	  public Connection getConnection() throws SQLException {
	    if (this.connection == null) {
	      openConnection();
	    }
	    return this.connection;
	  }

获取数据库连接。

##### close() 方法
	  @Override
	  public void close() throws SQLException {
	    if (this.closeConnection && this.connection != null) {
	      if (log.isDebugEnabled()) {
	        log.debug("Closing JDBC Connection [" + this.connection + "]");
	      }
	      this.connection.close();
	    }
	  }

关闭数据库连接。

##### openConnection() 方法

	  protected void openConnection() throws SQLException {
	    if (log.isDebugEnabled()) {
	      log.debug("Opening JDBC Connection");
	    }
	    this.connection = this.dataSource.getConnection();
	    if (this.level != null) {
	      this.connection.setTransactionIsolation(this.level.getLevel());
	    }
	  }

当事务没有获取到数据库连接时，调用 openConnection() 方法，获取连接。

### Mybatis 中的事务配置
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driverClassName}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

Mybatis中关于事务的配置是通过<transaction type="xx"/>来指定的。

type为**”JDBC”**时、使用JdbcTransaction管理事务。

type为**”managed”**时、使用ManagedTransaction管理事务(也就是交由外部容器管理)。




