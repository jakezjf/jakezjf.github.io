---
layout:     post
title:      "java 并发编程中会遇到的问题"
subtitle:   "介绍并发编程相关知识"
date:       2016-09-08
author:     "JianFeng"
header-img: "img/blog/"
catalog: true
tags:
    - java
    - 多线程
---

> 本篇将介绍Java 并发编程中会遇到的问题

## 上下文
在执行多线程程序时，CPU通过为每个线程分配CUP时间片来实现并发。时间片是CPU分配个各个线程的时间，一般为几十毫秒。CPU通过快速的在线程间切换，让我们感觉多个线程是同时执行的。

CPU采用时间片分配算法，在线程间来回切换，当从一个线程切换到另一个线程时，将会保存当前线程的任务状态，以便下次切换回该线程时，能够继续执行该线程的任务。**上下文的切换**就是任务从保存到再加载的过程。

## 死锁
死锁是两个甚至多个线程被永久阻塞时的一种运行局面，这种局面的生成伴随着至少两个线程和两个或者多个资源。看下面一个实例，线程1持有A资源，请求B资源；线程2持有B资源，请求A资源。


	public class Main {
	
	    public static String A = "A";
	
	    public static String B = "B";
	
	    public static void main(String[] args){
	        new Main().thread();
	    }
	
	    private void thread(){
	        Thread thread1 = new Thread(new Runnable() {
	            @Override
	            public void run() {
	                synchronized (A){  //持有A资源
	                    try {
	                        Thread.currentThread().sleep(4000);
	                    } catch (InterruptedException e) {
	                        e.printStackTrace();
	                    }
	                    synchronized (B){  //请求B资源
	                        System.out.println(B);
	                    }
	                }
	            }
	        });
	        Thread thread2 = new Thread(new Runnable() {
	            @Override
	            public void run() {
	                synchronized (B){  //持有B资源
	                    try {
	                        Thread.currentThread().sleep(4000);
	                    } catch (InterruptedException e) {
	                        e.printStackTrace();
	                    }
	                    synchronized (A){  //请求A资源
	                        System.out.println(A);
	                    }
	                }
	            }
	        });
	        thread1.start();
	        thread2.start();
	    }
	
	}

