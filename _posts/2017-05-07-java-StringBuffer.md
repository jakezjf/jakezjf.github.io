---
layout:     post
title:      "Java 性能优化之 StringBuffer、StringBuilder"
subtitle:   "StringBuffer 和 StringBuilder 的合理使用"
date:       2017-05-07
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> StringBuffer 和 StringBuilder 的合理使用

# Java 性能优化之 StringBuffer、StringBuilder
String 是我们开发中最常用的类之一，但它是一个不可变的类，在进行大量的字符串拼接操作时，会产生大量的新的 String 对象，会吃掉大量的内存。通常在这种场景下我们不会使用 String 类，而会使用 StringBuffer 。

StringBuffer 是平时开发中常用的 Java 容器对象，它的底层数据存储类型是 char[] ，直接对 char 数组操作，再也不用 new 新对象了。

但是随着新元素的不断添加，StringBuffer 也要相应的调整字符数组的大小，调整的方法是将原先的char数组扩大一倍，将原char数组的元素腾到新生成的char数组中，而原数组的元素将被废弃，这个新生成的char数组长度是原char数组长度的两倍。

通过 StringBuffer 可以探寻它是怎么扩展的。

# StringBuffer 源码解析

##### 构造函数

	public StringBuffer() {
        super(16);
    }
    
StringBuffer 继承了 AbstractStringBuilder

    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
    
new 一个长度为 capacity 的 char 数组。


    public StringBuffer() {
        super(16);
    }
    
可以看到，默认生成的 char 数组长度为 16。

    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }
    
还可以传入字符串进行初始化。

##### append() 方法

    AbstractStringBuilder append(AbstractStringBuilder asb) {
        if (asb == null)
            return appendNull();
        int len = asb.length();
        ensureCapacityInternal(count + len);
        asb.getChars(0, len, value, count);
        count += len;
        return this;
    }
    
ensureCapacityInternal() 方法

    private void ensureCapacityInternal(int minimumCapacity) {
    	 //count + length 是字符串拼接后的总长度
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));
        }
    }

当 minimumCapacity - value.length > 0 时，说明原字符串和新字符串进行拼接时，长度大于 char 数组长度，这时候就通过 Arrays.copyOf(value, newCapacity(minimumCapacity)) 进行扩容。

newCapacity() 方法

    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int newCapacity = (value.length << 1) + 2;
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
    
char 数组扩大了两倍。


# 性能优化
StringBuffer 相对于 String 而言，可以减少内存消耗，正确的使用 StringBuffer 还能更好的优化程序性能。


##### 场景1
通常在实际运用中可能遇到 StringBuffer 、 StringBuilder 对象实例只使用16个或更少字符数组元素的情况。

    public static void main(String[] args){
        StringBuffer stringBuffer = new StringBuffer(10);
        stringBuffer.append("Hello");
        stringBuffer.append("World");
    }

能减少内存的消耗，适用于提前知道字符传长度的场景，这个场景貌似比较少。

##### 场景2

在实际应用中极少出现 StringBuilder 或 StringBuffer 对象实例最后只使用16或更少字符数组元素的情况，为了避免出现调整 StringBuffer 大小的情况，可以事前指定初始化容量。

    public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer(100000);
        for (int i = 0; i < 100000; i++) {
            stringBuffer.append(i);
        }
    }
   
# 总结
合理的初始化 StringBuffer ，或多或少能给系统提升性能。现在 JVM 已经可以分析 StringBuffer 和 StringBuilder 的使用情况，并尝试对 StringBuffer 和 StringBuilder char 数组长度进行优化，减少由于 StringBuilder 和 StringBuffer 的增大所引起的不必要的 char[] 对象。










