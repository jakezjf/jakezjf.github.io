---
layout:     post
title:      "Java 性能优化之静态工厂方法"
subtitle:   "静态工厂方法代替构造函数"
date:       2017-03-03
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

>Java 性能优化之静态工厂方法

# Java 性能优化之静态工厂方法

在项目开发中，我们经常遇到一个问题，一个类有许多构造器，然而构造器的名字都是一样的，只有构造器的参数不一样，如果没有很好的API去说明，那么其他人使用这个类会十分麻烦，有一些类的功能比较简单，可以使用静态工厂方法来代替构造器。



# 使用静态工厂方法代替构造器有以下几个优势



##### 有签名

类的构造器名字都是一样的，除了传入的参数，当参数十分复杂的情况下，开发人员使用该类会很不方便。使用静态工厂方法，我们可以为方法起不同的名字，这个签名可以使我们了解这个方法具体是干吗用的，有利于开发人员使用该类的方法。



##### 不必创建对象 

使用静态工厂方法不需要创建新的对象，过多的对象会占用 JVM 堆的内存，从而会降低程序的性能，





看看下面的实例：

定义类 Sum ：

```
public class Sum {

    public static int getSum(int i,int j){
        return i+j;
    }

    public int getSums(int i,int j){
        return i+j;
    }

}
```

类 Sum 有一个静态方法 getSum() 和一个动态方法 getSums() 

调用静态方法不需要创建一个新的对象，调用动态方法需要创建新的对象。



```
public static void main(String[] agrs){
    long start = System.currentTimeMillis();
    for(int i = 0;i<1000000000;i++){
        Sum sum = new Sum();
        sum.getSums(1,1);
    }
    System.out.println(System.currentTimeMillis() - start);
    start = System.currentTimeMillis();
    for(int i = 0;i<1000000000;i++){
        Sum.getSum(1,1);
    }
    System.out.println(System.currentTimeMillis() - start);

}
```



打印结果为：

8
1

可以看到使用静态工厂方法比使用构造器的性能高很多。



##### 返回原返回类型的子类型对象

使用静态工厂方法可以选择返回该对象类型也可以返回原类型的子类型对象，这是十分方便的，可以为开发人员提供不同的策略。



在 java.util 包中的 EnumSet 类，它没有公有构造函数，可以使用 EnumSet.noneOf 静态方法，当它的底层枚举元素小于等于 64 时，返回 RegularEnumSet 对象，用单个 long 来存储；当枚举元素大于 64 时，返回 JumboEnumSet 对象，用 long 数组来存储。

```
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```





















