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

在JVM垃圾回收机制中，如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

#### 构造方法

	  public SoftCache(Cache delegate) {
	    this.delegate = delegate;
	    this.numberOfHardLinks = 256;
	    this.hardLinksToAvoidGarbageCollection = new LinkedList<Object>();
	    this.queueOfGarbageCollectedEntries = new ReferenceQueue<Object>();
	  }

SoftCache对象内部维护一个双链表 **LinkedList** 和一个引用队列 **ReferenceQueue** ，**ReferenceQueue** 限制了双链表的长度，默认为256。

#### removeGarbageCollectedItems() 方法

	  private void removeGarbageCollectedItems() {
	    SoftEntry sv;
		//弹出队列中的所有key，并删除
	    while ((sv = (SoftEntry) queueOfGarbageCollectedEntries.poll()) != null) {
	      delegate.removeObject(sv.key);
	    }
	  }

removeGarbageCollectedItems() 将缓存对象中被JVM回收的 value 对象的key删除。

SoftCache 在写缓存之前，会先调用 **removeGarbageCollectedItems()** 方法删除已经被垃圾回收器回收的 key和value ，之后想缓存对象中写入 **SoftEntry** 类型的对象（定义在SoftCache的内部，是SoftReference类的子类）。

	private static class SoftEntry extends SoftReference<Object> {
	    private final Object key;
	
	    SoftEntry(Object key, Object value, ReferenceQueue<Object> garbageCollectionQueue) {
	      super(value, garbageCollectionQueue);
	      this.key = key;
	    }
	  }


#### getObject(Object key) 方法


	  @Override
	  public Object getObject(Object key) {
	    Object result = null;
	    @SuppressWarnings("unchecked") // assumed delegate cache is totally managed by this cache
        //通过key获取缓存对象
	    SoftReference<Object> softReference = (SoftReference<Object>) delegate.getObject(key);
        //如果缓存对象为null，说明不存在key/value键值对
	    if (softReference != null) {
          //当获取缓存对象不为null时，键值对存在的情况下，获取软引用变量引用的对象
	      result = softReference.get();
	      if (result == null) {
            如果引用的对象为null，则说明已被JVM垃圾回收机制回收，从缓存中删除这个key
	        delegate.removeObject(key);
	      } else {
	        // See #586 (and #335) modifications need more than a read lock 
            如果引用对象不为null，说明该缓存对象未被回收，且这次访问到了，则把此对象引用加入强引用集合
	        synchronized (hardLinksToAvoidGarbageCollection) {
	          hardLinksToAvoidGarbageCollection.addFirst(result);
	          if (hardLinksToAvoidGarbageCollection.size() > numberOfHardLinks) {
                //当链表长度大于限制长度时，删除末尾结点
	            hardLinksToAvoidGarbageCollection.removeLast();
	          }
	        }
	      }
	    }
	    return result;
	  }

#### clear() 方法

	  @Override
	  public void clear() {
	    synchronized (hardLinksToAvoidGarbageCollection) {
	      hardLinksToAvoidGarbageCollection.clear();
	    }
	    removeGarbageCollectedItems();
	    delegate.clear();
	  }

使用 **synchronized** 关键字，对 **hardLinksToAvoidGarbageCollection** 加锁，防止多线程情况下对链表的破坏，让其对其他线程可见。

### WeakCache 弱引用实现的缓存管理

WeakCache 类弱引用实现缓存机制的原理和 SoftCache 软引用相似，只是使用的引用状态不同。把引用对象由 SoftReference 软引用换成了 WeakReference 弱引用。

在JVM的垃圾回收机制中，弱引用的对象一旦被垃圾收集器发现，则会被回收，无论内存是否足够。

#### 构造方法

	  public WeakCache(Cache delegate) {
	    this.delegate = delegate;
	    this.numberOfHardLinks = 256;
	    this.hardLinksToAvoidGarbageCollection = new LinkedList<Object>();
	    this.queueOfGarbageCollectedEntries = new ReferenceQueue<Object>();
	  }

同样是对象内部维护一个双链表 **LinkedList** 和一个引用队列 **ReferenceQueue** ，**ReferenceQueue** 限制了双链表的长度，默认为256。


#### WeakEntry 对象

	  private static class WeakEntry extends WeakReference<Object> {
	    private final Object key;
	
	    private WeakEntry(Object key, Object value, ReferenceQueue<Object> garbageCollectionQueue) {
	      super(value, garbageCollectionQueue);
	      this.key = key;
	    }
	  }

实现了静态内部类 WeakEntry，该类继承了 WeakReference 。


#### getObject(Object key) 方法


	  @Override
	  public Object getObject(Object key) {
	    Object result = null;
	    @SuppressWarnings("unchecked") // assumed delegate cache is totally managed by this cache
	    WeakReference<Object> weakReference = (WeakReference<Object>) delegate.getObject(key);
	    if (weakReference != null) {
	      result = weakReference.get();
	      if (result == null) {
	        delegate.removeObject(key);
	      } else {
	        hardLinksToAvoidGarbageCollection.addFirst(result);
	        if (hardLinksToAvoidGarbageCollection.size() > numberOfHardLinks) {
	          hardLinksToAvoidGarbageCollection.removeLast();
	        }
	      }
	    }
	    return result;
	  }

和 SoftCache 类的 getObject() 方法实现相类似，只是将 SoftReference 软引用换成了 WeakReference 弱引用。

## SoftCache 和 WeakCache 对比

在JVM垃圾回收机制下：

- WeakCache ：弱引用的对象一旦被垃圾收集器发现，则会被回收，无论内存是否足够。
- SoftCache ：软引用的对象，只要内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

弱引用和软引用缓存策略适用不同的场景，WeakCache 缓存的对象生存的周期较短，但不会给 JVM 性能带来太大的压力，适合不经常访问的对象使用。SoftCache 缓存对象生存周期相对较长，能生存至新生代的 eden 区快溢出的前一刻，适合较常访问的对象。