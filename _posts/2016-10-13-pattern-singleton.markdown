---
layout:     post
title:      "Singleton 单例模式"
subtitle:   "理解 Singleton 单例模式"
date:       2016-10-13
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - 设计模式
    - 单例模式
---

> 本篇将介绍Singleton 单例模式的相关知识


## 单例模式
顾名思义，单例模式就是一个类只能产生一个对象(一个实例)，这个类提供了访问唯一的对象的方式。

- 单例类只能有一个实例。
- 单例类必须自己创建自己的唯一实例。
- 单例类必须给所有其他对象提供这一实例。

单例模式在生活中使用广泛，例如线程池、对话框、缓存、注册表、日志对象、打印机驱动对象、等等，它们都有一个相同的特点，就是类对象只能产生一个实例。

## 懒汉式单例
可以从字面上理解，懒，它只在别人对它进行调用时才创建实例对象，不会主动进行创建。
	
	public class Singleton {
	    
	    private static Singleton singleton = null;
	    
	    private Singleton(){}
	    
	    public Singleton getSingleton(){
	        if (singleton == null){
	            singleton = new Singleton();
	        }
	        return singleton; 
	    }
	}

我们将Singleton的构造函数声明为private，使得构造函数不会被外部类直接调用。声明了一个公有的方法 getSingleton()，用来获取实例，首先判断实例对象是否为null，如果为null则进行创建，使得该类只产生单个实例。

上面所写的单例模式不是线程安全的，如果 getSingleton()方法被多个线程同时调用，可能会产生多个实例，所以我们要对代码进行改造。

#### 为 getSingleton() 方法添加synchronized关键字
synchronized 可以确保 getSingleton()在同一时间内只能被单个线程使用。

	
	public class Singleton {
	    
	    private static Singleton singleton = null;
	    
	    private Singleton(){}
	    
	    public synchronized Singleton getSingleton(){
	        if (singleton == null){
	            singleton = new Singleton();
	        }
	        return singleton; 
	    }
	}

上面的代码确实是确保了线程安全，但是却会产生巨大的性能消耗。每个线程获取 singleton 实例时，都要获取锁进入同步代码块，然后释放锁，产生了不必要的开销。怎么解决呢？可以使用双重检查锁定的方式。


#### 双重检查锁定
	public class Singleton {
	
	    private static Singleton singleton = null;
	
	    private Singleton(){}
	
	    public Singleton getSingleton(){
	        if (singleton == null){
	            synchronized (Singleton.class){
	                if (singleton == null){
	                    singleton = new Singleton();
	                }
	            }
	        }
	        return singleton;
	    }
	}

首先判断对象是否为null，如果不为null，直接返回实例，不需要进入同步代码块。如果为null，就进入同步代码块。好处就是对于进入同步代码块的开销只存在首次创建实例时。

#### 静态内部类

	public class Singleton {
	
	    private Singleton(){
	    }
	
	    private static class Sing{
	        private static final Singleton singleton = new Singleton();
	    }
	
	    public static final Singleton getSingleton(){
	        return Sing.singleton;
	    }
	
	}

静态内部类的方式要比上面两种方式要好，确保了线程安全，也避免了同步带来的性能问题。

### 饿汉式单例
饿汉模式在类初始化时，已经自行实例化。

	public class Singleton {
	
	    private static final Singleton singleton = new Singleton();
	
	    private Singleton(){}
	
	    public static Singleton getSingleton(){
	        return singleton;
	    }
	
	}


## 枚举方式

它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。

	public enum Singleton {  

	    INSTANCE;  

	    public void whateverMethod() {  
	    }  

	} 


## 总结
线程安全：饿汉式单例是天生线程安全的，可以在多线程环境下使用。
懒汉式单例要考虑线程安全的问题，使用静态内部类的方式比较好。

饿汉式在类创建的同时就实例化一个静态对象出来，不管之后会不会使用这个单例，都会占据一定的内存，但是相应的，在第一次调用时速度也会更快，因为其资源已经初始化完成。

懒汉式会延迟加载，只有在第一次使用该单例时才会产生实例对象。