---
layout:     post
title:      "Mybatis 构造动态SQL语句功能的设计与实现"
subtitle:   "Mybatis AbstractSQL 源码解析"
date:       2016-11-02
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - Mybatis
---

> 本篇将介绍 Mybatis 构造动态SQL语句 的相关知识

## SQL语句拼接
在之前学习 JDBC 过程中，经常会遇到SQL语句的拼接问题，在Java代码中嵌入SQL是最讨厌的事情之一。

例如：
	
	String sql = "SELECT P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME, " +
	"P.LAST_NAME,P.CREATED_ON, P.UPDATED_ON " +
	"FROM PERSON P, ACCOUNT A " +
	"INNER JOIN DEPARTMENT D on D.ID = P.DEPARTMENT_ID " +
	"INNER JOIN COMPANY C on D.COMPANY_ID = C.ID " +
	"WHERE (P.ID = A.ID AND P.FIRST_NAME like ?) " +
	"OR (P.LAST_NAME like ?) " +
	"GROUP BY P.ID " +
	"HAVING (P.LAST_NAME like ?) "

这样看起来十分混乱，不好维护，用String拼接也会在JVM产生大量的对象，影响性能。 Mybatis 的出现解决了 SQL 拼接的问题。

## Mybatis
Mybatis 是常用的 java ORM 框架，Mybatis实现了一些数据库操作的功能，构造动态SQL是一个常用的功能。

构造动态SQL的相关类有 SQL 、AbstractSQL 、SelectBuilder 和SqlBuilder 等。在 Mybatis3.0以上版本，SelectBuilder 和 SqlBuilder 两个类被废弃了。主要由 **SQL** 和 **AbstractSQL** 实现构造动态SQL。

## 源码解析
SQL 、AbstractSQL 、SelectBuilder 和 SqlBuilder 这几个类都在 **package org.apache.ibatis.jdbc** 包下

### SelectBuilder
首先来看看 SelectBuilder 的实现

#### 成员变量和构造器

	public class SelectBuilder {
	
	  private static final ThreadLocal<SQL> localSQL = new ThreadLocal<SQL>();
	
	  static {
	    BEGIN();
	  }
	
	  private SelectBuilder() {
	    // Prevent Instantiation
	  }

	}

SelectBuilder 中将构造器私有化，防止被外部实例化。SelectBuilder 中定义了一个成员变量 ThreadLocal<T> 类，静态初始化执行 BEGIN() 方法 。


##### RESET() 方法

	  public static void RESET() {
	    localSQL.set(new SQL());
	  }

初始化时调用 BEGIN() 方法，BEGIN() 方法调用 RESET() 方法，该方法将 **SQL** 创建一个对象，以 **ThreadLocal** 对象为key，**SQL** 对象为value，实现多线程之间的隔离。

##### sql() 方法

	  private static SQL sql() {
	    return localSQL.get();
	  }

从 localSQL 中，以 ThreadLocal 为key获取 **SQL** 对象，进行下面的操作。

##### SELECT()、FROM() 等方法

	  public static void SELECT(String columns) {
	    sql().SELECT(columns);
	  }
	
	  public static void SELECT_DISTINCT(String columns) {
	    sql().SELECT_DISTINCT(columns);
	  }
	
	  public static void FROM(String table) {
	    sql().FROM(table);
	  }
	
	  public static void JOIN(String join) {
	    sql().JOIN(join);
	  }
	
	  public static void INNER_JOIN(String join) {
	    sql().INNER_JOIN(join);
	  }
	
	  public static void LEFT_OUTER_JOIN(String join) {
	    sql().LEFT_OUTER_JOIN(join);
	  }
	
	  public static void RIGHT_OUTER_JOIN(String join) {
	    sql().RIGHT_OUTER_JOIN(join);
	  }
	
	  public static void OUTER_JOIN(String join) {
	    sql().OUTER_JOIN(join);
	  }
	
	  public static void WHERE(String conditions) {
	    sql().WHERE(conditions);
	  }
	
	  public static void OR() {
	    sql().OR();
	  }
	
	  public static void AND() {
	    sql().AND();
	  }
	
	  public static void GROUP_BY(String columns) {
	    sql().GROUP_BY(columns);
	  }
	
	  public static void HAVING(String conditions) {
	    sql().HAVING(conditions);
	  }
	
	  public static void ORDER_BY(String columns) {
	    sql().ORDER_BY(columns);
	  }

