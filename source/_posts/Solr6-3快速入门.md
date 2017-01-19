---
title: Solr6.3快速入门
date: 2017-01-07 15:14:53
categories: Solr
tags: [Solr]
---
刚入门做Java，做Solr一个多月了，其实还是了解点皮毛，今天正好看了官方的pdf，从入门开始说起吧。

平时开发是在Windows环境下的，就说说Windows吧
<!-- more -->
# 一、环境搭建
1. 下载Solr引擎，直接去[官网](http://lucene.apache.org/solr/)，我这里用的是6.3版本的，贴个6.3版本的下载链接吧，[Solr6.3](https://mirrors.tuna.tsinghua.edu.cn/apache/lucene/solr/6.3.0/solr-6.3.0.zip)
2. 安装JDK，这个没什么好说的直接1.8，配置好对象的JAVA_HOME

# 二、安装启动Solr
下载解压缩之后，进入Solr的根目录，执行`#./bin/solr start -e cloud -noprompt `

Solr在2个节点上运行，一个端口是8983，另一个端口是7574。并自动建立了名称为gettingstarted的collection，此collection有2个shard，每个shard有2个replicas。
>shard相当于分片，将一个collection分成块，每块的内容不同
>replicas是副本，对每个分片内容的完整拷贝。

```
Welcome to the SolrCloud example!

Starting up 2 Solr nodes for your example SolrCloud cluster.

Creating Solr home directory D:\env\solr-6.3.0\example\cloud\node1\solr
Cloning D:\env\solr-6.3.0\example\cloud\node1 into
   D:\env\solr-6.3.0\example\cloud\node2

Starting up Solr on port 8983 using command:
D:\env\solr-6.3.0\bin\solr.cmd start -cloud -p 8983 -s "D:\env\solr-6.3.0\example\cloud\node1\solr"


Starting up Solr on port 7574 using command:
D:\env\solr-6.3.0\bin\solr.cmd start -cloud -p 7574 -s "D:\env\solr-6.3.0\example\cloud\node2\solr" -z localhost:9983


Connecting to ZooKeeper at localhost:9983 ...
Uploading D:\env\solr-6.3.0\server\solr\configsets\data_driven_schema_configs\conf for config gettingstarted to ZooKeeper at localhost:9983

Creating new collection 'gettingstarted' using command:
http://localhost:8983/solr/admin/collections?action=CREATE&name=gettingstarted&numShards=2&replicationFactor=2&maxShardsPerNode=2&collection.configName=gettingstarted

{
  "responseHeader":{
    "status":0,
    "QTime":5560},
  "success":{
    "10.1.64.31:8983_solr":{
      "responseHeader":{
        "status":0,
        "QTime":3983},
      "core":"gettingstarted_shard2_replica1"},
    "10.1.64.31:7574_solr":{
      "responseHeader":{
        "status":0,
        "QTime":4296},
      "core":"gettingstarted_shard2_replica2"}}}

Enabling auto soft-commits with maxTime 3 secs using the Config API

POSTing request to Config API: http://localhost:8983/solr/gettingstarted/config
{"set-property":{"updateHandler.autoSoftCommit.maxTime":"3000"}}
Successfully set-property updateHandler.autoSoftCommit.maxTime to 3000


SolrCloud example running, please visit: http://localhost:8983/solr
```
我启动的是solrcloud模式。Solr在2个节点上运行，一个端口是8983，另一个端口是7574。并自动建立了名称为gettingstarted的collection，此collection有2个shard，每个shard有replicas。 好了，直接访问上面的链接，就可以看到它自动为你新建了一个Collection名为gettingstarted。

# 三、添加索引
由于windows自带的cmd和power shell不支持shell脚本，那么就用git bash吧。安装好git之后，打开git bash，进入Solr的根目录，执行

```
$ bin/post -c gettingstarted example/exampledocs/*.xml
```
用SimplePostTool 为example/exampledocs目录下的所有xml文件建立索引。exampledocs目录下还有一些其他格式的文件

为JSON文件建立索引
```
$ bin/post -c gettingstarted example/exampledocs/*.json
```
为CSV文件建立索引
```
$ bin/post -c gettingstarted example/exampledocs/books.csv
```
为solr安装目录下的docs文件夹中的文件建立索引。 
```
$ bin/post -c gettingstarted docs/
```

# 四、搜索
打开[搜索链接](http://localhost:8983/solr/#/gettingstarted/query)，在这里面就可以搜索你想要的信息啦。



# 五、简便方法
还有一种启用Solr单机模式的方法，并且自动帮你导入数据
```
$ bin/solr.cmd start -e techproducts
```
```
Creating Solr home directory D:\env\solr-6.3.0\example\techproducts\solr

Starting up Solr on port 8983 using command:
D:\env\solr-6.3.0\bin\solr.cmd start -p 8983 -s "D:\env\solr-6.3.0\example\techproducts\solr"

Waiting up to 30 to see Solr running on port 8983
Started Solr server on port 8983. Happy searching!

Copying configuration to new core instance directory:
D:\env\solr-6.3.0\example\techproducts\solr\techproducts

Creating new core 'techproducts' using command:
http://localhost:8983/solr/admin/cores?action=CREATE&name=techproducts&instanceDir=techproducts

{
  "responseHeader":{
    "status":0,
    "QTime":1396},
  "core":"techproducts"}


Indexing tech product example docs from D:\env\solr-6.3.0\example\exampledocs
SimplePostTool version 5.0.0
Posting files to [base] url http://localhost:8983/solr/techproducts/update using content-type application/xml...
POSTing file gb18030-example.xml to [base]
POSTing file hd.xml to [base]
POSTing file ipod_other.xml to [base]
POSTing file ipod_video.xml to [base]
POSTing file manufacturers.xml to [base]
POSTing file mem.xml to [base]
POSTing file money.xml to [base]
POSTing file monitor.xml to [base]
POSTing file monitor2.xml to [base]
POSTing file mp500.xml to [base]
POSTing file sd500.xml to [base]
POSTing file solr.xml to [base]
POSTing file utf8-example.xml to [base]
POSTing file vidcard.xml to [base]
14 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/techproducts/update...
Time spent: 0:00:00.366

Solr techproducts example launched successfully. Direct your Web browser to http://localhost:8983/solr to visit the Solr Admin UI

D:\env\solr-6.3.0\bin>solr.cmd stop -all
Stopping Solr process 9768 running on port 8983

等待 4 秒，按一个键继续 ...

D:\env\solr-6.3.0\bin>solr.cmd start -e techproducts -f
Solr home directory D:\env\solr-6.3.0\example\techproducts\solr already exists.

Starting up Solr on port 8983 using command:
D:\env\solr-6.3.0\bin\solr.cmd start -p 8983 -s "D:\env\solr-6.3.0\example\techproducts\solr"

Archiving 1 old GC log files to D:\env\solr-6.3.0\example\techproducts\solr\..\logs\archived
Archiving 1 console log files to D:\env\solr-6.3.0\example\techproducts\solr\..\logs\archived
Rotating solr logs, keeping a max of 9 generations

WARNING: Core 'techproducts' already exists!
Checked core existence using Core API command:
http://localhost:8983/solr/admin/cores?action=STATUS&core=techproducts


Solr techproducts example launched successfully. Direct your Web browser to http://localhost:8983/solr to visit the Solr Admin UI
```
就是这么简单
