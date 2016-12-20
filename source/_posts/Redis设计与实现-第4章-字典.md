---
title: Redis设计与实现-第4章-字典
date: 2016-12-20 18:32:00
categories: Redis
tags: [Redis, 数据结构]
---
## 字典的实现
Redis字典所使用的哈希表由dict.h/dictht结构定义：
```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小，即table数组的大小
    unsigned long size;
    // 哈希表大小掩码
    // 总是等于size - 1
    unsigned long sizemask;
    // 该哈希表已有的节点数量
    unsigned long used;
} dictht;
```

哈希表节点使用dictEntry结构表示，每个dictEntry结构都保存着一个键值对
```c
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```
- key属性保存着键值对中的键，
- 而v属性则保存着键值对中的值，其中键值对的值可以是一个指针，或者是一个unit64_t证书，又或者是一个int64_t整数。
- next指向另一个哈希表节点，可以将多个哈希值相同的键值对连接在一起。

Redis中的字典由dict.h/dict结构表示：
```c
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash索引
    // 当rehash不在进行时，值为-1
    int trehashidx;
} dict
```
- type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函数。
- private属性保存了需要传给那些类型特定函数的可选参数。
- ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht[0]哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用。
- rehashidx记录了rehash的进度，-1表示没有进行rehash。

## 哈希算法
Redis计算哈希值和索引值的方法如下：
```c
// 计算键key的哈希值
hash = dict->type->hashFunction(key);
// 计算出索引值
index = hash & dict->ht[x].sizemask;
```
当字典被用作数据库的底层实现，或者哈希键的底层实现时，Redis使用MurmurHash2算法来计算键的哈希值。

## 解决键冲突
Redis使用链地址法，程序总是将新节点添加到链表的表头位置（这点和Java中的HashMap也很像）。

## rehash
步骤如下：
1. 为字典的ht[1]哈希表分配空间，如果是扩展操作，ht[1]大小为第一个大于等于ht[0].used*2的2^n；如果是收缩操作，ht[1]大小为第一个大于等于ht[0].used的2^n。
2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面
3. 释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]新创建一个空白哈希表。

### 哈希表的扩展与收缩
满足下列任意一个条件时，自动开始扩展操作
1. 服务器目前没有在执行BGSAVE或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于1。
2. 服务器目前正在执行BGSAVE或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于5。

`负载因子load_factor = ht[0].used / ht[0].size`

另一方面，当哈希表的负载因子小于0.1时，自动执行收缩操作。

## 渐进式rehash
为了避免rehash对服务器性能造成影响，当服务器里键值对过多时，详细步骤如下
1. 为ht[1]分配空间
2. 维持一个索引计数器变量rehashidx，并设为0，表示rehash工作正式开始
3. 在rehash进行期间，每次对字典进行增删改查操作时，会将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，rehash完成后，rehashidx++;
4. 随着字典不断执行，最终ht[0]的所有键值对都会被rehash到ht[1]，这时设置rehashidx为-1，表示rehash操作已完成。

### 渐进式rehash执行期间的哈希表操作
- 删除 查找 更新会在两个哈希表上进行
- 添加操作一律在ht[1]执行
