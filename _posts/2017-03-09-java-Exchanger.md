---
layout:     post
title:      "Java 并发编程之 Exchanger "
subtitle:   "Java Exchanger 的使用"
date:       2017-03-09
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

>Java 并发编程之 Exchanger 



# Exchanger

Exchanger 是 Java concurrent 包中的并发工具类，可以在两个线程之间交换数据，值得注意的是它只支持在两个线程之间进行数据交互，听起来是不是很像生产者/消费者模式，一个线程生产数据，另一个线程消费数据。它不支持更多的线程交换数据。



# 实例



##### ThreadA 类

```
public class ThreadA extends Thread{

    private Exchanger<String> exchanger;

    public ThreadA(Exchanger<String> exchanger){
        super();
        this.exchanger = exchanger;
    }

    @Override
    public void run(){
        try {
            System.out.println("线程A中得到B的消息" + exchanger.exchange("线程A"));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



线程A 向另一个线程(线程B)发送消息 “线程A”

##### ThreadB 类

```
public class ThreadB extends Thread{

    private Exchanger<String> exchanger;

    public ThreadB(Exchanger<String> exchanger){
        super();
        this.exchanger = exchanger;
    }

    @Override
    public void run(){
        try {
            System.out.println("线程B中得到A的消息" + exchanger.exchange("线程B"));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

线程B 向另一个线程(线程A)发送消息 “线程B”

##### 

##### Main 类

```
public static void main(String[] args){
    Exchanger<String> exchanger = new Exchanger<>();
    ThreadA threadA = new ThreadA(exchanger);
    ThreadB threadB = new ThreadB(exchanger);
    threadA.start();
    threadB.start();

}
```

打印：

```
线程B中得到A的消息线程A
线程A中得到B的消息线程B
```

可以看到 线程A 发送的消息传递给了 线程B ；线程B 发送的消息传递给了 线程A 。

# 源码分析

研究了一下 Exchanger 的部分源码，了解了它的一些实现原理。

Exchanger 的成员变量：

```
private volatile Node slot;

private final Participant participant;

private volatile int bound;

private volatile Node[] arena;



```



##### Participant 内部类

```
static final class Participant extends ThreadLocal<Node> {
    public Node initialValue() { return new Node(); }
}
```

可以看出该类继承了 ThreadLocal ，具有 ThreadLocal 的功能，ThreadLocal 内部维护了一个 ThreadMap ，将 ThreadLocal 对象作为 key ，value 作为值进行存储，解决在多个线程同时需要使用一个初始值而该值不受其他线程影响的场景。

Participant 储存了每个线程的的值，就是上面实例中调用 exchanger.exchange() 方法进行传递的值。



##### slotExchange() 方法

```
private final Object slotExchange(Object item, boolean timed, long ns) {
    //获取participant中的值
    Node p = participant.get();
    Thread t = Thread.currentThread();
    if (t.isInterrupted()) // preserve interrupt status so caller can recheck
        return null;

	//阻塞
    for (Node q;;) {
        if ((q = slot) != null) {
        	//判断 slot、p 是否为空，也可以说判断是否有其他线程写入slot
            if (U.compareAndSwapObject(this, SLOT, q, null)) {
                Object v = q.item;
                q.match = item;
                Thread w = q.parked;
                if (w != null)
                    U.unpark(w);
                return v;
            }
            // create arena on contention, but continue until slot null
            if (NCPU > 1 && bound == 0 &&
                U.compareAndSwapInt(this, BOUND, 0, SEQ))
                arena = new Node[(FULL + 2) << ASHIFT];
        }
        else if (arena != null)
            return null; // caller must reroute to arenaExchange
        else {
        	//运行到这，说明另一个交互的线程没有写入数据
            p.item = item;
            //将p写入 SLOT 用的是 CAS 算法
            if (U.compareAndSwapObject(this, SLOT, null, p))
            	//如果修改成功，跳出无限循环。
                break;
            p.item = null;
        }
    }

    int h = p.hash;
    long end = timed ? System.nanoTime() + ns : 0L;
    int spins = (NCPU > 1) ? SPINS : 1;
    Object v;
    while ((v = p.match) == null) {
        if (spins > 0) {
            h ^= h << 1; h ^= h >>> 3; h ^= h << 10;
            if (h == 0)
                h = SPINS | (int)t.getId();
            else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                Thread.yield();
        }
        else if (slot != p)
            spins = SPINS;
        else if (!t.isInterrupted() && arena == null &&
                 (!timed || (ns = end - System.nanoTime()) > 0L)) {
            U.putObject(t, BLOCKER, this);
            p.parked = t;
            if (slot == p)
                U.park(false, ns);
            p.parked = null;
            U.putObject(t, BLOCKER, null);
        }
        else if (U.compareAndSwapObject(this, SLOT, p, null)) {
            v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
            break;
        }
    }
    U.putOrderedObject(p, MATCH, null);
    p.item = null;
    p.hash = h;
    return v;
}
```

可以看出 Exchanger 类在两个线程交互时是的阻塞，也就是单工模式。