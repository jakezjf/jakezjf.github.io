---
layout:     post
title:      "Java 并发之 ArrayBlockingQueue"
subtitle:   "ArrayBlockingQueue 的实现"
date:       2017-07-12
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> Java 并发之 ArrayBlockingQueue

# ArrayBlockingQueue
 
 从这个工具类的名字可以看出，这是一个基于数组实现的阻塞队列。ArrayBlockingQueue内部通过维护一个数组items来存放队列元素，通过 takeIndex 来确定出队下标，putIndex 来确定入队下标，count确定这个数组的长度，也就是队列的大小。ArrayBlockingQueue 中的成员变量都没有用 volatile 修饰，这是因为访问这些变量使用都是在锁块内，不存在可见性问题。通过独占锁lock用来对出入队操作加锁，保证内存的可见性，同一时刻只有一个线程可以访问入队出队。notEmpty，notFull条件变量用来进行出入队的同步。
 
#### 成员变量
 
- final Object[] items;

	通过数组维护整个队列

- int takeIndex;

	出队下标

- int putIndex;

	入队下标

- int count;

	统计当前数组长度

- final ReentrantLock lock;

	通过 ReentrantLock 保证内存可见性

- private final Condition notEmpty;

- private final Condition notFull;

#### 构造函数


    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
    
ArrayBlockingQueue 构造函数可以传两个参数，capacity 设置数组长度，fair 设置是否为公平的锁策略。

    public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
        this(capacity, fair);

        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
    
 也可以在初始化 ArrayBlockingQueue 时添加集合元素。

##### size()方法

    public int size() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
    
先获取这个对象的锁，可以保证数据一致性，在高并发场景，不会出现脏数据。

##### put()方法

    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    
  put()方法可以插入一个元素，采用了中断锁，当 cout == items.length 时，一直阻塞，直到队列有空的位置。
  
  ##### poll()方法
  
     public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
    
   如果队列为空返回 null
   
   ##### peek()方法
   
       public E peek() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return itemAt(takeIndex); // null when queue is empty
        } finally {
            lock.unlock();
        }
    }
   
   取队头元素
   
   ##### remove()方法
   
   
       public boolean remove(Object o) {
        if (o == null) return false;
        final Object[] items = this.items;
        //获取锁
        final ReentrantLock lock = this.lock;
        //加锁
        lock.lock();
        try {
            if (count > 0) {
                final int putIndex = this.putIndex;
                int i = takeIndex;
                do {
                		//遍历数组获取元素
                    if (o.equals(items[i])) {
                        removeAt(i);
                        return true;
                    }
                    if (++i == items.length)
                        i = 0;
                } while (i != putIndex);
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
    
   删除元素的方法，时间复杂度为O(n)，可以看出删除元素是十分耗时的，并且还阻塞了队列。
   
  
  # 总结
  从上面可以看出，这是一个典型的生产消费模式。 ArrayBlockingQueue 和 LinkedBlockingQueue 的区别是，ArrayBlockingQueue 一开始就确定了容器的大小，适合对内存、队列大小有严格要求的场景，LinkedBlockingQueue 是以链表为基础实现的方式，可以不限制队列长度。








