---
layout:     post
title:      "Mybatis Cache 设计及实现(二)"
subtitle:   "Mybatis Cache 源码解析"
date:       2016-10-28
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - Mybatis
---

> 本篇将介绍 Mybatis Cache 的相关知识


## Mybatis

### FifoCache 源码解读

FifoCache 采用的是先进先出页面调度算法，在 FifoCache 类定义了三个属性，Cache 缓存对象、keyList 使用 **LinkedList** 双向链表作为辅助容器实现先进先出缓存管理机制，size 为 keyList 的限制长度，默认为 **1024** 。

#### 构造方法

	  public FifoCache(Cache delegate) {
        //初始化 FifoCache 时，内部维护一个不限容量的LinkedList，名称为keyList，在构造方法中被创建，限制长度为 1024
	    this.delegate = delegate;
	    this.keyList = new LinkedList<Object>();
	    this.size = 1024;
	  }

#### clear() 方法

	 @Override
	  public void clear() {
	    delegate.clear();
	    keyList.clear();
	  }

调用 LinkedList 自身的clear()清空缓存。

#### putObject(Object key, Object value)方法

	@Override
	  public void putObject(Object key, Object value) {
	    cycleKeyList(key);
	    delegate.putObject(key, value);
	  }

调用 cycleKeyList(key) 方法，并向 HashMap 中加入缓存对象。


#### cycleKeyList(Object key) 方法

	private void cycleKeyList(Object key) {
	    keyList.addLast(key);
        //判断 keyList 长度是否超过限制长度，如果超过了，那么将第一个节点对象删除
	    if (keyList.size() > size) {
          //调用 LinkedList removeFirst() 删除第一个节点对象，并返回该对象，再到 HashMap 中删除该对象
	      Object oldestKey = keyList.removeFirst();
	      delegate.removeObject(oldestKey);
	    }
	}

cycleKeyList(Object key) 方法在增加缓存对象时会被调用，实现了先进先出缓存机制。removeFirst()方法实现如下：


	public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

如果链表为空，抛出异常。不为空调用 unlinkFirst(Node<E> f) 方法，实现如下：

	private E unlinkFirst(Node<E> f) {
        //临时保存对象，用于返回被删除的对象
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // 辅助gc
        //第一个节点指向下一个节点
        first = next;
        if (next == null)
            last = null;
        else
            //双向链表 next 前驱也要为 null
            next.prev = null;
        size--;
        modCount++;
        return element;
    }


### LruCache 源码解读

LruCache 采用最近最少使用的回收策略，LruCache 类定义了三个属性，Cache 缓存对象、keyMap 维护一个 **LinkedHashMap** 、**eldestKey**。

#### 构造方法

	  public LruCache(Cache delegate) {
	    this.delegate = delegate;
	    setSize(1024);
	  }

获取 Cache 缓存对象，调用 setSize() 方法。

#### setSize(final int size) 方法

	  public void setSize(final int size) {
        //构造一个长度为1024 因子为0.75 的 LinkedHashMap
	    keyMap = new LinkedHashMap<Object, Object>(size, .75F, true) {
	      private static final long serialVersionUID = 4267176411845948333L;
	
          //重写 LinkedHashMap 的 removeEldestEntry() 方法，用于获取在达到容量限制时被删除的key
	      @Override
	      protected boolean removeEldestEntry(Map.Entry<Object, Object> eldest) {
	        boolean tooBig = size() > size;
	        if (tooBig) {
	          eldestKey = eldest.getKey();
	        }
	        return tooBig;
	      }
	    };
	  }

在每次调用setSize方法时，都会创建一个新的该类型的对象，同时指定其容量大小。第三个参数为true代表Map中的键值对列表要按照访问顺序排序，每次被方位的键值对都会被移动到列表尾部（值为false时按照插入顺序排序）。

#### clear() 方法

	  @Override
	  public void clear() {
	    delegate.clear();
	    keyMap.clear();
	  }

清空缓存。

#### putObject(Object key, Object value) 方法

	  @Override
	  public void putObject(Object key, Object value) {
	    delegate.putObject(key, value);
	    cycleKeyList(key);
	  }

调用 Cache 对象本身的 putObject()方法，调用 cycleKeyList(Object key)方法。每次添加完键值对后，都会调用 cycleKeyList() 方法进行一次检查。

#### cycleKeyList(Object key) 方法

	  private void cycleKeyList(Object key) {
        //将key添加到keyMap中
	    keyMap.put(key, key);
	    if (eldestKey != null) {
          //把缓存对象中key为eldestKey的键值对删除
	      delegate.removeObject(eldestKey);
	      eldestKey = null;
	    }
	  }

### LoggingCache 源码解读

LoggingCache 主要用于打印缓存命中的日志信息，在 LoggingCache 类中主要定义如下几个属性：

      //日志对象
	  private Log log;  
      //缓存对象
	  private Cache delegate;
      //每次调用getObject(key)查询时，此值+1
	  protected int requests = 0;
      //每次调用getObject(key)获取到的value不为null时，此值+1
	  protected int hits = 0;

#### 构造方法

	  public LoggingCache(Cache delegate) {
	    this.delegate = delegate;
	    this.log = LogFactory.getLog(getId());
	  }

通过缓存对象的id从 LogFactory 中获取日志对象

#### getObject(Object key) 方法

	  @Override
	  public Object getObject(Object key) {
        //每次查询时，此值+1
	    requests++;
	    final Object value = delegate.getObject(key);
	    if (value != null) {
          //获取到的value不为null时，此值+1
	      hits++;
	    }
	    if (log.isDebugEnabled()) {
	      log.debug("Cache Hit Ratio [" + getId() + "]: " + getHitRatio());
	    }
	    return value;
	  }

#### getHitRatio() 方法

	private double getHitRatio() {
		return (double) hits / (double) requests;
	}
hits / requests 获取日志缓存的命中率
