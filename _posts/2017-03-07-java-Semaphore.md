---
layout:     post
title:      "Java 并发之 Semaphore"
subtitle:   "学习 java 并发编程之 Semaphore"
date:       2017-03-07
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

>学习 Java 并发编程之 Semaphore

# Java 并发之 Semaphore

Semaphore 类在 Java concurrent包中，该类是一个可计数的信号量，该类维护了一定数量的信号许可集，在许可可用前会阻塞每一个 acquire() ，然后再获取该的许可。可以通过 release() 方法来添加一个许可，同时就解放了一个正在阻塞的信号量。通过 acquire() 和 release() 方法可以获取和释放访问的信号量。



创建一个 ServiceDemo 类，通过 Semaphore 来控制该对象 test() 方法的并发执行，

```
public class ServiceDemo {

	//设置同时拥有个许可，也就是只允许1个线程并发执行semaphore.acquire()
	//和semaphore.release(); 之间的代码块
    private Semaphore semaphore = new Semaphore(1);

    public void test(int flag){
        try {
            semaphore.acquire();
            System.out.println(System.currentTimeMillis() + "  " + flag);
            Thread.sleep(5000);
            System.out.println(System.currentTimeMillis() + "  " + flag);
            semaphore.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```



创建 ThreadDemo 类并继承 Thread：

```
public class ThreadDemo extends Thread{

    private ServiceDemo serviceDemo;

    private int flag;

    public ThreadDemo(ServiceDemo seviceDemo,int flag){
        super();
        this.serviceDemo = seviceDemo;
        this.flag = flag;
    }

    @Override
    public void run() {
        serviceDemo.test(flag);
    }
}
```



主函数：

```
public static void main(String[] agrs){
    ServiceDemo serviceDemo = new ServiceDemo();
    for(int i = 0;i<5;i++){
        new ThreadDemo(serviceDemo,i).start();
    }
}
```



运行主函数打印：

```
1494424236960  0

1494424241961  0

1494424241961  1

1494424246966  1

1494424246966  2

1494424251969  2

1494424251969  3

1494424256971  3

1494424256971  4

1494424261974  4

```

从打印的信息可以看出，同一时刻只有一个线程执行 test() 。

将 private Semaphore semaphore = new Semaphore(1);

改成  private Semaphore semaphore = new Semaphore(5);

运行结果：

```
1494424886910  0

1494424886910  2

1494424886910  1

1494424886910  3

1494424886910  4

1494424891911  0

1494424891911  2

1494424891911  3

1494424891911  4

1494424891911  1

```

可以看出，五个线程同时执行了 test() 方法功能。



# 总结

Semaphore 在许多场景都使用，比如在一些场景需要限制并发数量，限制管理员的个数来执行任务，限制消费者(生产者)的数量。使用 Semaphore 有一个缺点是，当限制的许可个数大于1个时，会导致代码块会被多个线程同时执行，会导致脏数据的出现，适合对数据一致性要求不高或数据不会修改的场景。