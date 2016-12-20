---
layout:     post
title:      "Mybatis Cache 设计及实现(四)"
subtitle:   "Mybatis Cache 源码解析"
date:       2016-11-01
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - mybatis
    - cache
---

> 本篇将介绍 Mybatis Cache 的相关知识

## Mybatis

### SoftCache 软引用实现的缓存管理

#### 构造方法

	  public SoftCache(Cache delegate) {
	    this.delegate = delegate;
	    this.numberOfHardLinks = 256;
	    this.hardLinksToAvoidGarbageCollection = new LinkedList<Object>();
	    this.queueOfGarbageCollectedEntries = new ReferenceQueue<Object>();
	  }
SoftCache对象内部维护一个双链表 **LinkedList** 和一个引用队列 **ReferenceQueue** ，**ReferenceQueue** 限制了双链表的长度，默认为256。


