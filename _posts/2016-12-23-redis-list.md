---
layout:     post
title:      "Redis 中链表的设计与实现"
subtitle:   "学习和探索 Redis 中 链表 的实现原理"
date:       2016-12-23
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Redis
---

> 本篇将介绍 Redis 中 链表 的实现原理



# redis 中的链表

 链表是我们常用的数据结构之一，链表采用顺序访问，在对节点进行删除时时间复杂度为 O(1) ，能够灵活的进行删除节点，改变链表的长度。作为一种常用的数据结构，链表内置在很多高级编程语言中，链表在 redis 内部的使用十分广泛，列表建就是链表实现的。

### redis 中节点的定义

##### ListNode

	typedef struct listNode {
	    struct listNode *prev;
	    struct listNode *next;
	    void *value;
	} listNode;
	
redis 在 ListNode 定义了该节点的前驱和后继，以及该节点的值，多个节点链接起来，就是一个双链表。

##### List

为了使操作链表更加灵活，redis 定义了一个 list ，让 list 持有这个链表，下面是 List 的定义：

	
	typedef struct list {
		//表头节点
	    listNode *head;
	    //表尾节点
	    listNode *tail;
	    //复制方法
	    void *(*dup)(void *ptr);
	    //释放方法
	    void (*free)(void *ptr);
	    //比较方法
	    int (*match)(void *ptr, void *key);
	    //链表长度
	    unsigned long len;
	} list;
	
- head 指向头节点

head 能够快速帮组我们找到表头节点的位置。

- tail 指向尾节点

tail 能够快速帮我们找到表尾节点的位置，时间复杂度为 O(1)。

- dup 方法

dup 方法用于复制链表保存的值。

- free 方法

free 方法用于释放表节点保存的值。

- match

match 方法用于比较表节点的值和输入的值。

- len

len 记录了链表的长度，使得我们获取链表长度时，时间复杂度为 O(1) ，我们在获取链表长度时不需要再去遍历整个链表了，大大提高了我们的效率。



## redis 各种方法的实现


##### listCreate() 方法


list *listCreate(void)
{
    struct list *list;

    if ((list = zmalloc(sizeof(*list))) == NULL)
        return NULL;
    list->head = list->tail = NULL;
    list->len = 0;
    list->dup = NULL;
    list->free = NULL;
    list->match = NULL;
    return list;
}

 
listCreate() 方法用于创建一个空的链表，并且返回这个链表，时间复杂度为 O(1) 。


##### listAddNodeHead() 方法

	
	list *listAddNodeHead(list *list, void *value)
	{
	    listNode *node;
	
	    if ((node = zmalloc(sizeof(*node))) == NULL)
	        return NULL;
	    node->value = value;
	    if (list->len == 0) {
	    	//如果list为空，那么node就作为该list的表节点和尾节点，
	    	//因为链表就一个节点，所以它既是尾节点也是头节点
	        list->head = list->tail = node;
	        node->prev = node->next = NULL;
	    } else {
	    	//如果list不为空，那么node将作为头节点
	        node->prev = NULL;
	        node->next = list->head;
	        list->head->prev = node;
	        list->head = node;
	    }
	    list->len++;
	    return list;
	}
	
该方法用于将指定值添加到表头。时间复杂度为 O(1)。

##### listAddNodeTail() 方法

	list *listAddNodeTail(list *list, void *value)
	{
	    listNode *node;
	
	    if ((node = zmalloc(sizeof(*node))) == NULL)
	        return NULL;
	    node->value = value;
	    if (list->len == 0) {
	        list->head = list->tail = node;
	        node->prev = node->next = NULL;
	    } else {
	        node->prev = list->tail;
	        node->next = NULL;
	        list->tail->next = node;
	        list->tail = node;
	    }
	    list->len++;
	    return list;
	}

和上面的方法实现差不多，在尾节点添加一个新值，通常我们要在双链表的表尾添加一个新值，通常需要遍历整个链表获取尾节点，但我们之前定义的 list 已经记录下表尾节点的位置了，所以我们能够使用时间复杂度 O(1) 就能找到它。


##### listInsertNode() 方法

	list *listInsertNode(list *list, listNode *old_node, void *value, int after) {
	    listNode *node;
	
	    if ((node = zmalloc(sizeof(*node))) == NULL)
	        return NULL;
	    node->value = value;
	    if (after) {
	        node->prev = old_node;
	        node->next = old_node->next;
	        if (list->tail == old_node) {
	            list->tail = node;
	        }
	    } else {
	        node->next = old_node;
	        node->prev = old_node->prev;
	        if (list->head == old_node) {
	            list->head = node;
	        }
	    }
	    if (node->prev != NULL) {
	        node->prev->next = node;
	    }
	    if (node->next != NULL) {
	        node->next->prev = node;
	    }
	    list->len++;
	    return list;
	}
listInsertNode() 方法将新值添加到指定节点的前或者后，时间复杂度为O(1)。

##### listDelNode() 方法


	void listDelNode(list *list, listNode *node)
	{
	    if (node->prev)
	    	//将node节点的后继地址赋值给node节点的前驱的后继指针
	        node->prev->next = node->next;
	    else
	    	//如果node->prev为空说明node为表头节点，那么将list的head指向node的
	    	//后继节点
	        list->head = node->next;
	    if (node->next)
	        node->next->prev = node->prev;
	    else
	        list->tail = node->prev;
	    if (list->free) list->free(node->value);
	    zfree(node);
	    list->len--;
	}


listDelNode() 方法用于删除链表中的给定值，时间复杂度为 O(1)。


##### listDup() 方法

	list *listDup(list *orig)
	{
	    list *copy;
	    listIter iter;
	    listNode *node;
	
	    if ((copy = listCreate()) == NULL)
	        return NULL;
	    copy->dup = orig->dup;
	    copy->free = orig->free;
	    copy->match = orig->match;
	    listRewind(orig, &iter);
	    while((node = listNext(&iter)) != NULL) {
	        void *value;
	
	        if (copy->dup) {
	            value = copy->dup(node->value);
	            if (value == NULL) {
	                listRelease(copy);
	                return NULL;
	            }
	        } else
	            value = node->value;
	        if (listAddNodeTail(copy, value) == NULL) {
	            listRelease(copy);
	            return NULL;
	        }
	    }
	    return copy;
	}


listDup() 方法用于复制链表到一个新的链表。时间复杂度为 O(1)。


##### ListLength

返回链表的长度，时间复杂度为 O(1)。

##### ListFist

返回链表的头节点，时间复杂度为 O(1)。

##### Listlast

返回链表的尾节点，时间复杂度为 O(1)。


##### ListPrevNode

返回指定节点的前驱，时间复杂度为 O(1)。

##### ListNextNode

返回指定节点的后继，时间复杂度为 O(1)。

##### ListNodeValue

返回给定节点的 value ，时间复杂度为 O(1)。


# 总结

- redis 的链表表头节点的前驱指向NULL，表尾节点的后继指向NULL，实现了一个无环的链表。
- redis 链表的每个节点都有前驱和后继，属于双链表。
- 链表被 redis 广泛用于实现各种功能，例如：订阅和发布、慢查询等等。
- 每个链表用 list 进行管理。

研究了 redis 的链表实现，可以看出 redis 的链表通过 list 管理，变得效率很高，时间复杂度大多都为 O(1) 和 O(N)，这样的设计使得 redis 能够承受很高的并发量，性能十分优异。
 










	
	
