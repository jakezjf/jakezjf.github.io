---
layout:     post
title:      "Mybatis SQL构造器设计与实现"
subtitle:   "Mybatis AbstractSQL 源码解析"
date:       2016-11-02
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - mybatis
---

> 本篇将介绍 Mybatis AbstractSQL 的相关知识

## Mybatis
Mybatis 是常用的 java ORM 框架，Mybatis实现了一些数据库操作的功能，构造动态SQL是一个常用的功能。

构造动态SQL的相关类有 SQL 、AbstractSQL 、SelectBuilder 和SqlBuilder 等。在 Mybatis3.0以上版本，SelectBuilder 和 SqlBuilder 两个类被废弃了。主要由 **SQL** 和 **AbstractSQL** 实现构造动态SQL。

### 源码解析
SQL 、AbstractSQL 、SelectBuilder 和 SqlBuilder 这几个类都在 **package org.apache.ibatis.jdbc** 包下

#### SelectBuilder
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

SelectBuilder 中将构造器私有化，防止被外部实例化。SelectBuilder 中定义了一个成员变量 ThreadLocal<T> 类，静态初始化执行 **BEGIN() 方法** 。