使用jstack 查看程序的栈信息

	Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):
	
	"DestroyJavaVM" prio=6 tid=0x000000000138e800 nid=0x564 waiting on condition [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"Thread-1" prio=6 tid=0x000000000cee6800 nid=0x1af0 waiting for monitor entry [0x000000000d5cf000]
	   java.lang.Thread.State: BLOCKED (on object monitor)
	        at com.jf.Test03.Main$2.run(Main.java:42)
	        - waiting to lock <0x00000007d5d7d110> (a java.lang.String)
	        - locked <0x00000007d5d7d140> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	
	   Locked ownable synchronizers:
	        - None
	
	"Thread-0" prio=6 tid=0x000000000cee4000 nid=0x1974 waiting for monitor entry [0x000000000d4cf000]
	   java.lang.Thread.State: BLOCKED (on object monitor)
	        at com.jf.Test03.Main$1.run(Main.java:27)
	        - waiting to lock <0x00000007d5d7d140> (a java.lang.String)
	        - locked <0x00000007d5d7d110> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	
	   Locked ownable synchronizers:
	        - None
	
	"Monitor Ctrl-Break" daemon prio=6 tid=0x000000000cef2800 nid=0x2f44 runnable [0x000000000d3ce000]
	   java.lang.Thread.State: RUNNABLE
	        at java.net.DualStackPlainSocketImpl.accept0(Native Method)
	        at java.net.DualStackPlainSocketImpl.socketAccept(DualStackPlainSocketImpl.java:131)
	        at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:398)
	        at java.net.PlainSocketImpl.accept(PlainSocketImpl.java:199)
	        - locked <0x00000007d5df4b48> (a java.net.SocksSocketImpl)
	        at java.net.ServerSocket.implAccept(ServerSocket.java:530)
	        at java.net.ServerSocket.accept(ServerSocket.java:498)
	        at com.intellij.rt.execution.application.AppMain$1.run(AppMain.java:90)
	        at java.lang.Thread.run(Thread.java:745)
	
	   Locked ownable synchronizers:
	        - None
	
	"Service Thread" daemon prio=6 tid=0x000000000b50d800 nid=0x2d30 runnable [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"C2 CompilerThread1" daemon prio=10 tid=0x000000000b503000 nid=0x2c0c waiting on condition [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"C2 CompilerThread0" daemon prio=10 tid=0x000000000b501800 nid=0x2c1c waiting on condition [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"Attach Listener" daemon prio=10 tid=0x000000000b500800 nid=0x1c1c waiting on condition [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"Signal Dispatcher" daemon prio=10 tid=0x000000000b4fe000 nid=0x2224 runnable [0x0000000000000000]
	   java.lang.Thread.State: RUNNABLE
	
	   Locked ownable synchronizers:
	        - None
	
	"Finalizer" daemon prio=8 tid=0x000000000b4a9800 nid=0x11d0 in Object.wait() [0x000000000c85e000]
	   java.lang.Thread.State: WAITING (on object monitor)
	        at java.lang.Object.wait(Native Method)
	        - waiting on <0x00000007d5c04858> (a java.lang.ref.ReferenceQueue$Lock)
	        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	        - locked <0x00000007d5c04858> (a java.lang.ref.ReferenceQueue$Lock)
	        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)
	
	   Locked ownable synchronizers:
	        - None
	
	"Reference Handler" daemon prio=10 tid=0x000000000b4a6000 nid=0xfd4 in Object.wait() [0x000000000c75f000]
	   java.lang.Thread.State: WAITING (on object monitor)
	        at java.lang.Object.wait(Native Method)
	        - waiting on <0x00000007d5c04470> (a java.lang.ref.Reference$Lock)
	        at java.lang.Object.wait(Object.java:503)
	        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	        - locked <0x00000007d5c04470> (a java.lang.ref.Reference$Lock)
	
	   Locked ownable synchronizers:
	        - None
	
	"VM Thread" prio=10 tid=0x000000000b4a2000 nid=0x1d3c runnable
	
	"GC task thread#0 (ParallelGC)" prio=6 tid=0x0000000003077800 nid=0x860 runnable
	
	"GC task thread#1 (ParallelGC)" prio=6 tid=0x0000000003079800 nid=0x2810 runnable
	
	"GC task thread#2 (ParallelGC)" prio=6 tid=0x000000000307b000 nid=0xfa8 runnable
	
	"GC task thread#3 (ParallelGC)" prio=6 tid=0x000000000307d800 nid=0x2ae4 runnable
	
	"VM Periodic Task Thread" prio=10 tid=0x000000000b527800 nid=0x2c30 waiting on condition
	
	JNI global references: 144
	
	
	Found one Java-level deadlock:
	=============================
	"Thread-1":
	  waiting to lock monitor 0x000000000b4b00a8 (object 0x00000007d5d7d110, a java.lang.String),
	  which is held by "Thread-0"
	"Thread-0":
	  waiting to lock monitor 0x000000000b4aecb8 (object 0x00000007d5d7d140, a java.lang.String),
	  which is held by "Thread-1"
	
	Java stack information for the threads listed above:
	===================================================
	"Thread-1":
	        at com.jf.Test03.Main$2.run(Main.java:42)
	        - waiting to lock <0x00000007d5d7d110> (a java.lang.String)
	        - locked <0x00000007d5d7d140> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	"Thread-0":
	        at com.jf.Test03.Main$1.run(Main.java:27)
	        - waiting to lock <0x00000007d5d7d140> (a java.lang.String)
	        - locked <0x00000007d5d7d110> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	
	Found 1 deadlock.

在线程Thread-1 和Thread-0 出现了java.lang.Thread.State: BLOCKED 状态。



	"Thread-1" prio=6 tid=0x000000000cee6800 nid=0x1af0 waiting for monitor entry [0x000000000d5cf000]
	   java.lang.Thread.State: BLOCKED (on object monitor)
	        at com.jf.Test03.Main$2.run(Main.java:42)
	        - waiting to lock <0x00000007d5d7d110> (a java.lang.String)
	        - locked <0x00000007d5d7d140> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	
	   Locked ownable synchronizers:
	        - None
	
	"Thread-0" prio=6 tid=0x000000000cee4000 nid=0x1974 waiting for monitor entry [0x000000000d4cf000]
	   java.lang.Thread.State: BLOCKED (on object monitor)
	        at com.jf.Test03.Main$1.run(Main.java:27)
	        - waiting to lock <0x00000007d5d7d140> (a java.lang.String)
	        - locked <0x00000007d5d7d110> (a java.lang.String)
	        at java.lang.Thread.run(Thread.java:745)
	
	   Locked ownable synchronizers:
	        - None

可以看出是Main类的42行和27行引起了死锁。

## 资源限制

在并发编程中计算机硬件资源和软件资源对程序运行的影响很大，比如对一个文件进行下载，下载速度是500Kb/s，即使启动多个线程，也不能提高下载速度。有时候资源的限制也会导致并发执行的效率，比如使用单个线程对执行写内存，已经达到IO最大写速率，这时使用多个线程执行并不能提高执行效率，反而会使程序变慢，因为增加了上下文切换的时间和资源调度的时间。







