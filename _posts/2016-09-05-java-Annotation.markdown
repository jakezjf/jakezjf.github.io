---
layout:     post
title:      "java 注解的用法"
subtitle:   "深入了解Java自定义注解"
date:       2016-09-05
author:     "JianFeng"
header-img: "img/blog/birthdaylevelup.jpg"
catalog: true
tags:
    - java
    - 注解
    - Annotation
---

> 本篇将介绍Java注解的知识

# 注解

## 注解的定义

注解为我们在代码中添加信息提供了一种形式化的方法，是我们可以在某个时刻非常方便的使用数据。注解在一定程度上是把元数据与源代码文件集合起来，而不是保存在外部文档中。在实际开发中，我们常见到三种JavaSE内置的注解，这些注解定义在java.lang中:

- @Override
表示当前的方法定义将覆盖超类中的方法

- @Deprecated 是一个标记注释。所谓标记注释，就是在源程序中加入这个标记后，并不影响程序的编译，但有时编译器会显示一些警告信息。在IDE中有时会看到属性和方法带有这个注解，如果某个类成员的提示中出现了个词，就表示这个并不建议使用这个类成员。因为这个类成员在未来的JDK版本中可能被删除。


- @SuppressWarnings 关闭不当的编译器警告信息

![img](/img/blog/zhujie.jpg)

## 基本的语法
	
	public class Example{
	
	    @Test void print(){}

	}

例子中使用@Test 对print()方法进行注解，注解@Test 可以任何修饰符进行搭配使用。

## 定义注解

我们可以使用java自带的注解，也可以使用自定义注解。但想要自定义注解就必须要了解Java为我们提供的元注解和相关定义注解的语法。

### 元注解

元注解的作用就是负责注解其他注解，在Java 5.0中定义了四个标准的meta-annotation类型，它们被用来提供其他注解的说明。以下是四个元注解：

1. @Target

2. @Retention

3. @Documented

4. @Inherited

上述的四个元注解都在 java.lang.annotation 包中


#### 1. @Target

@Target 用来说明Annotation所修饰的对象范围，可被用于包(package)、接口(interface)、类(class)、方法、构造器、域声明、变量。

ElementType参数有：

1.CONSTRUCTOR:用于描述构造器

2.FIELD:用于描述域

3.LOCAL_VARIABLE:用于描述局部变量

4.METHOD:用于描述方法

5.PACKAGE:用于描述包

6.PARAMETER:用于描述参数

7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

#### 2. @Retention

@Retention **表示需要在什么级别保存该注解信息，用于描述注解的生命周期**，RetentionPolicy参数有：

1.SOURCE:在源文件中有效（即源文件保留）

2.CLASS:在class文件中有效（即class保留）

3.RUNTIME:在运行时有效（即运行时保留） 虚拟机将在运行期也保留该注解，可以通过**反射机制**读取注解信息。

#### 3. @Documented

@Documented 将此注解包含在 javadoc 中，**Documented是一个标记注解，没有成员。**

#### 4. @Inherited

@Inherited 允许子类继承父类中的注解，使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。

当@Inherited annotation类型标注的annotation中的Retention类型为RUNTIME，反射API会增强这种继承性。通过反射机制检测class和它的父类，直到发现annotation类型或到达继承结构的顶层。


### 对类进行注解

**例子：**

UserAnnotation.java

	import java.lang.annotation.*;
	
	/**
	 * Created by JF
	 */
	@Retention(RetentionPolicy.RUNTIME) //设置Retention在运行时有效
	@Target(ElementType.TYPE)  //描述类和接口
	@Documented
	public @interface UserAnnotation {
	
	    String value() default "";
	
	}



User.java

	
	/**
	 * Created by JF 
	 */
	@UserAnnotation(value = "zhujie")
	public class User {
	
	    private String userId;
	
	    private String userName;
	
	    public String getUserId() {
	        return userId;
	    }
	
	    public void setUserId(String userId) {
	        this.userId = userId;
	    }
	
	    public String getUserName() {
	        return userName;
	    }
	
	    public void setUserName(String userName) {
	        this.userName = userName;
	    }
	    
	}

Main.java

	/**
	 * Created by JF 
	 */
	public class Main {
	
	    public static void main(String[] args){
	        Class<User> user = User.class;
	        if (user.isAnnotationPresent(UserAnnotation.class)){ 
	            System.out.println("这是一个注解类");
	            UserAnnotation userAnnotation = (UserAnnotation)user.getAnnotation(UserAnnotation.class);
				//java.lang.Class的getAnnotation方法，如果有注解，则返回注解。否则返回null
	            if (userAnnotation != null){
	                System.out.println(userAnnotation.value());
	            }else{
	                System.out.println("userAnnotation 为NULL");
	            }
	        }else {
	            System.out.println("这不是一个注解类");
	        }
	    }
	}

输出：

	这是一个注解类
	zhujie


### 对方法进行注解

UserAnnotation.java

	/**
	 * Created by JF
	 */
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)  
	@Documented
	public @interface UserAnnotation {
	
	    public String getUserId() default "";
	
	}


User.java
	
	/**
	 * Created by JF 
	 */
	public class User {
	
	    private String userId;
	
	    @UserAnnotation(getUserId = "123")
	    public String getUserId() {
	        return userId;
	    }
	
	    public void setUserId(String userId) {
	        this.userId = userId;
	    }
	}


Main.java

import java.lang.reflect.Method;

	/**
	 * Created by JF
	 */
	public class Main {
	
	    public static void main(String[] args){
	        Class<User> user = User.class;
	        try {
	            Method method = user.getDeclaredMethod("getUserId");
	            if (method.isAnnotationPresent(UserAnnotation.class)){
	                System.out.println("这是一个注解方法");
	                UserAnnotation userAnnotation = method.getAnnotation(UserAnnotation.class);
	                if (userAnnotation !=null ){
	                    System.out.println(userAnnotation.getUserId());
	                }else {
	                    System.out.println("userAnnotation 为NULL");
	                }
	            }else{
	                System.out.println("这不是一个注解方法");
	            }
	        } catch (NoSuchMethodException e) {
	            e.printStackTrace();
	        }
	    }
	}

输出：

	这是一个注解方法
	123

## 总结

注释是JavaSE5.0提供的一项非常有趣的功能。它不但有趣，而且还非常有用。还有Hibernate3除了使用传统的方法生成hibernate映射外，也可以使用注释来生成hibernate映射。总之，如果能将注释灵活应用到程序中，将会使你的程序更加简洁和强大。

