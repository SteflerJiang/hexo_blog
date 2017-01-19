---
title: Redis设计与实现-第8章-对象
date: 2017-01-06 19:56:25
categories: Redis
tags: [Redis, 数据结构]
---
前面陆续介绍了Redis用到的所有主要数据结构。但是Redis并没有**直接**使用这些数据结构来实现键值对数据库，而是给予这些数据结构创建了一个对象系统，这个系统包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象这五种类型的对象，每种对象都用到了至少一种我们前面所介绍的数据结构。

好处是：

- 可以根据对象的类型来判断一个对象是否可以执行给定的命令
- 针对不同的使用场景来为对象设置多种不同的数据结构实现，从而优化对象在不同场景下的使用效率

Redis的对象系统实现了基于引用计数的的内存回收机制；还通过引用技术技术实现了对象共享机制，可以让多个数据库键共享同一个对象来节约内存。

Redis对象带有访问时间记录信息，该信息可以同于计算数据库键的空转时长，在服务器启用了maxmemory功能的情况下，空转时长较大的那些键可能会有点被服务器删除。

<!-- more -->

## 对象的类型与编码
Redis使用对象来表示数据库中的键和值。其中键肯定是字符串对象。

Redis中的每个对象都由一个redisObject结构表示，该结构中和保存数据有关的三个属性跟别是type属性、encoding属性和ptr属性:
```c
typedef struct redisObject {
    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void *ptr;

    // ...
} robj;
```

### 类型
对象type属性记录了对象的类型，共吴忠，如下表所示，而TYPE命令的实现也与此类似，读取对象的type值即可知道对象的属性。

|对象|对象type属性的值|TYPE命令的输出|
|-|-|-|
|字符串对象|REDIS_STRING|"string"|
|列表对象|REDIS_LIST|"list"|
|哈希对象|REDIS_HASH|"hash"|
|集合对象|REDIS_SET|"set"|
|有序集合对象|REDIS_ZSET|"zset"|

### 编码和底层实现
对象的ptr指针指向对象的底层实现数据结构，而这些数据结构由对象的encoding属性决定。

encoding属性记录了对象所使用的编码，也即是说这个对象使用了什么数据结构作为对象的底层实现，这个属性的值是下表中的任意一个。

|编码常量|编码所对应的底层数据结构|使用该编码常量的对象类型|
|-|-|-|
|REDIS_ENCODING_INT|long类型的整数|REDIS_STRING|
|REDIS_ENCODING_EMBSTER|embstr编码的简单动态字符串REDIS_STRING||
|REDIS_ENCODING_RAW|简单动态字符串|REDIS_STRING|
|REDIS_ENCODING_HT|字典|REDIS_HASH, REDIS_SET|
|REDIS_ENCODING_LINKEDLIST|双端链表|REDIS_LIST|
|REDIS_ENCODING_ZIPLIST|压缩列表|REDIS_LIST, REDIS_HASH, REDIS_ZSET|
|REDIS_ENCODING_INTSET|整数集合|REDIS_SET|
|REDIS_ENCODING_SKIPLIST|跳跃表和字典|REDIS_ZSET|
使用OBJECT ENCODING命令可以查看一个数据库键的值对象的编码
```
redis> SET msg "hello world"
OK
redis> OBJECT ENCODING msg
"embstr"
```
通过encoding属性来设定对象所使用的编码，而不是为特定类型的对象关联一种固定的编码，极大地提升了Redis的灵活性和效率。一种对象，会因为使用场景的不同而使用不同的底层编码实现。

## 字符串对象
字符串对象的编码可以是int, raw或者embstr。

|值|编码|
|-|-|
|可以用long类型保存的整数|int|
|可以用long double类型保存的浮点数|embstr或者raw|
|字符串值或者因为长度太大而没办法用long类型表示的整数，又或者因为长度太大而没办法用long double类型表示的浮点数|embstr或者raw|

### 编码的转换
int编码如果执行了命令后，不再是整数值而是字符串值，编码将变为raw。

因为Redis没有为embstr编码的字符串对象编写任何相应的修改程序，所以embstr编码的字符串对象实际上是只读的。

## 列表对象
列表对象的编码可以是ziplist或者linkedlist。

### 编码转换
当列表对象可以同事满足以下两个条件时，列表对象使用ziplist编码：

- 列表对象保存的所有字符串元素的长度都小鱼64字节
- 列表对象保存的元素数量小于512个；不能满足这两个条件的列表对象需要使用linkedlist编码

## 哈希对象

哈希对象的编码可以是ziplist或者hashtable。

ziplist编码的哈希对象使用压缩列表作为底层实现，每当有新的键值对要加入到哈希对象时，程序会现将保存了键的压缩列表节点推入到压缩列表表尾，然后再将保存了值的压缩列表节点推入到压缩列表表尾，因此：

- 保存了同一个键值对的两个节点总是紧挨在一起，键在前，值在后
- 先添加到哈希对象中的键值对会被放在压缩列表的表头方向，而后来添加到哈希对象中的键值对会被放在压缩列表的表尾方向。

另一方面，hashtable编码的哈希对象使用字典作为底层实现，哈希对象中的每个键值对都使用一个字典键值对来保存。

### 编码转换

当哈希对象可以同时满足以下两个条件时，哈希对象使用ziplist编码：

- 哈希对象保存的所有键值对的键和值的紫都城长度都小鱼64字节
- 哈希对象保存的键值对数量小鱼512个；不能满足这两个条件的哈希对象需要使用hashtable编码。
