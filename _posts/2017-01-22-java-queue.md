---
layout:     post
title:      "BlockingQueue 中 ArrayBlockingQueue 和 LinkedBlockingQueue 的实现"
subtitle:   "Java 并发的学习"
date:       2017-01-22
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> 学习 BlockingQueue 的设计


# BlockingQueue 中 ArrayBlockingQueue 和 LinkedBlockingQueue 的实现


# BlockingQueue
**BlockingQueue** 从字面上看我们知道这是一个阻塞队列的接口， BlockingQueue 接口继承了 Queue 接口，具有队列 FIFO (先进先出) 的特性， 在对 BlockingQueue 进行 insert(插入)，如果队列满了（当队列到达缓冲区最大值时），会阻塞操作，直到队列出队操作释放队列元素，才会唤醒插入线程操作。如果队列为空且有线程进行出队操作时，也会阻塞操作，直到队列有新的元素入队，才唤醒出队线程。


# BlockingQueue 接口

	public interface BlockingQueue<E> extends Queue<E> {
	
		//向队列加入元素
	    boolean add(E e);
	
		//将指定元素插入队列中
	    boolean offer(E e);
	
	   	//入队操作
	    void put(E e) throws InterruptedException;
	
		//将指定元素插入队列中，并设置超时时间
	    boolean offer(E e, long timeout, TimeUnit unit)
	        throws InterruptedException;
	
		//删除队列头
	    E take() throws InterruptedException;
	
		//使该队列进行出队操作，设置超时时间，时间单位
	    E poll(long timeout, TimeUnit unit)
	        throws InterruptedException;
	
		//返回可入队的元素数量，最大为 Integer.MAX_VALUE 
	    int remainingCapacity();
	
		//从该队列中删除元素
	    boolean remove(Object o);
	
		//判断队列是否有该元素，返回true、false
	    public boolean contains(Object o);
	
		//从该队列中删除所有元素，并将它们添加到给定的集合中
	    int drainTo(Collection<? super E> c);
	
		//设置删除元素数量并添加到指定集合
	    int drainTo(Collection<? super E> c, int maxElements);
	}

# BlockingQueue 主要的实现

BlockingQueue 接口的实现类主要有两个：

- ArrayBlockingQueue
- LinkedBlockingQueue

ArrayBlockingQueue 类是基于数组实现的，在进行初始化时要设置队列容量，并且无法修改。

LinkedBlockingQueue 类是基于链表实现的，在进行初始化时可不设置队列容量，最大限制容量为 Integer.MAX_VALUE 。


# ArrayBlockingQueue
ArrayBlockingQueue 实现的是一个有界的缓冲区队列，初始化时需要制定有界缓冲区的容量，并且无法更改。当队列已满，进行入队操作会导致阻塞，直到有消费者消费队列。当队列为空时，进行出队操作也会导致阻塞，直到有生产者进行入队。适合于实现“生产者消费者”模式。


##### ArrayBlockingQueue 属性


	//序列化相关
	private static final long serialVersionUID = -817911632652898426L;

    //数组存储元素
    final Object[] items;

    //删除元素、获取下一个元素索引
    int takeIndex;

    //入队操作，加入元素操作索引
    int putIndex;

    //用于计数
    int count;

    //队列用的锁，读写操作都要获取容器的锁
    final ReentrantLock lock;

    //等待策略
    private final Condition notEmpty;

    private final Condition notFull;

    //迭代器
    transient Itrs itrs = null;



##### ArrayBlockingQueue 的构造函数
ArrayBlockingQueue 的构造函数主要有两个：

    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
    
该构造器设置了队列的容量，使用的 **ReentrantLock** 是公平还是非公平的锁。也就是选择公平策略，在未对 fair 进行指定时，默认情况下用的 ReentrantLock 是 NonfairSync ，它将不能保证队列 FIFO 的特性，但能提高队列的吞吐量。如果 fair 设置为 True ，将能保证多线程环境下 FIFO 的特性，但会降低吞吐量。


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

上面这个构造器比第一个构造器多了 Collection 参数，在初始化时可以初始化集合元素并指定顺序。


##### enqueue() 入队操作

    private void enqueue(E x) {    
        //获取元素
        final Object[] items = this.items;        
       	//通过putIndex 确定队尾位置
        items[putIndex] = x;
        //如果putIndex 的长度等于数组长度那么设置到数组头部
        if (++putIndex == items.length)        
            putIndex = 0;
        //计数
        count++;
        notEmpty.signal();
    }
    
    
##### dequeue() 出队操作


    private E dequeue() {
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        //获取出队元素
        E x = (E) items[takeIndex];
        //设置为null
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }

