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

ZAB协议的核心是定义了对于那些会改变ZooKeeper服务器数据状态的事务请求的处理方式，即：
> 所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而余下的其他服务器则称为Follower服务器。Leader服务器负责将一个客户端事务请求转换成一个事务Proposal(提议)，并将该Proposal分发给集群中所有的Follower服务器。之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确的反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交。

## 协议介绍
### 消息广播
针对客户端的事务请求，Leader服务器会为其生成对应的事务Proposal，并将其发送给集群中其余所有的机器，然后再分别收集各自的选票，最后进行事务提交。

### 崩溃恢复
核心：能够确保提交已经被Leader提交的事务Proposal，同时丢弃已经被跳过的事务Proposal。

### 算法描述
- 发现 discovery 选取最大的epoch
- 同步 synchronization prepare commit
- 广播 broadcast

# Zookeeper安装
在windows上部署伪集群模式的链接，注意需要给每个机器的data目录下添加一个myid文件。
http://blog.csdn.net/morning99/article/details/40426133

启动zkCli.cmd -server ip:host

# Java客户端API使用
## 创建会话
Zookeeper zk = new Zookeeper();

节点类型共有四种
- 持久 PERSITENT
- 持久顺序 PERSITENT_SEQUENTIAL
- 临时 EPHEMERAL
- 临时顺序 EPHEMERAL

## 创建节点
zkClient.create()

Zookeeper不支持递归创建，即无法在父节点不存在的情况下创建一个子节点。

## 删除节点
zkClient.delete()

在Zookeeper中，只允许删除叶子节点。也就是说如果一个节点存在至少一个子节点的话，那么该节点将无法被直接删除，必须先删除掉其所有子节点。

## 读取数据
zkClient.getChildren()

调用getChildren()获取到的节点列表，都是数据节点的相对节点路径。

Zookeeper服务端在向客户端发送Watcher"NodeChildrenChanged"事件通知的时候，仅仅只会发出一个通知，而不会把节点的变化情况发送给客户端，需要客户端自己重新获取。另外，由于Watcher通知是一次性的，即一旦触发一次通知后，该Watcher就失效了，因此客户端需要反复注册Watcher。

zkClient.getData()

节点的数据内容或是节点的数据版本编号，都被看作是Zookeeper节点的编号。

## 更新数据
setData(final String path, byte date[], int version)

CAS理论，对于值V，每次更新前都会比对其值是否是预期值A，只有符合预期，才会将V原子化地更新到新值B。

如果传入的version为-1，就是告诉Zookeeper服务器，客户端需要基于数据的最新版本进行更新操作，没有原子性要求。

## 检测节点是否存在
zkClient.exists()

- 无论指定节点是否存在，通过调用exists接口都可以注册Watcher。
- exists接口中注册的Watcher，能够对节点创建、节点删除和节点数据更新事件进行监听
- 对于指定节点的子节点的各种变化，都不会通知客户端。

## 权限控制
当客户端对一个数据节点添加了权限信息后，对于删除操作而言，其作用范围是其子节点。也就是说，当我们对一个数据节点添加权限信息后，依然可以自由地删除这个节点，但是对于这个节点的子节点，就必须使用相应的权限信息才能够删除掉它。
