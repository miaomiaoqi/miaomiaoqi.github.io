---
layout: post
title: ElasticSearch 搜索原理
categories: [BigData]
description: 
keywords: 
---

* content
{:toc}


## 图解 ElasticSearch

### 云上的集群

![https://miaomiaoqi.github.io/images/elastic/search/search/search_1.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_1.png)

### 集群里的盒子

云里面的每个白色正方形的盒子代表一个节点——Node. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_2.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_2.png)

### 节点之间

在一个或者多个节点直接, 多个绿色小方块组合在一起形成一个 ElasticSearch 的索引. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_3.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_3.png)

### 索引里的小方块

在一个索引下, 分布在多个节点里的绿色小方块称为分片——Shard. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_4.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_4.png)

### Shard＝Lucene Index

一个 ElasticSearch 的 Shard 本质上是一个 Lucene Index. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_5.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_5.png)

Lucene 是一个 Full Text 搜索库(也有很多其他形式的搜索库), ElasticSearch 是建立在 Lucene 之上的. 接下来的故事要说的大部分内容实际上是ElasticSearch 如何基于 Lucene 工作的. 



## 图解 Lucene

### Mini 索引——segment

在Lucene里面有很多小的segment, 我们可以把它们看成Lucene内部的mini-index. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_6.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_6.png)

### Segment 内部

有着许多数据结构

-   Inverted Index
-   Stored Fields
-   Document Values
-   Cache

![https://miaomiaoqi.github.io/images/elastic/search/search/search_7.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_7.png)

### 最最重要的 Inverted Index

![https://miaomiaoqi.github.io/images/elastic/search/search/search_8.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_8.png)

Inverted Index主要包括两部分：

1.  一个有序的数据字典 Dictionary(包括单词Term和它出现的频率). 
2.  与单词 Term 对应的 Postings(即存在这个单词的文件). 

当我们搜索的时候, 首先将搜索的内容分解, 然后在字典里找到对应 Term, 从而查找到与搜索相关的文件内容. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_9.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_9.png)

#### 查询“the fury”

![https://miaomiaoqi.github.io/images/elastic/search/search/search_10.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_10.png)

#### 自动补全(AutoCompletion-Prefix)

如果想要查找以字母“c”开头的字母, 可以简单的通过二分查找(Binary Search)在 Inverted Index 表中找到例如“choice”、“coming”这样的词(Term). 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_11.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_11.png)

#### 昂贵的查找

如果想要查找所有包含“our”字母的单词, 那么系统会扫描整个 Inverted Index, 这是非常昂贵的. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_12.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_12.png)

在此种情况下, 如果想要做优化, 那么我们面对的问题是如何生成合适的 Term. 

#### 问题的转化

![https://miaomiaoqi.github.io/images/elastic/search/search/search_13.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_13.png)

对于以上诸如此类的问题, 我们可能会有几种可行的解决方案：

-   \* suffix -> xiffus *

    如果我们想以后缀作为搜索条件, 可以为 Term 做反向处理. 

-   (60.6384, 6.5017) -> u4u8gyykk

    对于 GEO 位置信息, 可以将它转换为 GEO Hash. 

-   123 -> {1-hundreds, 12-tens, 123}

    对于简单的数字, 可以为它生成多重形式的 Term. 



#### 解决拼写错误

一个 Python 库为单词生成了一个包含错误拼写信息的树形状态机, 解决拼写错误的问题. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_14.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_14.png)



### Stored Field 字段查找

当我们想要查找包含某个特定标题内容的文件时, Inverted Index 就不能很好的解决这个问题, 所以 Lucene 提供了另外一种数据结构 Stored Fields 来解决这个问题. 本质上, Stored Fields 是一个简单的键值对 key-value. 默认情况下, ElasticSearch 会存储整个文件的 JSON source. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_15.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_15.png)



### Document Values 为了排序, 聚合

即使这样, 我们发现以上结构仍然无法解决诸如：排序、聚合、facet, 因为我们可能会要读取大量不需要的信息. 

所以, 另一种数据结构解决了此种问题：Document Values. 这种结构本质上就是一个列式的存储, 它高度优化了具有相同类型的数据的存储结构. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_16.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_16.png)



## 搜索发生时

搜索时, Lucene 会搜索所有的 segment 然后将每个 segment 的搜索结果返回, 最后合并呈现给客户. 

Lucene的一些特性使得这个过程非常重要：

-   Segments是不可变的(immutable)

-   -   **Delete?** 当删除发生时, Lucene做的只是将其标志位置为删除, 但是文件还是会在它原来的地方, 不会发生改变
    -   **Update?** 所以对于更新来说, 本质上它做的工作是：先**删除**, 然后**重新索引(Re-index)**

-   随处可见的压缩

    Lucene非常擅长压缩数据, 基本上所有教科书上的压缩方式, 都能在Lucene中找到. 

-   缓存所有的所有

    Lucene也会将所有的信息做缓存, 这大大提高了它的查询效率. 



## 缓存的故事

当 ElasticSearch 索引一个文件的时候, 会为文件建立相应的缓存, 并且会定期(每秒)刷新这些数据, 然后这些文件就可以被搜索到. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_17.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_17.png)

