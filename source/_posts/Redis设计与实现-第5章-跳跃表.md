---
title: Redis设计与实现-第5章-跳跃表
date: 2016-12-22 19:34:18
categories: Redis
tags: [Redis, 数据结构]
---
跳跃表(skiplist)是一种有序数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。

跳跃表平均支持O(logN)、最坏O(N)复杂度的节点查找，还可以通过顺序性操作来批量处理节点。

Redis使用跳跃表作为有序集合键的底层实现之一，如果一个有序集合包含的元素数量比较多，又或者有序集合中元素的成员是比较长的字符串时，Redis就会使用跳跃表来作为有序集合键的底层实现。
- 实现有序集合键
- 在集群节点中用做内部数据结构
<!-- more -->

## 跳跃表的实现
Redis的跳跃表由redis.h/zskiplistNode和redis.h/zskiplist两个结构定义

### 跳跃表节点
跳跃表节点的实现由redis.h/zskiplistNode结构定义：
```c
typedef struct zskiplistNode {
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
} zskiplistNode;
```
#### 层
跳跃表节点的level数组可以包含多个元素，每个元素都包含一个指向其他节点的指针，程序可以通过这些层来加快访问其他节点的速度，一般来说，层的数量越多，访问其他节点的速度就越快。

每次创建一个新跳跃表节点的时候，程序都根据幂次定律随机生成一个结余1和32之间的值作为level数组的大小，这个大小就是层的高度。

#### 前进指针
每个层都有一个指向表尾方向的前进指针，用于从表向表尾方向访问节点

#### 跨度
层的跨度用于记录两个节点之间的距离。
- 两个节点之间的跨度越大，它们就相距得越远。
- 指向null的所有前进指针的跨度都为0，因为它们没有连向任何节点

#### 后退指针
节点的后退指针用于从表尾向表头方向访问节点：跟可以一次跳过多个节点的前进指针不同，因为每个节点只有一个后退指针，所以每次只能后退至前一个节点。

#### 分值和成员
分值是一个double类型的浮点数，跳跃表中的所有节点都按分值从小到大来排序。
成员是一个指针，它指向一个字符串对象，而字符串对象则保存着一个SDS值。

### 跳跃表
redis.h/zskiplist定义如下：
```c
typedef struct zskiplist {
    // 表头和表尾节点
    struct skiplistNode  *header, *tail;
    // 表中节点数量，表头不算
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```
