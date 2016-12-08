

## ThreadLocal 
ThreadLocal从字面上理解，是线程局部变量的意思。查阅许多资料博客，对ThreadLocal的解释是解决多线程程序的并发问题，可以用来解决多线程访问资源时的共享问题，我觉得不是太准确。

JDK文档是这样介绍的：

	This class provides thread-local variables. 
	These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, 
	independently initialized copy of the variable. ThreadLocal instances are typically private
	static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).

大致意思是：ThreadLocal 这个类可以提供线程内的局部变量，这种变量在多线程环境下访问(通过get或set方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量，ThreadLocal 实例通常是**private static**类型，用于关联线程和线程的上下文。

通常在并发编程中，我们会经常使用 **volatile** 修饰成员变量，确保被 volatile 修饰的成员变量的可见性。而 **ThreadLocal** 这个类主要是确保每个线程拥有独立的局部变量，这个局部变量只在该线程内使用有效，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。 

举个栗子：

举行一次跳绳比赛，有十位学生进行比赛，简单写个为学生技术跳绳个数的方法：

	int sum = 0;
	for(int i = 0;i<100;i++){
		sum++;//计数
	}

假如我们有十位老师，那么每位同学配备一位老师进行计数，这十位同学就能够同时进行比赛了。程序中的sum就相当于每位老师，ThreadLocal 这个类做的就是要为每个线程拷贝一份变量，在线程的生命周期中独自使用这个变量，其它线程对这个线程是不可见的。ThreadLocal设计的初衷是：提供线程内部的局部变量，在本线程内随时随地可取，隔离其他线程。

## ThreadLocal 基本操作


### ThreadLocal 构造函数

	public ThreadLocal() {
    }

构造函数啥也没做

### 初始化方法

	protected T initialValue() {
        return null;
    }

初始化 value 为 null，如果没有事先调用 set() 方法，直接调用 get() 方法，则该函数会被调用。

### createMap(Thread t, T firstValue) 方法

	void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

### remove() 方法

	public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }

通过 getMap() 方法获取线程的ThreadLocalMap，如果不为空，则调用ThreadLocalMap的remove()方法进行删除。


### setInitialValue()方法

	private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

setInitialValue() 方法在 ThreadLocalMap 为空的情况下，会被调用。如果 ThreadLocalMap 为空，会调用 createMap (Thread t, T firstValue) 方法调用，实例化 ThreadLocalMap。

### set()方法

	public void set(T value) {
		Thread t = Thread.currentThread();
		ThreadLocalMap map = getMap(t);
		if (map != null)
			map.set(this, value);
		else
			createMap(t, value);
	}

Thread.currentThread() 方法获取当前线程。

### getMap(Thread t) 方法

	ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

通过线程参数，获取线程的 ThreadLocalMap。

### get()方法

	public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

get()方法通过 getMap() 方法获取线程的 ThreadLocalMap，每个Thread对象内部都维护了一个ThreadLocalMap这样一个ThreadLocal的Map，可以存放若干个ThreadLocal。

看看 Thread 中的代码

	/* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;


从本质上来讲，每个线程都维护了一个 Entry 数组，Entry 中的 key 就是ThreadLocal，value就是我们通过set方法传入的值，当我们想要获取这个值得时候，可以通过 get方法获取。
