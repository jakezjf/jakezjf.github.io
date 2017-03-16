---
layout:     post
title:      "学习Java Unsafe 类的使用"
subtitle:   "学习Java Unsafe 类的使用"
date:       2017-02-09
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> 学习Java Unsafe 类的使用

# Unsafe 类

在学习 Java concurrent 包中的原子类 Atomic 时，发现 Atomic 原子性都是基于 Unsafe 类实现的，Unsafe 类到底是什么？有什么作用呢？

sun.misc.Unsafe 在 Java 1.4 版本就出现了，这个类用于执行低级别、不安全操作的方法集合。大家都知道 Java 是一个注重安全的语言，不允许程序员进行底层的操作，Unsafe 为我们提供了一些对底层进行操作的方法，为开发者提供了很大的便利。


##### Unsafe 构造器
Unsafe 类内部的方法大多都是 public ，但 Unsafe 构造器是私有的，不能直接被调用，需要调用 getUnsafe() 方法。

    private Unsafe() {
    
    }
    
并且使用了饿汉式单例。

	private static final Unsafe theUnsafe;
    
    
##### getUnsafe() 
getUnsafe() 方法用来获取 Unsafe 实例，但一般情况下拿不到这个实例，只能由 JDK 库里的类使用。

    public static Unsafe getUnsafe() {
    	//获取调用该方法的调用者信息
        Class var0 = Reflection.getCallerClass();
        if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }
    
VM.isSystemDomainLoader() 方法用来判断该类是不是由系统类加载器加载的，如果该类不是系统类加载器加载的，那么将会抛出 SecurityException 异常。
 

## Unsafe 主要方法

- allocateMemory		用于分配内存
- reallocateMemory		用于扩充内存
- freeMemory			用于释放内存




# Unsafe 其余方法

	
	//返回一个静态字段的偏移量
	public native long staticFieldOffset(Field f);
	
	//返回一个非静态字段字段的偏移量
	public native long objectFieldOffset(Field f);
	
	//获取给定对象指定偏移量的int值
	public native int getInt(Object o, long offset);
	
	//把给定对象指定偏移量上的值设置为整型变量ｘ
	public native void putInt(Object o, long offset, int x);
	
	//获取给定内存地址上的byte值
	public native byte    getByte(long address);
	
	//把给定内存地址上的值设置为byte值ｘ
	public native void    putByte(long address, byte x);
	
	//获取给定内存地址上的值，该值是表示一个内存地址
	public native long getAddress(long address);
	
	//分配一块本地内存，这块内存的大小便是给定的大小．这块内存的值是没有被初始化的
	public native long allocateMemory(long bytes);



























	
	