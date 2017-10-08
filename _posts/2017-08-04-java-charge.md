---
layout:     post
title:      "链表转数组"
subtitle:   "链表转数组"
date:       2017-08-04
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> 链表转数组

# 链表转成数组
集合转成数组是开发中经常遇到的？如何将链表转成数组呢？

比较简单的是采用赋值的方法:

	public static void main(String[] args){
		ArrayList<String> list = new ArrayList<>();
		list.add("zhong");
		list.add("feng");
		String[] arr = new String[list.size()];
		int index = 0;
		for(String str:list){
			arr[index++] = str;
		}
	}

可以使用赋值的方法:


	public static void main(String[] args){
		ArrayList<String> list = new ArrayList<>();
		list.add("zhong");
		list.add("feng");
		String[] arr = list.toArray(new String[0]);
	}

第二种方法是官方推荐的写法。
