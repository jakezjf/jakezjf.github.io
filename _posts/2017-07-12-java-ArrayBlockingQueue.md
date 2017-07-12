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




