---
layout:     post
title:      "探索Java构造器"
subtitle:   "掌握Java构造器的知识"
date:       2016-08-25
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - 构造器
---

> 本篇将介绍构造器的知识


## 构造器 

### 构造器定义 
构造器是在创建对象时被自动调用的特殊方法，通过构造器，可以确保对象得到初始化，构造一个实例。构造器和方法是有区别的，构造器是一种特殊类型的方法，它没有返回值，也没有返回类型(例如void)。

### 构造器的类型 

- 默认构造函数
- 带参构造函数
- 无餐构造函数

### 默认构造函数 
在java中，当你在类中未定义构造函数时，java会默认生成一个构造函数，这个构造函数称为默认构造函数。默认的构造函数在源代码中是不可见的。这是默认构造函数演示： 

<pre><code>
public Demo(){

}

</code></pre>

### 带参构造函数 
带有参数的构造函数称为带参构造函数。带参构造函数的演示：

<pre><code>
public Demo{
	public Demo(int i,String str){
		System.out.println("这是一个带参的构造函数：" + i + " " + str);
	}
}

</code></pre>

### 如何调用一个带参的构造函数？
调用一个带参的构造函数，可以使用**new**关键字，然后加构造函数名，再引入参数，例如：
<pre><code>new Demo(1,"demo");
</code></pre>

### 无参构造函数
无参构造函数就是没有参数的构造函数，它的实现方法和默认构造函数一样，重要的是：它的方法体可以有代码，默认构造函数内没有代码。无参构造函数的演示：
<pre><code>
public Demo{
	public Demo(){
		System.out.println("这是一个无参的构造函数");
	}
}

</code></pre>

### 这是一个在类中不写构造函数的示例：
<pre><code>
public Demo{

	public void demoMethod(){
		System.out.println("Hello world");
	}

	public static void main(String[] args){
		Demo demo = new Demo();
		demo.demoMethod();
	}

}

</code></pre>

在public static void main 方法中，调用了**new Demo()**默认构造函数，在此之前我没有定义构造函数，java编译器也没有显示出来，说明在未定义新的构造函数时，java编译器会自动产生默认的构造函数。

### 当一个类继承了另一个类时，如果对子类进行实例化，那么将先调用第一个超类（父类）的构造函数，然后再调用子类构造函数。
看下面的例子：

class A：

<pre><code>
public class A {

    String str1,str2;

    public A(){
        str1 = "A class";
        str2 = "B class";
    }

}

</code></pre>

class B：
<pre><code>
public class B extends A {

    public B(){
        str2 = "子类B class";
    }

    public void disp(){
        System.out.println("str1 :" + str1);
        System.out.println("str2 :" + str2);
    }

    public static void main(String[] args){
        B b = new B();
        b.disp();
    }

}

</code></pre>


输出：
<pre><code>str1 :A class
str2 :子类B class
</code></pre>
在例子中，类A为B的父类，子类B进行实例化时，将先调用父类A的构造函数，这时str1 = "A class"，str2 = "B class"，然后才调用子类B的构造函数，这时str2 = "子类B class"。调用disp()方法，输出str1 :A class  ，str2 :子类B class。

###子类构造函数调用父类构造函数

class A：

<pre><code>
public class A {

    String str1,str2;

    public A(){
        str1 = "A class";
        str2 = "B class";
    }

    public A(String str){
        str1 = str;
    }

}

</code></pre>

class B：
<pre><code>
public class B extends A {

    public B(){
        super("super");
        str2 = "子类B class";
    }

    public void disp(){
        System.out.println("str1 :" + str1);
        System.out.println("str2 :" + str2);
    }

    public static void main(String[] args){
        B b = new B();
        b.disp();
    }

}

</code></pre>
输出：
<pre><code>str1 :super
str2 :子类B class
</code></pre>

super()应在构造函数的第一行。

## 构造函数和方法有何不同？
<table class="table table-bordered table-striped table-condesed">

	<tr>
		<td>类型</td>
		<td>构造函数</td>
		<td>方法</td>
	</tr>
	<tr>
		<td>功能</td>
		<td>创建一个类的实例</td>
		<td>java功能语句</td>
	</tr>
	<tr>
		<td>返回类型</td>
		<td>没有void，没有返回值</td>
		<td>有void或者返回值</td>
	</tr>
	<tr>
		<td>修饰</td>
		<td>不能用abstract，final，native，static，synchronized</td>
		<td>能</td>
	</tr>
	<tr>
		<td>命名</td>
		<td>与类名相同，大写开头</td>
		<td>以小写开头，一般为动词</td>
	</tr>
	<tr>
		<td>this</td>
		<td>指向同一个类中另一个个构造函数，写在第一行</td>
		<td>指向当前类的一个实例，不能用于静态方法</td>
	</tr>
	<tr>
		<td>super</td>
		<td>调用父类的构造函数，写在第一行</td>
		<td>调用父类中的一个重载方法</td>
	</tr>
	<tr>
		<td>继承</td>
		<td>不能被继承</td>
		<td>可以被继承</td>
	</tr>
	<tr>
		<td>编译器自动加入一个缺省的构造器</td>
		<td>自动加入(如果没有)</td>
		<td>不支持</td>
	</tr>
	<tr>	
		<td>编译器自动加入一个缺省的调用超类的构造器</td>
		<td>自动加入</td>
		<td>不支持</td>
	</tr>

</table>

## 总结

1. 每一个类都有一个构造函数，无论它是普通类或是一个抽象类。
2. 像上面所说的，构造函数不是方法，他们没有任何返回类型。
3. 构造函数的名称和类名称应该相同。
4. 构造函数可以使用任何访问修饰符，当构造函数是私有的，将无法被类以外的函数使用，只能由内部方法调用。
5. 如果你在类中没定义任何构造函数，那么java编译器会自动创建默认构造函数。
6. this()和super()方法应该在构造函数代码中的第一条语句。
7. 构造函数可以重载，但不能重写。
8. 构造函数不能被继承。
9. 如果父类没有无参数构造函数或者默认构造函数，正常情况下，编译器会为子类定义一个默认的构造函数。
10. 接口没有构造函数，因为它不能被实例化。
11. 抽象类虽然不能被实例化，但可以有构造函数。