##### SQL() 方法

	  public static String SQL() {
	    try {
	      return sql().toString();
	    } finally {
	      RESET();
	    }
	  }

该方法用来获取构造的动态 SQL 语句，并将调用 **RESET()** 方法，将其重置。

### SelectBuilder
SelectBuilder 的实现和 SelectBuilder 类似

### SQL 、AbstractSQL 
SQL 类继承了 AbstractSQL 类，并重写了 **getSelf() 方法** 

##### getSelf() 方法

	  @Override
	  public SQL getSelf() {
	    return this;
	  }


##### 成员变量

	public abstract class AbstractSQL<T> {
	
	  private static final String AND = ") \nAND (";
	  private static final String OR = ") \nOR (";
	
	  private SQLStatement sql = new SQLStatement();
	
	}

定义了 AND 和 OR 字符串常量，用于 SQL 语句拼接。静态内部类 SQLStatement ，SQLStatement 内部为每个 SQL 关键字(例如：select、join、where、having、groupBy、orderBy)维护一个 **ArrayList** 数组，存放相应的条件语句。

##### 主要方法

	  public T UPDATE(String table) {
	    sql().statementType = SQLStatement.StatementType.UPDATE;
	    sql().tables.add(table);
	    return getSelf();
	  }
	
	  public T SET(String sets) {
	    sql().sets.add(sets);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T SET(String... sets) {
	    sql().sets.addAll(Arrays.asList(sets));
	    return getSelf();
	  }
	
	  public T INSERT_INTO(String tableName) {
	    sql().statementType = SQLStatement.StatementType.INSERT;
	    sql().tables.add(tableName);
	    return getSelf();
	  }
	
	  public T VALUES(String columns, String values) {
	    sql().columns.add(columns);
	    sql().values.add(values);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T INTO_COLUMNS(String... columns) {
	    sql().columns.addAll(Arrays.asList(columns));
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T INTO_VALUES(String... values) {
	    sql().values.addAll(Arrays.asList(values));
	    return getSelf();
	  }
	
	  public T SELECT(String columns) {
	    sql().statementType = SQLStatement.StatementType.SELECT;
	    sql().select.add(columns);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T SELECT(String... columns) {
	    sql().statementType = SQLStatement.StatementType.SELECT;
	    sql().select.addAll(Arrays.asList(columns));
	    return getSelf();
	  }
	
	  public T SELECT_DISTINCT(String columns) {
	    sql().distinct = true;
	    SELECT(columns);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T SELECT_DISTINCT(String... columns) {
	    sql().distinct = true;
	    SELECT(columns);
	    return getSelf();
	  }
	
	  public T DELETE_FROM(String table) {
	    sql().statementType = SQLStatement.StatementType.DELETE;
	    sql().tables.add(table);
	    return getSelf();
	  }
	
	  public T FROM(String table) {
	    sql().tables.add(table);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T FROM(String... tables) {
	    sql().tables.addAll(Arrays.asList(tables));
	    return getSelf();
	  }
	
	  public T JOIN(String join) {
	    sql().join.add(join);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T JOIN(String... joins) {
	    sql().join.addAll(Arrays.asList(joins));
	    return getSelf();
	  }
	
	  public T INNER_JOIN(String join) {
	    sql().innerJoin.add(join);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T INNER_JOIN(String... joins) {
	    sql().innerJoin.addAll(Arrays.asList(joins));
	    return getSelf();
	  }
	
	  public T LEFT_OUTER_JOIN(String join) {
	    sql().leftOuterJoin.add(join);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T LEFT_OUTER_JOIN(String... joins) {
	    sql().leftOuterJoin.addAll(Arrays.asList(joins));
	    return getSelf();
	  }
	
	  public T RIGHT_OUTER_JOIN(String join) {
	    sql().rightOuterJoin.add(join);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T RIGHT_OUTER_JOIN(String... joins) {
	    sql().rightOuterJoin.addAll(Arrays.asList(joins));
	    return getSelf();
	  }
	
	  public T OUTER_JOIN(String join) {
	    sql().outerJoin.add(join);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T OUTER_JOIN(String... joins) {
	    sql().outerJoin.addAll(Arrays.asList(joins));
	    return getSelf();
	  }
	
	  public T WHERE(String conditions) {
	    sql().where.add(conditions);
	    sql().lastList = sql().where;
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T WHERE(String... conditions) {
	    sql().where.addAll(Arrays.asList(conditions));
	    sql().lastList = sql().where;
	    return getSelf();
	  }
	
	  public T OR() {
	    sql().lastList.add(OR);
	    return getSelf();
	  }
	
	  public T AND() {
	    sql().lastList.add(AND);
	    return getSelf();
	  }
	
	  public T GROUP_BY(String columns) {
	    sql().groupBy.add(columns);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T GROUP_BY(String... columns) {
	    sql().groupBy.addAll(Arrays.asList(columns));
	    return getSelf();
	  }
	
	  public T HAVING(String conditions) {
	    sql().having.add(conditions);
	    sql().lastList = sql().having;
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T HAVING(String... conditions) {
	    sql().having.addAll(Arrays.asList(conditions));
	    sql().lastList = sql().having;
	    return getSelf();
	  }
	
	  public T ORDER_BY(String columns) {
	    sql().orderBy.add(columns);
	    return getSelf();
	  }
	
	  /**
	   * @since 3.4.2
	   */
	  public T ORDER_BY(String... columns) {
	    sql().orderBy.addAll(Arrays.asList(columns));
	    return getSelf();
	  }

这些方法的参数一般为一个 String 或者一个 String 数组，然后将 String 或者 String 数组添加到 SQLStatement 类对应的成员变量中，下面会提到。


##### sql() 方法
	  private SQLStatement sql() {
	    return sql;
	  }

sql() 方法获取 SQLStatement 对象。

#### 静态内部类 SQLStatement

##### 成员变量和构造器

	private static class SQLStatement {
	
	    public enum StatementType {
	      DELETE, INSERT, SELECT, UPDATE
	    }
	
	    StatementType statementType;
	    List<String> sets = new ArrayList<String>();
	    List<String> select = new ArrayList<String>();
	    List<String> tables = new ArrayList<String>();
	    List<String> join = new ArrayList<String>();
	    List<String> innerJoin = new ArrayList<String>();
	    List<String> outerJoin = new ArrayList<String>();
	    List<String> leftOuterJoin = new ArrayList<String>();
	    List<String> rightOuterJoin = new ArrayList<String>();
	    List<String> where = new ArrayList<String>();
	    List<String> having = new ArrayList<String>();
	    List<String> groupBy = new ArrayList<String>();
	    List<String> orderBy = new ArrayList<String>();
	    List<String> lastList = new ArrayList<String>();
	    List<String> columns = new ArrayList<String>();
	    List<String> values = new ArrayList<String>();
	    boolean distinct;
	
	    public SQLStatement() {
	        // Prevent Synthetic Access
	    }
	}

SQLStatement 内部为每个 SQL 关键字(例如：select、join、where、having、groupBy、orderBy)维护一个 **ArrayList** 数组，存放相应的条件语句。StatementType 枚举数据库四种操作。


##### sqlClause() 方法

    private void sqlClause(SafeAppendable builder, String keyword, List<String> parts, String open, String close,
                           String conjunction) {
      if (!parts.isEmpty()) {
        //判断parts是否已经初始化
        if (!builder.isEmpty()) {
          //判断builder是否已经初始化
          builder.append("\n");
        }
        builder.append(keyword);
        builder.append(" ");
        builder.append(open);
        String last = "________";
        for (int i = 0, n = parts.size(); i < n; i++) {
          String part = parts.get(i);
          if (i > 0 && !part.equals(AND) && !part.equals(OR) && !last.equals(AND) && !last.equals(OR)) {
            //判断是否为AND OR 关键字
            builder.append(conjunction);
          }
          builder.append(part);
          last = part;
        }
        builder.append(close);
      }
    }

sqlClause() 方法中的参数 SafeAppendable 对象持有 **Appendable** ，Appendable 接口有许多实现类，例如：StringBuffer 、Writer 、PrintStream 等，可以满足多种 io 方式的需求。

keyword 是 SQL 语句的关键字。parts 该 keyword 下的条件语句。open 和 close 一般作为区分，例如：where()，open 和 close 分别相当于 **(** 和 **)**，conjunction 连接。


##### sql(Appendable a) 方法


    private String selectSQL(SafeAppendable builder) {
      if (distinct) {
        sqlClause(builder, "SELECT DISTINCT", select, "", "", ", ");
      } else {
        sqlClause(builder, "SELECT", select, "", "", ", ");
      }

      sqlClause(builder, "FROM", tables, "", "", ", ");
      sqlClause(builder, "JOIN", join, "", "", "\nJOIN ");
      sqlClause(builder, "INNER JOIN", innerJoin, "", "", "\nINNER JOIN ");
      sqlClause(builder, "OUTER JOIN", outerJoin, "", "", "\nOUTER JOIN ");
      sqlClause(builder, "LEFT OUTER JOIN", leftOuterJoin, "", "", "\nLEFT OUTER JOIN ");
      sqlClause(builder, "RIGHT OUTER JOIN", rightOuterJoin, "", "", "\nRIGHT OUTER JOIN ");
      sqlClause(builder, "WHERE", where, "(", ")", " AND ");
      sqlClause(builder, "GROUP BY", groupBy, "", "", ", ");
      sqlClause(builder, "HAVING", having, "(", ")", " AND ");
      sqlClause(builder, "ORDER BY", orderBy, "", "", ", ");
      return builder.toString();
    }

    private String insertSQL(SafeAppendable builder) {
      sqlClause(builder, "INSERT INTO", tables, "", "", "");
      sqlClause(builder, "", columns, "(", ")", ", ");
      sqlClause(builder, "VALUES", values, "(", ")", ", ");
      return builder.toString();
    }

    private String deleteSQL(SafeAppendable builder) {
      sqlClause(builder, "DELETE FROM", tables, "", "", "");
      sqlClause(builder, "WHERE", where, "(", ")", " AND ");
      return builder.toString();
    }

    private String updateSQL(SafeAppendable builder) {

      sqlClause(builder, "UPDATE", tables, "", "", "");
      sqlClause(builder, "SET", sets, "", "", ", ");
      sqlClause(builder, "WHERE", where, "(", ")", " AND ");
      return builder.toString();
    }

    public String sql(Appendable a) {
      SafeAppendable builder = new SafeAppendable(a);
      if (statementType == null) {
        return null;
      }

      String answer;

      switch (statementType) {
        case DELETE:
          answer = deleteSQL(builder);
          break;

        case INSERT:
          answer = insertSQL(builder);
          break;

        case SELECT:
          answer = selectSQL(builder);
          break;

        case UPDATE:
          answer = updateSQL(builder);
          break;

        default:
          answer = null;
      }

      return answer;
    }

## 测试
	
	  @Test
	  public void test() {
	    final StringBuilder sb = new StringBuilder();
	    final String sql = new SQL() {{
	      SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME");
	      SELECT("P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON");
	      FROM("PERSON P");
	      FROM("ACCOUNT A");
	      INNER_JOIN("DEPARTMENT D on D.ID = P.DEPARTMENT_ID");
	      INNER_JOIN("COMPANY C on D.COMPANY_ID = C.ID");
	      WHERE("P.ID = A.ID");
	      WHERE("P.FIRST_NAME like ?");
	      OR();
	      WHERE("P.LAST_NAME like ?");
	      GROUP_BY("P.ID");
	      HAVING("P.LAST_NAME like ?");
	      OR();
	      HAVING("P.FIRST_NAME like ?");
	      ORDER_BY("P.ID");
	      ORDER_BY("P.FULL_NAME");
	    }}.usingAppender(sb).toString();
	  }

执行以上测试用例，可以得到以下 SQL：

	SELECT P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME, P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON
	FROM PERSON P, ACCOUNT A
	INNER JOIN DEPARTMENT D on D.ID = P.DEPARTMENT_ID
	INNER JOIN COMPANY C on D.COMPANY_ID = C.ID
	WHERE (P.ID = A.ID AND P.FIRST_NAME like ?) 
	OR (P.LAST_NAME like ?)
	GROUP BY P.ID
	HAVING (P.LAST_NAME like ?) 
	OR (P.FIRST_NAME like ?)
	ORDER BY P.ID, P.FULL_NAME


## 总结
实际上，AbstractSQL这个类单独来看，并不能给我们构造SQL语句带来什么方便，反而是把语句拆分特别复杂，还不如直接写来得方便。但是结合MyBatis的Annotation和Statement解析器，这个自动过程下，它的功能就显得特别强大了。

