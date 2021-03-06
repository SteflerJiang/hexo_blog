---
title: Redis设计与实现-第2章-简单动态字符串
date: 2016-12-19 19:57:23
categories: Redis
tags: [Redis, 数据结构]
---
Redis没有直接使用C语言传统的字符串表示，而是自己构建了一种名为简单动态字符串(simple dynamic string, SDS)的抽象类型，并将SDS用作Redis的默认字符串表示。
```shell
redis> SET msg "hello world"
OK
```
<!--more-->

## SDS的定义
每个shs.h/sdshr结构表示一个SDS值：
```c
struct sdshdr {
    // 记录buf数组中已使用字节的数量
    // 等于SDS所保存字符串的长度
    int len;

    // 记录buf数组中未使用字节的数量
    int free;

    // 字节数组，用于保存字符串，二进制安全
    char buf[];
} sdshr;
```
保存空字符的1字节空间不计算在SDS的len属性里面，并且**自动**为空字符串分配额外的1字节空间，添加空字符串到字符串末尾。这样可以重用一部分C字符串函数库里的函数。

## SDS与C字符串的区别
1. 常数复杂度获取字符串长度
2. 杜绝缓冲区溢出
3. 减少修改字符串时带来的内存重分配次数

## 空间预分配策略
1. 修改后SDS长度小于1MB，则分配相同的未使用和已使用空间。
2. 修改后长度大于1MB，分配1MB的未使用空间。

## 惰性空间释放
用于优化SDS的字符串缩短操作，当SDS的API需要缩短SDS保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。

## 总结
|C字符串|SDS|
|-|-|
|获取字符串长度的复杂度为O(N)|获取字符串长度的复杂度为O(1)|
|API是不安全的，可能会造成缓冲区溢出|API是安全的，不会造成缓冲区溢出|
|修改字符串长度N次必然需要执行N此内存重分配|修改字符串长度N次最多需要执行N次内存重分配|
|只能保存文本数据|可以保存文本或者二进制数据|
|可以使用所有<string.h>库中的函数|只能使用部分函数|
