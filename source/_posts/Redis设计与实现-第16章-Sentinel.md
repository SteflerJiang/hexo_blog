---
title: Redis设计与实现-第16章-Sentinel
date: 2017-06-03 11:34:12
categories: Redis
tags:
---
Sentinel是Redis的高可用性解决方案：由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器以及这些主服务器属下的所有从服务器，并在被监视的主服务器进去下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求。
<!-- more -->

## 启动并初始化Sentinel
```
$ redis-sentinel /path/to/your/sentinel.conf
```
当一个Sentinel启动时，它需要执行以下步骤：
1. 初始化服务器
2. 将普通Redis服务器使用的代码替换成Sentinel专用代码
3. 初始化Sentinel状态
4. 根据给定的配置问加你，初始化Sentinel的监视主服务器列表
5. 创建连向主服务器的网络连接

### 初始化服务器
跟启动一个Redis服务器差不多，但是并不完全相同、

### 使用Sentinel专用代码
比如默认端口号不一样，载入的服务器命令表也不一样

### 初始化Sentinel状态
在应用了Sentinel的专用代码之后，接下来，服务器会初始化一个sentinel.c/sentinelState结构，这个结构保存了服务器中所有和Sentinel功能有关的状态。

``` c
struct sentinelState {
    // 当前纪元，用于实现故障转移
    unit64_t current_epoch;

    // 保存了所有被这个sentinel监视的主服务器
    // 字典的键是主服务器的名字
    // 字典的值则是一个指向sentinelRedisInstance结构的指针
    dict *masters;

    // 是否进入了TILT模式
    int tilt;

    // 目前正在执行的脚本的数量
    int running_scripts;

    // 进入TILT模式的时间
    mstime_t tilt_start_time;

    // 最后一次执行时间处理器的时间
    mstime_t previous_time;

    // 一个FIFO队列，包含了所有需要执行的用户脚本
    list *script_queue;
} sentinel;
```

### 初始化Sentinel状态的masters属性
每个sentinelRedisInstance结构代表一个被Sentinel监视的Redis服务器实例，这个实例可以是主服务器，从服务器或者另外一个Sentinel。

``` c
struct sentinelRedisInstance {
    // 标识值，记录了实例的类型，以及该实例的当前状态
    int flags;

    // 实例的名字
    // 主服务器的名字由用户在配置文件中设置
    // 从服务器以及Sentinel的名字由Sentinel自动设置
    // 格式为ip:port
    char *name;

    // 实例的运行ID
    char *runid;

    // 配置纪元，用于实现故障转移
    unit64_t config_epoch;

    // 实例的地址
    sentinelAddr *addr;

    // 实例无响应多少毫秒之后才会被判断为主观下线
    mstime_t down_after_period;

    // 判断这个实例为客观下线所需的支持投票数量
    int quorum;

    // 在执行故障转移操作时，可以同事对行的主服务器进行同步的从服务器数量
    int parallel_syncs;

    // 刷新故障迁移状态的最大时限
    mstime_t failover_timeout;

    // 一个主服务器所有从服务器实例
    dict *slaves;

    // 监视这个主服务器的所有sentinels实例
    dict *sentinels;

    // ...
} sentinelRedisInstance;
```

### 创建连向主服务器的网络连接
对于每个被Sentilen监视的主服务器来说，Sentinel会创建两个连向主服务器的异步网络连接
- 一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复
- 另一个是订阅连接，这个连接专门用于订阅主服务器的__sentinel__:hello频道，主要用来感知监视服务器的其他Sentinel。

## 获取主服务器信息
Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息。

通过分析主服务器返回的INFO命令回复，Sentinel可以回去以下两方面的信息：
- 一方面是关于主服务器本身的信息，包括run_id域记录的服务器运行ID，以及role域记录的服务器角色。
- 另一方面是关于主服务器属下所有从服务器的信息，每个从服务器都由一个"slave"字符串开头的行记录，每行的ip域记录了从服务器的IP地址，而port域则记录了从服务器的端口号。

## 获取从服务器的信息
在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送INFO命令。

从服务器的实例保存在主服务器实例结构的slaves字典里面。

## 向主服务器和从服务器发送信息
在默认情况下，Sentinel会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送hello频道的命令。

## 接收来自主服务器和从服务器的频道信息
当Sentinel与一个主服务器或者从服务器建立起订阅连接之后，Sentinel就会通过订阅连接，向服务器发送以下命令`SUBSCRIBE __sentilen__:hello`。

对于每个与Sentinel连接的服务器，Sentinel既通过命令连接向服务器的hello频道发送信息，又通过订阅连接从服务器的hello频道接收信息，因此监视同一个服务器的Sentinel可以获知其他Sentinel。

### 更新sentinels字典
### 创建连向其他Sentinel的命令连接
互相创建命令连接

## 检测主观下线状态
在默认情况下，Sentinel会以每秒一次的频率向所有与它创建了命令连接的实例（包括主服务器，从服务器，其他Sentinel在内）发送PING命令，并通过实例返回的PING命令来判断实例是否在线。

如果在down-after-milliseconds毫秒内，连续向Sentinel返回无效回复时，那么Sentinel会修改这个实例所对应的实例结构，在结构的flags属性中打开SRI_S_DOWN标识，以此来标识这个实例已经进入主观下线状态。

主观下线时长选项的作用范围为master和它对象的slaves，sentinels。

多个Sentinel设置的主观下线时长可能不同。

## 检查客观下线状态
当Sentinel将一个主服务器判断为主观。下线之后，为了确认这个主服务器是否真的下线了，它会向同样监视这一主服务器的其他Sentinel进行询问，看它们是否也认为主服务器已经进入了下线状态。当Sentinel从其他Sentinel那里接收到足够数量的已下线判断之后，Sentinel就会将从服务器判定为客观下线，并对主服务器执行故障转移操作。

## 选举领头Sentinel
当一个主服务器被判断为客观下线时，监视这个下线主服务器的各个Sentinel会进行协商，选举出一个领头Sentinel，并由领头Sentinel对下线主服务器进行故障转移操作。

以下是Redis选举领头Sentinel的规则和方法
- 所有在线的Sentinel都有被选为领头Sentinel的资格。
- 每次进行领头Sentinel选举之后，不论选举是否成功们所有Sentinel的配置纪元的值都会自增一次。
- 在一个配置纪元里面，所有Sentinel都有一次将某个Sentinel设置为局部领头Sentinel的集合，并且局部领头一旦设置，在这个配置纪元里面就不能再更改。
- 每个发现主服务器进入客观下线的Sentinel都会要求其他Sentinel将自己设置为局部领头Sentinel。
- 。。。

## 故障转移
在选举产生出领头Sentinel之后，领头Sentinel将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤：
1. 在已下线的主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转换为主服务器。
2. 让已下线的主服务器属下的所有从服务器改为复制新的主服务器
3. 将已下线的主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器。
