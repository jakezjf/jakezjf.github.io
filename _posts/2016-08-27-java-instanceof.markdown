---
layout:     post
title:      "java instanceof 用法"
subtitle:   "instanceof 用法"
date:       2016-08-27
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
---

> 本篇将介绍instanceof的知识

## instanceof

### 什么是instanceof ？
instanceof是java的一种运算符，instanceof运算符是双目运算符，左面的操作元是一个对象，右面是一个类。当左面的对象是右面的类创建的对象时，该运算符运算的结果是tru，否则是false。

看代码：
<pre><code>
interface A {
//接口A
}

public class B implements A {
//类B实现接口A
}

public class C extends B {
//类C继承类B
}


public class Main {

    public static void main(String[] args){
        A ab = new B();
        A ac = new C();
        B bb = new B();
        B bc = new C();
        C cc = new C();
        //对象都实现了同一个接口A，每个对象和接口A进行 instanceof，为true
        System.out.println("ab instanceof A " + (ab instanceof A));
        System.out.println("ac instanceof A " + (ac instanceof A));
        System.out.println("bb instanceof A " + (bb instanceof A));
        System.out.println("bc instanceof A " + (bc instanceof A));
        System.out.println("cc instanceof A " + (cc instanceof A));
        //对象和父类进行instanceof，为true
        System.out.println("ab instanceof B " + (ab instanceof B));
        System.out.println("ac instanceof B " + (ac instanceof B));
        System.out.println("bb instanceof B " + (bb instanceof B));
        System.out.println("bc instanceof B " + (bc instanceof B));
        System.out.println("cc instanceof B " + (cc instanceof B));
        //对象和子类进行instanceof时，为false
        System.out.println("ab instanceof C " + (ab instanceof C));
        System.out.println("ac instanceof C " + (ac instanceof C));
        System.out.println("bb instanceof C " + (bb instanceof C));
        System.out.println("bc instanceof C " + (bc instanceof C));
        System.out.println("cc instanceof C " + (cc instanceof C));

    }
}
</code></pre>
输出结果：
<pre><code>
ab instanceof A true
ac instanceof A true
bb instanceof A true
bc instanceof A true
cc instanceof A true
ab instanceof B true
ac instanceof B true
bb instanceof B true
bc instanceof B true
cc instanceof B true
ab instanceof C false
ac instanceof C true
bb instanceof C false
bc instanceof C true
cc instanceof C true

</code></pre>

可以看出：

- 当对象实现了某接口，该对象和接口进行instanceof运算时，都返回true
- 子类对象和其父类对象进行instanceof运算时，都返回true
- 当父类对象和其子类对象进行instanceof运算时，都返回false


### instanceof 运行时的语义

<pre><code>
public class Test {

    public static void main(String[] args){
        boolean result;
        B b = new B();
        try{
            C c = (C) b;
            result = true;
        }catch (ClassCastException e){
            result = false;
        }
        System.out.println(result);
    }

}

</code></pre>
类B是类C的子类，Java中父类强制转换成子类的原则是：父类型的引用指向的是哪个子类的实例，就能转换成哪个子类的引用。代码中父类B指向的是本身的实例，所以会抛 ClassCastException 异常，result为false。

输出：
<pre><code>false
</code></pre>

### instanceof 实现原理

Javac的工作流程：
源码——词法分析器——Token流——语法分析器——语法树——语义分析器——注解语法树——代码生成器——字节码

instanceof 是java的一个关键字，javac在编译java代码时，能够识别instanceof 关键字，对应到Token.INSTANCEOF的token类型，对源代码进行词法分析，当扫描到instanceof 关键字时，将其映射到Token.INSTANCEOF。[移步Token的源码](http://hg.openjdk.java.net/jdk7u/jdk7u/langtools/file/tip/src/share/classes/com/sun/tools/javac/parser/Token.java)

javac的抽象语法树节点有一个JCTree.JCInstanceOf类用于表示instanceof运算。在做语法分析过程中，解析到instanceof 运算符就会生成JCTree.JCInstanceOf类型的结点，[JavacParser通过term2Rest()方法实现](http://hg.openjdk.java.net/jdk7u/jdk7u/langtools/file/tip/src/share/classes/com/sun/tools/javac/parser/JavacParser.java)。

