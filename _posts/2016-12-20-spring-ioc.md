---
layout:     post
title:      "Spring IOC 容器的实现"
subtitle:   "学习 Spring IOC 容器的实现"
date:       2016-12-20
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> 本篇将介绍 Spring IOC 容器的实现

# Spring IOC
Spring 的核心有两个，一个是 IOC 容器和 AOP 依赖注入。

### BeanFactory 

BeanFactory 定义了最基本的 IOC 功能，该接口的定义勾勒出了 IOC 容器的基本轮廓，BeanFactory 定义了哪些功能呢？

	public interface BeanFactory {
	
		String FACTORY_BEAN_PREFIX = "&";
		
		Object getBean(String name) throws BeansException;
		
		<T> T getBean(String name, Class<T> requiredType) throws BeansException;
		
		<T> T getBean(Class<T> requiredType) throws BeansException;
		
		Object getBean(String name, Object... args) throws BeansException;
		
		<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
		
		boolean containsBean(String name);
		
		boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
		
		boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
		
		boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
		
		boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
		
		Class<?> getType(String name) throws NoSuchBeanDefinitionException;
		
		String[] getAliases(String name);
		
	
	}


##### FACTORY_BEAN_PREFIX

可以看到 BeanFactory 中定义了一个变量 FACTORY_BEAN_PREFIX ，该变量初始值为 ‘&’ 。这变量主要的功能是获取 FactoryBean 本身，**FactoryBean** ？FactoryBean 和 BeanFactory 那么相像。。。但他们的本质是不同的。FactoryBean 从字面上可以看出他是一个工厂Bean，他是一个特殊的 Bean ，它能够生产和修饰对象，他的实现和工厂模式相类似。BeanFactory 是一个 Factory ，所有的 Bean 都是由 BeanFactory 来进行管理。

<T> T getBean(Class<T> requiredType)，该方法用于获取与 requiredType 类型相匹配的 Bean。


##### getBean() 方法

该方法用于获取一个 Bean ，这个方法是 IOC 的核心方法，通过这个方法我们可以获取到相应的 Bean ，该方法通过 BeanName 来索引的。

##### containsBean() 方法

该方法通过传入 name 参数来判断这个容器是否包含有指定容器的 Bean。

##### isSingleton() 方法

isSingleton() 方法用于判断这个 Bean 是否属于 Singleton 类型，用户可以在 BeanDifinition 中定义。


##### isPrototype() 方法

该方法通过传入的 name ，判断该 Bean 是否属于 Prototype 类型。

##### isTypeMatch() 方法

查询指定名字的 Bean 是否是特定的 Class 类型，这个 Class 类型可以由用户进行指定。


##### getType() 方法

查询指定名字的 Bean 的 Class 类型。


##### getAliases() 方法

该方法查询指定名字的 Bean 的别名，该别名可以在 BeanDifinition 中定义。


**可以看到通过 BeanFactory 的接口定义，我们能很快地找到我们想找的 Bean ，程序员不需要了解具体实现，十分方便。**


### HierarchicalBeanFactory

通过源码可以看到 HierarchicalBeanFactory 继承 BeanFactory，并在 BeanFactory 基础上，添加了两个方法：

##### getParentBeanFactory() 方法

该方法使得 BeanFactory 能具备双亲IOC容器的管理功能。

##### containsLocalBean() 方法





















































