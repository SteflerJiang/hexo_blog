---
title: Solr中Nested Doduments拓展
date: 2016-12-16 19:35:45
categories: Java
tags: [Java, Solr]
---
# 一、介绍
[Nested Documents](http://yonik.com/solr-nested-objects/)是Solr引擎提供的一种用来描述父子关系的一种Document，描述的是一种一对多的关系。举个例子，一篇博客和很多评论，那么该博客就可以作为父doc的形式存在，而每条评论就可以作为子doc包含在父doc中。
<!-- more -->
一个简单的用XML形式描述的Nested Documents如下所示:
``` xml
<?xml version="1.0" encoding="utf-8"?>
<add>
  <doc>
    <field name="id">1</field> 
    <field name="title">Solr adds block join support</field> 
    <field name="content_type">parentDocument</field> 
    <doc>
      <field name="id">2</field> 
      <field name="comments">SolrCloud supports it too!</field>
    </doc>
  </doc> 
  <doc>
    <field name="id">3</field> 
    <field name="title">New Lucene and Solr release is out</field> 
    <field name="content_type">parentDocument</field> 
    <doc>
      <field name="id">4</field> 
      <field name="comments">Lots of new features</field>
    </doc>
  </doc>
</add>
```
在这个例子中，有两个父doc，它们都有`content_type=parentDocument`，通过这个字段就可以区分出父子doc。

而为了支持这样的形式，Solr中的schema必须包含一个**indexed/non-stored**的字段`_root_`。也就是说_root_这个字段是不需要手动去配置它的值，在Solr内部当以父子关系加入进索引的时候，就会保持该block具有相同的_root_值，并且它的自增的。当然了子doc也可以作为父doc再去包含子doc，不管继承的深度，同一系的父子doc总是作为一个单独的block存在的，并且一个block中所有的doc都具有相同的`_root_`字段。

# 问题
目前有两个问题需要解决
- 如何在只更新父doc的时候同时删除子doc
- 如何在查询子doc的时候同时附带相应的父doc的信息

由于Solr中数据来源是hive，由于自己实现了Build Document这个步骤，所以必须满足Solr中对于父子doc的约束条件。
- Schema中配置了一个indexed/non-stored `index`字段；
- 同一个Block中的所有Documents都拥有相同的`_root_`字段；
- 对于一组父子doc，其存储的位置关系必须是连续的，而且是子doc在前，父doc在后。也即它们的docid必须的连续的，而且父doc的id最大。

## 如何在只更新父doc的时候同时删除子doc
在Solr提供的`DirectUpdateHandler2`中，我们看到了一个方法`doNormalUpdate(AddUpdateCommand cmd)`其中是这样更新doc的
``` java
Term idTerm = new Term(cmd.isBlock() ? "_root_" : idField.getName(), cmd.getIndexedId());

if (cmd.isBlock()) {
    writer.updateDocuments(updateTerm, cmd);
} else {
    Document luceneDocument = cmd.getLuceneDocument();
    // SolrCore.verbose("updateDocument",updateTerm,luceneDocument,writer);
    writer.updateDocument(updateTerm, luceneDocument);
}
```

`isBlock()`方法用来判断是否有子节点，所以这里idTerm的key就是，有子节点为`_root_`，无子节点为`id`。据此可以判断，单独更新父doc时，取`id`来做update操作，所以相同`id`的会被更新，也就是父doc被更新。而在更新一个block时，取`_root_`来做update操作，所以相同`_root_`的节点会被更新掉。这里Solr显然不提供一种删除所有子doc的方法。

那么我们要做的就是继承这个类，在它的doNormalUpdate()方法中，只取_root_来做update操作。不过这也要求，我们更新的这个父doc必须包含_root_字段，不然是无法判断，你到底是需要更新一条记录还是需要更新一个block。

做完之后需要在solrconfig.xml中更改原有的配置，添加一项
``` xml
<updateHandler class="org.apache.solr.MyDirectUpdateHandler2">
<!-- *** -->
</updateHandler>  
```

## 如何在查询子doc的时候同时附带相应的父doc的信息
有了这样的Nested Documents，就能去做需要的查询。Solr提供了这样一种[Block Join Query Parsers](https://cwiki.apache.org/confluence/display/solr/Other+Parsers#OtherParsers-BlockJoinQueryParsers)的查询方式，其中包括Block Join Children Query Parser和Block Join Parent Query Parser，但是这种查询方式只能支持查询父doc或者是子doc，无法满足我们需要的子doc中包含父doc的某些信息。

Solr还提供一种Transforming Result Documents，这种transformers可以用来修改返回出的doc信息，包括增加某些固定的字段(ValueAugmenterFactory)，对于父doc来说返回其所有的父子doc(ChildDocTransformerFactory)等等。但是没有我们想要的方式，不过可以在solrconfig.xml中通过配置transformer的方式，来应用自己的transformer。
添加的配置信息如下：
``` xml
<transformer name="parent" class="org.apache.solr.MyDocTransformerFactory"></transformer>
```
这也的话，在使用Solr进行查询时，只需要在field中加入`[parent parentFilter=type:t extraFields=discount_amount]`，来告知需要使用到`MyDocTransformerFactory`和其中的一些参数。`parentFilter`用来过滤子doc，只剩下父doc；`extraFields`用来表明需要额外添加的父doc的字段名。

在`ParentDocTransformer`中，其实也是根据上面约束条件三的父子doc的位置关系，找到子doc后面的_root_值相同的第一个父doc。这里查询父doc，是使用Solr提供的`org.apache.lucene.search.join.ToParentBlockJoinQuery`。当然了，对于比较少的内容，也可以自己写一个query去做查询，只需要找到所有和子doc的_root_值相同的docs，并筛选出其中的父doc即可。相对而言，还是第一种效率更高，不过在数量级不多的情况下，第二种也是可以接受的。
