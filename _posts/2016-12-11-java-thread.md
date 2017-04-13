---
layout:     post
title:      "Java 并发编程的思考"
subtitle:   "Java 并发编程的思考"
date:       2017-02-10
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> Java 并发编程的思考

# Java 并发编程思考

###背景 
现代硬件发展快速，多核系统已成为主流，CPU 制造厂商发现提高时钟频率的成本越来越大，在提高时钟频率的研发越来越没有意义，所以都转向了对多核的研究，著名的 CPU 制造厂商英特尔宣布进入多核时代。Java 是一个天生支持多线程的语言，对于多线程的支持，使得 Java 在对于性能要求十分苛刻的场景备受青睐。然而编写多线程代码并非那么简单，不仅需要我们对 Java 并发库有足够的了解，而且对操作系统底层及汇编语言层也要有所了解，Java 编译器会将 Java源代码编译成字节码文件，字节码通过 Java 虚拟机运行，形成汇编指令。 


### 常见的场景  
在生活中有许多场景与多线程相类似，例如：我们要烧一壶水，我们不必一直等待直到水烧开，我们可以一边看书一边等水烧开，节约我们的时间。Java 的 GUI 中，为了提高图形界面的流畅性，我们通常会使用多线程，比如我们在点击鼠标的同时也能够进行键盘按键的触发。在 JVM 垃圾回收处理机制中，并发标记扫描垃圾回收器（CMS Garbage Collector）就使用了多线程，提高垃圾回收效率，减少 gc 时间。 

### Java 实例

##### 线程不安全

线程不安全的实例：

    private static int value;
    
    public static void add(){
        value++;
    }

看看下面的流程图，当两个线程同时执行这段代码时，将会发生意想不到的事：

[](thread1.png)

value++ 其实包含三个步骤，首先要先获取value的初始值，然后对初始值进行+1操作，最后将上一步操作写入value。

##### 线程安全

线程安全实例：

使用 synchronized 给对象加锁，当一个线程执行这个方法时，该线程会先去获取这个锁，获得锁后，其他线程将无法对这个对象进行操作，这时候单位时间内只有一个线程对该对象进行操作，所以是线程安全的。但给对象加锁会带来较大的开销。

    private int value;

    public synchronized void add(){
        value++;
    }

使用 synchronized 关键字给对象加锁，还有一种方式是使用显示加锁。

    private int value;
    
    private Object lock = new Object();

    public synchronized void add(){
        synchronized (lock){
            value++;
        }
    }
    
这里我们使用了一个 object 对象，我们通过这个 lock 对象，实现代码块的同步，也是一种同步方式。

上面两种都有加锁的开销，Java 中提供了一个关键字：volatile


    private volatile int value;

    public synchronized void add(){
        value++;
    }
    
volatile 可以保证变量的数据一致性，在汇编层面，Java 编译器会为被 volatile 关键字修饰的变量前后加上标记，在转化为汇编指令时，通过 store、load 实现数据一致性。


统计方法的调用次数是比较常见的需求

    private int count = 0;

    public void add(){
        count++;
        //doing
    }
    
上面这段代码的 count 可以用来统计 add() 方法被调用的次数。

但这并不是线程安全的，Java 中提供了一个原子变量类，在 java.util.concurrent.atomic 中包含许多原子变量类，例如：AtomicLong

    private AtomicLong count = new AtomicLong(0);
    
    public long getCount(){
        return count.get();
    }
    
    public void add(){
        count.incrementAndGet();
        //doing
    }
    
通过原子变量类 AtomicLong 代替使用 Long 类型就可以实现线程安全了，十分方便，它能确保我们访问变量都是原子性操作。











