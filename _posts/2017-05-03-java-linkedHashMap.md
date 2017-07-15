---
layout:     post
title:      "HashMap 和 LinkedHashMap 的小坑"
subtitle:   "LinkedHashMap 的有序性"
date:       2017-05-03
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> LinkedHashMap 

# 稍不注意就会踩的 Map 小坑
开发中遇到一个需求，上游传来订单信息，通过比较订单信息，将符合一定规则的信息归并到一起，上游传过来的信息字段太多，信息之间字段两两匹配效率很低，想法是将一个字符串通过Hash算法生成Long型的值，并存到相应表中，Long型的比较会比文本比较效率高得多。主要解决思路是：将上游传的信息转换成Map，再转成JSON形式。

# HashMap 和 LinkedHashMap
开始直接就使用 HashMap 作为容器，在转化为 JSON 。一开始就入了坑，HashMap 插入数据并不保证有序，原本两个相同的信息插入 Hash ，最后通过 toString() 得到的字符串信息并不一定一样。

    public static void main(String[] args){
        Map<String, Object> hashMap = new HashMap<>();
        hashMap.put("name", "zjf");
        hashMap.put("age", 21);
        hashMap.put("sex", "boy");
        System.out.println("HashMap" + hashMap.toString());
        hashMap = new LinkedHashMap<>();
        hashMap.put("name", "zjf");
        hashMap.put("age", 21);
        hashMap.put("sex", "boy");
        System.out.println("LinkedHashMap" + hashMap.toString());
    }

得到的答案可能是:

	HashMap{sex=boy, name=zjf, age=21}
	LinkedHashMap{name=zjf, age=21, sex=boy}
	
也可能是:

	HashMap{sex=boy, name=zjf, age=21}
	LinkedHashMap{name=zjf, age=21, sex=boy}

可以看到 HashMap 是不保证插入有序的。LinkedHashMap 继承自 HashMap，但它能保证遍历元素时，输出的顺序和输入时的顺序相同。
