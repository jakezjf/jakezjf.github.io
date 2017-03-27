

2017-01-22
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









