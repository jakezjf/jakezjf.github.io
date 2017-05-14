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



