---
title: ZooKeeper基础
date: 2017-04-14 17:11:41
categories: ZooKeeper
tags: ZooKeeper
---

# 初识ZooKeeper
## ZooKeeper介绍
ZooKeeper是一个开放源代码的分布式协调服务，由雅虎创建，是Google Chubby的开源实现。ZooKeeper的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

## ZooKeeper是什么
ZooKeeper可以保证如下分布式一致性特性：
- 顺序一致性
- 原子性
- 单一视图
- 可靠性
- 实时性

## ZooKeeper的设计目标
ZooKeeper致力于提供一个高性能、高可用，且具有严格的顺序访问控制能力的分布式协调服务。
- 简单的数据模型
- 可以构建集群
- 顺序访问
- 高性能

## ZooKeeper的基本概念
### 集群角色
ZooKeeper没有沿用船用的slave/master概念，而是引入了Leader、Follower和Observer三种角色。ZooKeeper集群中的所有机器通过一个Leader选举过程来选定一台被称为“Leader”的机器，Leader服务器为客户端提供读和写服务。Follower和Observer都能提供读服务，唯一的区别在于Observer机器不参与Leader选举过程，也不参与写操作的“过半写成功”策略。

### 会话 Session
客户端和服务器之间建立一个TCP长连接，通过心跳检测与服务器保持有效的会话，也能够向ZooKeeper服务器发生请求并接受响应，通过来能够通过该连接接收来自服务器的Watch时间通知。

### 数据节点 Znode
指数据模型中的数据单元。ZooKeeper将所有数据村粗在内存中，数据模型是一棵树，由斜杠进行分隔的路径就是一个Znode，例如/foo/path1.每个Znode上都会保存自己的数据内容，同事还会保存一系列属性信息。

- 持久节点：需要主动进行Znode移除操作。
- 临时结点：它的生命周期和客户端会话绑定。

### 版本
每个Znode都会维护一个叫做Stat的数据结构，Stat中记录了这个Znode的三个数据版本，分别是version-当前Znode的版本，cversion-当前Znode子节点的版本，aversion-当前Znode的acl版本。

### Watcher
事件监听器。ZooKeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper服务端会将时间通知到感兴趣的客户端上去，该机制是ZooKeeper实现分布式协调服务的重要特性。

### ACL
access control lists来进行权限控制。ZooKeeper定义了如下5种权限：
- CREATE 创建子节点的权限
- READ 获取节点数据和子节点列表的权限
- WRITE 更新节点数据的权限0
- DELETE 删除子节点的权限
- ADMIN 设置节点ACL的权限

# ZooKeeper的ZAB协议
ZooKeeper Atomic Broadcast， ZooKeeper原子消息广播协议。ZooKeeper使用一个单一的主进程来接收并处理客户端的所有事务请求，并采用ZAB的原子广播协议，将服务器数据的状态变更以书屋Proposal的形式广播到所有的副本进程上去。

## 协议介绍
- 崩溃恢复
- 消息广播
