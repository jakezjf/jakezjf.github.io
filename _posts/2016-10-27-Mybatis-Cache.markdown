---
layout:     post
title:      "Mybatis Cache 设计及实现"
subtitle:   "Mybatis Cache 源码解析"
date:       2016-10-27
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

**Mybatis** 是在java web开发中一个重要的 ORM 框架，Mybatis 支持定制化的 SQL、存储过程以及高级映射的优秀的持久层框架。Mybatis 对 JDBC 进行了封装，避免了复杂的 SQL 语句和一些手动参数。

Mybatis 的 Cache (本地缓存)是十分重要的，Mybatis 默认开启的是本地高速缓存，也可以在配置文件中选择开启第二级缓存。

	localCacheScope=SESSION|STATEMENT     (default = SESSION)

## Mybatis 缓存

在 mybatis 中的 (package org.apache.ibatis.cache) 定义了 Cache 接口，整个缓存系统采用了装饰器模式，cache 包中通过 PerpetualCache 类实现 Cache 接口，来实现数据存储和缓存的基本功能。


Cache 接口定义了如下方法：

	public interface Cache {
	
	  /**
	   * 获取 Cache 的唯一标识Id
	   */
	  String getId();
	
	  /**
	   * 通过key 和 value 加入缓存对象中
	   * key/value 可以为任何对象
	   */
	  void putObject(Object key, Object value);
	
	  /**
	   * 通过 key 获取 value
	   */
	  Object getObject(Object key);
	
	  /**
	   * 通过 key 移除缓存对象
	   */
	  Object removeObject(Object key);
	
	  /**
	   * 清空缓存
	   */  
	  void clear();
	
	  /**
	   * 获取缓存对象的数量
	   */
	  int getSize();
	  
	  /** 
	   * 获取读写锁
	   */
	  ReadWriteLock getReadWriteLock();
	
	}

在 (package org.apache.ibatis.cache.decorators)包中，实现了8个用于装饰 PerpetualCache 的装饰器。分别是：

- BlockingCache ： 简单低效的可阻塞装饰器
- FifoCache ： 使用先进先出算法缓存回收策略
- LoggingCache ： 用于打印缓存命中的日志信息
- LruCache ： 使用LRU最近最少使用算法缓存回收策略
- ScheduledCache ： 调度缓存，定义 **clearInterval** 属性，定时清空缓存
- SerializedCache ： 缓存序列化和反序列化存储
- SoftCache ： 基于软引用实现的缓存管理策略
- SynchronizedCache ： 同步缓存管理策略，通过 synchronized 关键字实现线程安全
- TransactionalCache ： 二级缓存事务管理策略
- WeakCache ： 基于弱引用实现的缓存管理策略

### PerpetualCache 源码解读

	public class PerpetualCache implements Cache {
	  //id 作为缓存的唯一标识
	  private String id;
	  //使用 HashMap 实现缓存策略
	  private Map<Object, Object> cache = new HashMap<Object, Object>();
	
	  public PerpetualCache(String id) {
	    this.id = id;
	  }
	
	  @Override
	  public String getId() {
	    return id;
	  }
	
	  @Override
	  public int getSize() {
	    return cache.size();
	  }
	
	  @Override
	  public void putObject(Object key, Object value) {
	    cache.put(key, value);
	  }
	
	  @Override
	  public Object getObject(Object key) {
	    return cache.get(key);
	  }
	
	  @Override
	  public Object removeObject(Object key) {
	    return cache.remove(key);
	  }
	
	  @Override
	  public void clear() {
	    cache.clear();
	  }
	
	  @Override
	  public ReadWriteLock getReadWriteLock() {
	    return null;
	  }
	
	  @Override
	  public boolean equals(Object o) {
	    if (getId() == null) {
	      throw new CacheException("Cache instances require an ID.");
	    }
	    if (this == o) {
	      return true;
	    }
	    if (!(o instanceof Cache)) {
	      return false;
	    }
	
	    Cache otherCache = (Cache) o;
	    return getId().equals(otherCache.getId());
	  }
	
	  @Override
	  public int hashCode() {
	    if (getId() == null) {
	      throw new CacheException("Cache instances require an ID.");
	    }
	    return getId().hashCode();
	  }
	
	}

**PerpetualCache** 是 Cache 中唯一的一个基础实现，其他10个实现类作为装饰器需要持有另一个缓存对象。


### BlockingCache 源码解读
BlockingCache 对象内部维护一个 ConcurrentHashMap，使用 ReentrantLock 类来保证线程安全。
 
	public class BlockingCache implements Cache {
	
	  private long timeout;
      //delegate 为持有的缓存对象
	  private final Cache delegate;
      //对象内部维护一个 ConcurrentHashMap 
	  private final ConcurrentHashMap<Object, ReentrantLock> locks;
	
	  public BlockingCache(Cache delegate) {
	    this.delegate = delegate;
	    this.locks = new ConcurrentHashMap<Object, ReentrantLock>();
	  }
	
	  @Override
	  public String getId() {
	    return delegate.getId();
	  }
	
	  @Override
	  public int getSize() {
	    return delegate.getSize();
	  }
	
	  @Override
	  public void putObject(Object key, Object value) {
	    try {
	      delegate.putObject(key, value);
	    } finally {
	      releaseLock(key);
	    }
	  }
	
	  @Override
	  public Object getObject(Object key) {
	    acquireLock(key);
	    Object value = delegate.getObject(key);
	    if (value != null) {
	      releaseLock(key);
	    }        
	    return value;
	  }
	
	  @Override
	  public Object removeObject(Object key) {
	    // 该方法用于释放锁
	    releaseLock(key);
	    return null;
	  }
	
	  @Override
	  public void clear() {
	    delegate.clear();
	  }
	
	  @Override
	  public ReadWriteLock getReadWriteLock() {
	    return null;
	  }
	  
	  private ReentrantLock getLockForKey(Object key) {
	    ReentrantLock lock = new ReentrantLock();
	    ReentrantLock previous = locks.putIfAbsent(key, lock);
	    return previous == null ? lock : previous;
	  }
	  
	  private void acquireLock(Object key) {
	    Lock lock = getLockForKey(key);
	    if (timeout > 0) {
	      try {
	        boolean acquired = lock.tryLock(timeout, TimeUnit.MILLISECONDS);
	        if (!acquired) {
	          throw new CacheException("Couldn't get a lock in " + timeout + " for the key " +  key + " at the cache " + delegate.getId());  
	        }
	      } catch (InterruptedException e) {
	        throw new CacheException("Got interrupted while trying to acquire lock for key " + key, e);
	      }
	    } else {
	      lock.lock();
	    }
	  }
	  
	  private void releaseLock(Object key) {
	    ReentrantLock lock = locks.get(key);
        //查询当前线程是否保持此锁定
	    if (lock.isHeldByCurrentThread()) {
          //释放锁
	      lock.unlock();
	    }
	  }
	
      /*
       *获取和设置超时时间
       */
	  public long getTimeout() {
	    return timeout;
	  }
	
	  public void setTimeout(long timeout) {
	    this.timeout = timeout;
	  }  
	}