随着时间的增加, 我们会有很多 segments, 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_18.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_18.png)

所以 ElasticSearch 会将这些 segment 合并, 在这个过程中, segment 会最终被删除掉

![https://miaomiaoqi.github.io/images/elastic/search/search/search_19.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_19.png)

这就是为什么增加文件可能会使索引所占空间变小, 它会引起 merge, 从而可能会有更多的压缩. 



### 举个栗子

有两个 segment 将会 merge

![https://miaomiaoqi.github.io/images/elastic/search/search/search_20.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_20.png)

这两个 segment 最终会被删除, 然后合并成一个新的 segment

![https://miaomiaoqi.github.io/images/elastic/search/search/search_21.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_21.png)

这时这个新的 segment 在缓存中处于cold状态, 但是大多数 segment 仍然保持不变, 处于 warm 状态. 

以上场景经常在 Lucene Index 内部发生的. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_22.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_22.png)

## 在 Shard 中搜索

ElasticSearch 从 Shard 中搜索的过程与 Lucene Segment 中搜索的过程类似. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_23.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_23.png)

与在 Lucene Segment 中搜索不同的是, Shard 可能是分布在不同Node上的, 所以在搜索与返回结果时, 所有的信息都会通过网络传输. 

需要注意的是：

1 次搜索查找 2 个shard ＝ 2 次分别搜索 shard

![https://miaomiaoqi.github.io/images/elastic/search/search/search_24.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_24.png)



### 对于日志文件的处理

当我们想搜索特定日期产生的日志时, 通过根据时间戳对日志文件进行分块与索引, 会极大提高搜索效率. 

当我们想要删除旧的数据时也非常方便, 只需删除老的索引即可. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_25.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_25.png)

在上种情况下, 每个 index 有两个 shards



## 如何 Scale

![https://miaomiaoqi.github.io/images/elastic/search/search/search_26.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_26.png)

shard 不会进行更进一步的拆分, 但是shard可能会被转移到不同节点上

![https://miaomiaoqi.github.io/images/elastic/search/search/search_27.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_27.png)

所以, 如果当集群节点压力增长到一定的程度, 我们可能会考虑增加新的节点, 这就会要求我们对所有数据进行重新索引, 这是我们不太希望看到的, 所以我们需要在规划的时候就考虑清楚, 如何去平衡足够多的节点与不足节点之间的关系. 



### 节点分配与 Shard 优化

-   为更重要的数据索引节点, 分配性能更好的机器
-   确保每个 shard 都有副本信息 replica

![https://miaomiaoqi.github.io/images/elastic/search/search/search_28.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_28.png)

### 路由 Routing

每个节点, 每个都存留一份路由表, 所以当请求到任何一个节点时, ElasticSearch 都有能力将请求转发到期望节点的 shard 进一步处理. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_29.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_29.png)

## 一个真实的请求

![https://miaomiaoqi.github.io/images/elastic/search/search/search_30.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_30.png)

### Query

![https://miaomiaoqi.github.io/images/elastic/search/search/search_31.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_31.png)

Query 有一个类型 filtered, 以及一个 multi_match 的查询

### Aggregation

![https://miaomiaoqi.github.io/images/elastic/search/search/search_32.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_32.png)

根据作者进行聚合, 得到 top10 的 hits 的 top10 作者的信息



### 请求分发

这个请求可能被分发到集群里的任意一个节点

![https://miaomiaoqi.github.io/images/elastic/search/search/search_33.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_33.png)

### 上帝节点

![https://miaomiaoqi.github.io/images/elastic/search/search/search_34.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_34.png)

这时这个节点就成为当前请求的协调者(Coordinator), 它决定：

-   根据索引信息, 判断请求会被路由到哪个核心节点
-   以及哪个副本是可用的
-   等等



### 路由

![https://miaomiaoqi.github.io/images/elastic/search/search/search_35.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_35.png)

### 在真实搜索之前

ElasticSearch 会将 Query 转换成 Lucene Query

![https://miaomiaoqi.github.io/images/elastic/search/search/search_36.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_36.png)

然后在所有的segment中执行计算

![https://miaomiaoqi.github.io/images/elastic/search/search/search_37.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_37.png)

对于 Filter 条件本身也会有缓存

![https://miaomiaoqi.github.io/images/elastic/search/search/search_38.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_38.png)

但 queries 不会被缓存, 所以如果相同的 Query 重复执行, 应用程序自己需要做缓存

![https://miaomiaoqi.github.io/images/elastic/search/search/search_39.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_39.png)

所以, 

-   filters 可以在任何时候使用
-   query 只有在需要 score 的时候才使用

### 返回

搜索结束之后, 结果会沿着下行的路径向上逐层返回. 

![https://miaomiaoqi.github.io/images/elastic/search/search/search_40.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_40.png)

![https://miaomiaoqi.github.io/images/elastic/search/search/search_41.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_41.png)

![https://miaomiaoqi.github.io/images/elastic/search/search/search_42.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_42.png)

![https://miaomiaoqi.github.io/images/elastic/search/search/search_43.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_43.png)

![https://miaomiaoqi.github.io/images/elastic/search/search/search_44.png](https://miaomiaoqi.github.io/images/elastic/search/search/search_44.png)