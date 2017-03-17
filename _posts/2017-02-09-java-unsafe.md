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


##### allocateMemory()
allocateMemory 方法不调用构造方法生成对象：
	
	import sun.misc.Unsafe;
	
	import java.lang.reflect.Field;
	
	/**
	 * Created by zhongjianfeng.
	 */
	public class Demo {
	
	    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, InstantiationException {
	        Field field = User.class.getDeclaredField("theUnsafe");
	        field.setAccessible(true);
	        Unsafe unsafe = (Unsafe)field.get(null);
	        //通过unsafe来获取对象实例
	        User user = (User) unsafe.allocateInstance(User.class);
	        System.out.println("userName:" + user.getName() + "  age:" + user.getAge());
	        //调用构造方法
	        User user1 = new User();
	        System.out.println("userName:" + user1.getName() + "  age:" + user1.getAge());
	    }
	
	
	}
	
	
	class User{
	
	    private String name = "init";
	
	    private int age = 0;
	
	    public User(){
	        this.name = "jianfeng";
	        this.age = 21;
	    }
	
	    public String getName(){
	        return this.name;
	    }
	
	    public int getAge(){
	        return this.age;
	    }
	
	}
	
	
运行和程序会打印：
	
	userName:init  age:0
	userName:jianfeng  age:21
	
可以看到使用 allocateMemory 方法并没有调用 User 类的构造方法。

通过 allocateMemory 方法，我们可以不使用构造方法创建对象，这个方法是很有用的，假如我们想获取对象的属性，那么我们要先构造这个对象，这需要消耗堆里的内存空间，不是很划算吧。如果使用 allocateMemory 方法，我们可以不是创建这个实例获得对象的信息，方便开发，减少内存消耗。

##### reallocateMemory()
reallocateMemory() 方法用于扩充内存

	public native long reallocateMemory(long var1, long var3); 


##### freeMemory()

	public native void freeMemory(long var1);

释放内存,参数是内存地址。


##### copyMemory()

	public native void copyMemory(Object var1, long var2, Object var4, long var5, long var7);

copyMemory() 方法用来内存数据的拷贝。




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
	public native byte getByte(long address);
	
	//把给定内存地址上的值设置为byte值ｘ
	public native void putByte(long address, byte x);
	
	//获取给定内存地址上的值，该值是表示一个内存地址
	public native long getAddress(long address);
	
	//分配一块本地内存，这块内存的大小便是给定的大小．这块内存的值是没有被初始化的
	public native long allocateMemory(long bytes);
	
	
	//定义一个类
	public native Class<?> defineClass(String name, byte[] b, int off, int len,
	                                       ClassLoader loader,
	                                       ProtectionDomain protectionDomain);
	
	//定义一个匿名类
	public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
	
	//获取给定对象上的锁（jvm中有monitorenter 和 monitorexit两个指令）
	public native void monitorEnter(Object o);
	
	//释放给定对象上的锁（jvm中有monitorenter 和 monitorexit
	public native void monitorExit(Object o);
	
	//CAS操作，修改对象指定偏移量上的（CAS简介见末尾）
	public final native boolean compareAndSwapObject(Object o, long offset,
	                                                 Object expected,
	                                                 Object x);
	
	//CAS操作
	public final native boolean compareAndSwapInt(Object o, long offset,
	                                                  int expected,
	                                                  int x);
	
	//CAS操作
	public final native boolean compareAndSwapLong(Object o, long offset,
	                                                   long expected,
	                                                   long x);
	
	//获取被volatile关键字修饰的字段的值
	public native Object getObjectVolatile(Object o, long offset);
	
	//取消阻塞线程 thread,如果 thread 已经是非阻塞状态，
	//那么下次对该 thread 进行park操作时就不会阻塞
	public native void unpark(Object thread);
	
	//阻塞当前线程，isAbsolute的作用不是很清楚．（如果该方法调用前在线程非阻塞情况下调用了unpart方法，那么此次调用该方法不会令当前线程阻塞）
	public native void park(boolean isAbsolute, long time);
	
	//CAS操作
	public final int getAndAddInt(Object o, long offset, int delta) {
	        int v;
	        do {
	            v = getIntVolatile(o, offset);
	        } while (!compareAndSwapInt(o, offset, v, v + delta));
	        return v;
	}



























	
	