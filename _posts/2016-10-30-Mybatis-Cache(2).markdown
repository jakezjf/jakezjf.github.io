---
layout:     post
title:      "Mybatis Cache 设计及实现(三)"
subtitle:   "Mybatis Cache 源码解析"
date:       2016-10-30
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - Mybatis
---

> 本篇将介绍 Mybatis Cache 的相关知识


## Mybatis

### SerializedCache 源码解读

SerializedCache 类用于缓存序列化和反序列化存储

#### putObject(Object key, Object object) 方法

	  @Override
	  public void putObject(Object key, Object object) {
		//检查value是否为空、是否为可序列化的对象
	    if (object == null || object instanceof Serializable) {
          //通过serialize()方法把，将value序列化为字节数组
	      delegate.putObject(key, serialize((Serializable) object));
	    } else {
          //如果不是可序列话的对象，那么会
	      throw new CacheException("SharedCache failed to make a copy of a non-serializable object: " + object);
	    }
	  }

所有要缓存的 value 必须实现 **Serializable** 接口，否则会抛出 **CacheException **异常。

#### serialize(Serializable value) 方法

	  private byte[] serialize(Serializable value) {
	    try {
	      ByteArrayOutputStream bos = new ByteArrayOutputStream();
	      ObjectOutputStream oos = new ObjectOutputStream(bos);
	      oos.writeObject(value);
	      oos.flush();
	      oos.close();
	      return bos.toByteArray();
	    } catch (Exception e) {
	      throw new CacheException("Error serializing object.  Cause: " + e, e);
	    }
	  }

ByteArrayOutputStream 对象，将 value 转换成字节数组，使用 ObjectOutputStream 对象 writeObject 方法写入对象，方法最后返回字节数组。

#### deserialize(byte[] value) 方法

	  private Serializable deserialize(byte[] value) {
	    Serializable result;
	    try {
	      ByteArrayInputStream bis = new ByteArrayInputStream(value);
	      ObjectInputStream ois = new CustomObjectInputStream(bis);
	      result = (Serializable) ois.readObject();
	      ois.close();
	    } catch (Exception e) {
	      throw new CacheException("Error deserializing object.  Cause: " + e, e);
	    }
	    return result;
	  }

CustomObjectInputStream 类对 value 反序列化。

#### CustomObjectInputStream 内部类

	  public static class CustomObjectInputStream extends ObjectInputStream {
	
	    public CustomObjectInputStream(InputStream in) throws IOException {
	      super(in);
	    }
	
	    @Override
	    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
	      return Resources.classForName(desc.getName());
	    }
	  }

在进行反序列化时，SerializedCache 类使用了内部定义的类 **CustomObjectInputStream** ，该类继承自ObjectInputStream，重写了父类的 **resolveClass** 方法，区别在于解析加载Class对象时使用的ClassLoader类加载器不同。


### TransactionalCache 源码解析

TransactionalCache 第二级缓存事务缓冲区(事务性缓存)，该类相比于其他类增加了两个方法：**commit()** 方法、**rollback()** 方法。TransactionalCache对象内部存在暂存区，所有对缓存对象的写操作都不会直接作用于缓存对象，而是被保存在暂存区。当执行 **commit()**方法时，才将缓存对象写入缓存。如果会话被回滚，那么之前没写入的暂存区对象将会被丢弃。

TransactionalCache 类内部属性：

	  private Cache delegate;
      //初始化为false，用于在commit()方法中进行判断是否清空缓存，
	  private boolean clearOnCommit;
      //在执行commit，需要进行添加操作时，putObject(Object key, Object object) 将对象加入 entriesToAddOnCommit
	  private Map<Object, Object> entriesToAddOnCommit;
      //用于存储需要删除的元素，在执行 rollback() 方法时，调用 unlockMissedEntries() 方法
	  private Set<Object> entriesMissedInCache;

     
#### reset() 方法

	private void reset() {
	    clearOnCommit = false;
	    entriesToAddOnCommit.clear();
	    entriesMissedInCache.clear();
	}

clearOnCommit 设置为false，下次执行 **commit()** 方法时不需要清空缓存，直接调用 **HashSet**、**HashMap** 的 **clear()** 方法，达到重置效果。

#### clear() 方法

	  @Override
	  public void clear() {
        //clearOnCommit 设置为 true ，下次 commit 时会清空缓存
	    clearOnCommit = true;
	    entriesToAddOnCommit.clear();
	  }

clear() 用于清空暂存区的未加入缓存的对象，当下次 **commit** 时才会清空缓存。

#### commit() 方法

	  public void commit() {
        //判断是否需要清空缓存
	    if (clearOnCommit) {
	      delegate.clear();
	    }
        //对需要删除的元素进行删除
	    flushPendingEntries();
        //重置暂存区
	    reset();
	  }

执行 **commit()** 方法首先判断是否需要清空缓存，**clearOnCommit** 在使用 **clear()** 方法时，会将标志 **clearOnCommit** 设置为 **true** ，

#### flushPendingEntries() 方法

	  private void flushPendingEntries() {
	    for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
	      delegate.putObject(entry.getKey(), entry.getValue());
	    }
	    for (Object entry : entriesMissedInCache) {
	      if (!entriesToAddOnCommit.containsKey(entry)) {
	        delegate.putObject(entry, null);
	      }
	    }
	  }

flushPendingEntries() 方法会被 commit() 方法调用，用于将暂存区 **entriesToAddOnCommit** 的对象加入缓存中，同时遍历暂存区 **entriesMissedInCache** ，将此暂存区的元素对应缓存的key，将 value 设为null。


#### unlockMissedEntries() 方法

	  private void unlockMissedEntries() {
	    for (Object entry : entriesMissedInCache) {
	      try {
	        delegate.removeObject(entry);
	      } catch (Exception e) {
	        log.warn("Unexpected exception while notifiying a rollback to the cache adapter."
	            + "Consider upgrading your cache adapter to the latest version.  Cause: " + e);
	      }
	    }
	  }

从缓存中移除key为 **entriesMissedInCache** 的对象。 