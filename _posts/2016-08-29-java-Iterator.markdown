---
layout:     post
title:      "java Iterator 用法"
subtitle:   "Iterator 使用方法及实现"
date:       2016-08-29
author:     "JianFeng"
header-img: "img/blog/2016-08-29-Iterator.png"
catalog: true
tags:
    - java
    - Iterator
    - 迭代器
---

> 本篇将介绍Iterator 的知识

## Iterator （迭代器）

### 为什么要使用迭代器 ？
对于任何容器，我们不可避免的是对它进行遍历、增加和删除的操作。那么我们使用不同的容器，必须对容器的确切类型进行编程。不同的容器有不同的数据结构，因此每个不同类型的容器有不同的Iterator迭代器，迭代器是用来遍历Collection子类的各个元素，java封装了这些数据结构，开发人员不需要知道内部具体怎么实现，仅当你想要遍历容器中的各个元素时你必须用collection.iterator()返回相应的迭代器，然后通过迭代器访问每一个元素。

### Iterator 定义
迭代器是一种设计模式，它能够遍历对象并选择序列中的对象，使用者不需要了解该序列底层的结构(例如List、Set)，迭代器的创建代价较小，通常被称为轻量级对象。

### Iterator 的功能

Iterator 接口代码：

<pre><code>
package java.util;

public interface Iterator<E> {
    /**
     * 检查序列中是否还有元素
     */
    boolean hasNext();

    /**
     * 获取序列下一个元素
     */
    E next();

    /**
     * 删除序列中当前元素
     */
    void remove();
}

</code></pre>

Java中的Iterator功能比较简单，并且只能单向移动：

1.使用方法iterator()要求容器返回一个Iterator。第一次调用Iterator的next()方法时，它返回序列的第一个元素。注意：iterator()方法是java.lang.Iterable接口,被Collection继承。

2.使用next()获得序列中的下一个元素。

3.使用hasNext()检查序列中是否还有元素。

4.使用remove()将迭代器新返回的元素删除。

### ArrayList 中 iterator 的实现
Java集合类库将集合的接口与实现分离。同样的接口，可以有不同的实现。Java集合类的基本接口是Collection接口，而Collection接口必须实现Iterator接口。

<pre><code>
package java.lang;

import java.util.Iterator;

public interface Iterable<T> {

    /**
     * Returns an iterator over a set of elements of type T.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}

</code></pre>

ArrayList 通过实现Collection接口，实现iterator()方法，通过返回iterator()方法返回 Itr 内部类

<pre><code>
public Iterator<E> iterator() {
        return new Itr();
}
</code></pre>

Itr 实现 Iterator接口，


<pre><code>
int cursor;           // cursor表示下一个元素的索引位置
int lastRet = -1;     // lastRet表示上一个元素的索引位置
int expectedModCount = modCount; //ArrayList不是线程安全的，modCount记录修改次数，expectedModCount初始化为 modCount
 
</code></pre>

checkForComodification()方法对修改进行同步检查，一旦发现这个对象的mcount和迭代器中存储的mcount不一样那就抛异常 

<pre><code>
final void checkForComodification() {
	if (modCount != expectedModCount)
		throw new ConcurrentModificationException();
}
 
</code></pre>

hasNext()方法通过 cursor 是否与 size(长度)相对，判断是否存在下一个元素

<pre><code>
public boolean hasNext() {
	return cursor != size;
}
 
</code></pre>

next() 方法实现比较简单，先返回cursor索引位置处的元素，然后修改cursor、lastRet

<pre><code>
public E next() {
	checkForComodification();
	int i = cursor; //记录索引位置  
	if (i >= size) //如果获取元素大于集合元素个数，则抛出异常
		throw new NoSuchElementException();
	Object[] elementData = ArrayList.this.elementData;  //获取ArrayList元素
	if (i >= elementData.length)
		throw new ConcurrentModificationException();
	cursor = i + 1; //cursor + 1  
	return (E) elementData[lastRet = i]; //lastRet + 1 且返回cursor处元素  
}
 
</code></pre>

remove() 方法调用了ArrayList自身的remove()方法删除lastRet位置元素，让cursor 等于 lastRet

<pre><code>
public void remove() {
	if (lastRet < 0)  //当lastRet<0抛出异常
		throw new IllegalStateException();
	checkForComodification();

	try {
		ArrayList.this.remove(lastRet);
		cursor = lastRet;    //设置cursor为当前被删除元素的位置
		lastRet = -1;
		expectedModCount = modCount;
	} catch (IndexOutOfBoundsException ex) {
		throw new ConcurrentModificationException();
	}
}
 
</code></pre>

### 举个栗子：

<pre><code>
public class Test {

    public static void main(String[] args){
        List<Integer> list = new ArrayList<Integer>();
        for(int i = 0;i < 10;i++ ){
            list.add(i);
        }
        Iterator<Integer> it = list.iterator();  //调用了ArrayList.iterator()方法，返回Iterator
        while(it.hasNext()){    //hasNext() 判断下一个元素是否存在
            int i = it.next();
            System.out.println(i);
        }
        it = list.iterator();
        for(int i = 0;i < 5;i++ ){  //对0~4的元素进行删除
            it.next();
            it.remove();
        }
        System.out.println(list);  //打印剩下的
    }
}
 
</code></pre>

输出：
<pre><code>
0
1
2
3
4
5
6
7
8
9
[5, 6, 7, 8, 9]
 
</code></pre>

有了Iterator就不需要为容器中元素的数量操心，Iterator会为我们解决。


Iterator只能向前移动，但是Iterator还有更强大的子类型，例如ListIterator可以进行双向移动。