##### removeAt() 删除指定下标的元素

    void removeAt(final int removeIndex) {
        final Object[] items = this.items;
        if (removeIndex == takeIndex) {
            //如果要删除takeIndex队头元素，时间复杂度为O(1)
            //不需要移动后面元素
            items[takeIndex] = null;
            if (++takeIndex == items.length)
                takeIndex = 0;
            count--;
            if (itrs != null)
                itrs.elementDequeued();
        } else {
            final int putIndex = this.putIndex;
            //找到要删除的元素使其后面的元素都前移
            for (int i = removeIndex;;) {
                int next = i + 1;
                if (next == items.length)
                    next = 0;
                if (next != putIndex) {
                    items[i] = items[next];
                    i = next;
                } else {
                    items[i] = null;
                    this.putIndex = i;
                    break;
                }
            }
            count--;
            if (itrs != null)
                itrs.removedAt(removeIndex);
        }
        notFull.signal();
    }

删除指定下标的元素，这个最坏复杂度为O(n)，这是数组本身的一个缺点。


##### offer() 对入队进行加锁操作

    public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }

通过 ReentrantLock 加锁，对 enqueue(e) 进行加锁封装。


##### put() 方法 


    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        //获取锁
        lock.lockInterruptibly();
        try {
        	//当队列满了，进行阻塞操作
        	//直到消费者消费队列，留出剩余空间，方可入队
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    
    
##### take() 方法 

    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        //lockInterruptibly 获取锁
        lock.lockInterruptibly();
        try {
        	//当队列为空时，进行阻塞操作，
        	//直到有生产者入队操作
            while (count == 0)
                notEmpty.await();
            //出队操作
            return dequeue();
        } finally {
            lock.unlock();
        }
    }



# LinkedBlockingQueue


##### LinkedBlockingQueue 的属性

	//初始容量，默认为Integer.MAX_VALUE
    private final int capacity;

    //通过AtomicInteger进行计数
    private final AtomicInteger count = new AtomicInteger();

    //队头节点
    transient Node<E> head;

    //队尾节点
    private transient Node<E> last;

    //锁
    private final ReentrantLock takeLock = new ReentrantLock();

    //等待策略
    private final Condition notEmpty = takeLock.newCondition();

    private final ReentrantLock putLock = new ReentrantLock();

    private final Condition notFull = putLock.newCondition();
    
    
从 **LinkedBlockingQueue** 的属性可以看出， **ArrayBlockingQueue** 和 **LinkedBlockingQueue** 一个不同之处在于， ArrayBlockingQueue 只使用了了一个锁，也就是读写 用的锁是同一个，而 LinkedBlockingQueue 用了 putLock 和 takeLock 两个锁，生产和消费的锁分离了。

##### LinkedBlockingQueue 的构造函数
LinkedBlockingQueue 的构造函数有下面三个：


    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
    
默认构造函数，调用 LinkedBlockingQueue(int capacity) 构造函数，将队列容量设置为 Integer.MAX_VALUE 。

    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }

可设置队列大小，初始化对头队尾节点。

    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                enqueue(new Node<E>(e));
                ++n;
            }
            count.set(n);
        } finally {
            putLock.unlock();
        }
    }
    
将集合初始化到队列中。


##### remainingCapacity() 方法

    public int remainingCapacity() {
        return capacity - count.get();
    }
    
该方法计算队列剩余容量。

##### remove() 方法 

    public boolean remove(Object o) {
        if (o == null) return false;
        fullyLock();
        try {
            for (Node<E> trail = head, p = trail.next;
                 p != null;
                 trail = p, p = p.next) {
                if (o.equals(p.item)) {
                    unlink(p, trail);
                    return true;
                }
            }
            return false;
        } finally {
            fullyUnlock();
        }
    }

remove() 进行删除元素，删除元素的时间复杂度为O(1)，这是链表的一个优点，不需要像 Array 这样移动元素的位置。


##### put() 方法

    public void put(E e) throws InterruptedException {
    	//判断入队节点是否为空
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            //当队列中的元素等于容量时，便会进行阻塞操作，
            //直到消费者消费队列，空出位置
            while (count.get() == capacity) {
                notFull.await();
            }
            //入队操作
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
    }



# 总结
ArrayBlockingQueue 和 LinkedBlockingQueue 分别是基于数组和链表实现的，适用于不同的场景。


队列中锁的不同实现：

- ArrayBlockingQueue
	
	ArrayBlockingQueue 进行出队和入队操作用的都是同一个锁，生产和消费用的是一个锁。
	
- LinkedBlockingQueue

	LinkedBlockingQueue 用了两个锁，takeLock 和 putLock，实现了生产和消费的分离，提高队列的吞吐量

进行生产和消费时的操作不同：

- ArrayBlockingQueue

	ArrayBlockingQueue 的生产消费是直接将元素插入和删除。

- LinkedBlockingQueue

	LinkedBlockingQueue 的生产消费执行前，要将元素转换为 Node ，再进行插入删除。

队列初始化的方式不同：

- ArrayBlockingQueue

	ArrayBlockingQueue 初始化容量固定后不可改变

- LinkedBlockingQueue

	LinkedBlockingQueue 可设置容量，不设置默认为 Integer.MAX_VALUE 。


	







