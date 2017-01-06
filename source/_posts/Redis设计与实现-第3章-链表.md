---
title: Redis设计与实现-第3章-链表
date: 2016-12-20 18:23:42
categories: Redis
tags: [Redis, 数据结构]
---

链表提供了高效的节点重排能力，以及顺序访问的节点访问方式，并且可以通过增删节点来灵活地调整链表的长度。
```shell
redis> LIEN integers
(integer) 1024

redis> LRANGE integers 0 5
1)"1"
2)"2"
3)"3"
4)"4"
5)"5"
```
<!--more-->

## 链表和链表节点的实现
每个链表节点使用一个adlist.h/listNode结构来表示：
```c
typedef struct liskNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
} listNode;
```

使用adlist.h/list来持有链表：
```c
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 链表所包含的节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
} list;
```

Redis的链表实现的特性可以总结如下：
- 双端
- 无欢
- 带表头指针和表尾指针
- 带链表长度计数器
- 多态，链表节点使用void*指针来保存节点值
