---
layout: post
title:  "ELK-ElasticSearch"
date:   2019-08-05 13:54:25
categories: BigData
tags: ElasticSearch
author: miaoqi
---

* content
{:toc}     
## ElasticSearch

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTfulweb接口。ElasticSearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。构建在全文检索开源软件Lucene之上的Elasticsearch，不仅能对海量规模的数据完成分布式索引与检索，还能提供数据聚合分析。据国际权威的数据库产品评测机构DBEngines的统计，在2016年1月，Elasticsearch已超过Solr等，成为排名第一的搜索引擎类应用

## 安装

我的是 mac, 使用命令 `brew install elasticsearch` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/elasticsearch/6.8.2` 配置文件目录为`/usr/local/etc/elasticsearch`

使用`brew services start elasticsearch` 可以将 es 作为服务启动, 我在启动项目的时候遇到了两个问题

* **Cannot open file logs/gc.log due to No such file or directory**

    这个问题是因为日志文件的路径有问题, 可以修改配置文件 `jvm.options` 配置文件, 指定 `8:-Xloggc:/Users/miaoqi/Documents/elasticsearch/logs/gc.log` 日志文件的路径

* **Plugin [analysis-ik] was built for Elasticsearch version 6.7.2 but version 6.8.2 is running**

    这个问题是因为安装了 IKAnalysis 插件, 但是插件的版本与 es 的版本不对应

    可以使用 `elasticsearch-plugin list` 查看已经安装的插件

    然后使用 `elasticsearch-plugin remove plugin-name` 删除已有的插件

    最后使用 `elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.2/elasticsearch-analysis-ik-6.8.2.zip` 安装对应版本的插件

## 配置

mac 上使用 brew 安装的 es 的配置目录位于 `/usr/local/etc/elasticsearch`

* elasticsearch.yml: es 相关配置
    * cluster.name: 集群名称, 以此作为是否同一集群的判断条件
    * node.name: 节点名称, 以此作为集群中不同节点的区分条件
    * network.host/http.port: 网络地址和端口, 用于 http 和 transport 服务使用
    * path.data: 数据存储地址
    * path.log: 日志存储地址
* jvm.options: jvm 相关配置
* log4j2.properties: 日志相关配置

## 启动

### 单节点

单节点直接运行 `elasticsearch` 命令

### 集群启动

elasticsearch

elasticsearch -Ehttp.port=8200 -Epath.data=node2

elasticsearch -Ehttp.port=7200 -Epath.data=node3



使用 `http://localhost:9200/_cat/nodes?v` 查看集群是否启动成功









## ElasticSearch的基本概念

Index: 类似于mysql数据库中的database

Type: 类似于mysql数据库中的table表，es中可以在Index中建立type（table），通过mapping进行映射。

Document: 由于es存储的数据是文档型的，一条数据对应一篇文档即相当于mysql数据库中的一行数据row，一个文档中可以有多个字段也就是mysql数据库一行可以有多列。

Field: es中一个文档中对应的多个列与mysql数据库中每一列对应

Mapping: 可以理解为mysql或者solr中对应的schema，只不过有些时候es中的mapping增加了动态识别功能，感觉很强大的样子，其实实际生产环境上不建议使用，最好还是开始制定好了对应的schema为主。

Indexed: 就是名义上的建立索引。mysql中一般会对经常使用的列增加相应的索引用于提高查询速度，而在es中默认都是会加上索引的，除非你特殊制定不建立索引只是进行存储用于展示，这个需要看你具体的需求和业务进行设定了。

Query DSL: 类似于mysql的sql语句，只不过在es中是使用的json格式的查询语句，专业术语就叫：QueryDSL

GET/PUT/POST/DELETE: 分别类似与mysql中的select/update/delete......







## ElasticSearch的架构

![http://www.miaomiaoqi.cn/images/elastic/search/es_1.png](http://www.miaomiaoqi.cn/images/elastic/search/es_1.png)

### **Gateway层**

es用来存储索引文件的一个文件系统且它支持很多类型，例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的hdfs分布式存储、亚马逊的S3。它的主要职责是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。

### **Distributed Lucene Directory**

Gateway上层就是一个lucene的分布式框架，lucene是做检索的，但是它是一个单机的搜索引擎，像这种es分布式搜索引擎系统，虽然底层用lucene，但是需要在每个节点上都运行lucene进行相应的索引、查询以及更新，所以需要做成一个分布式的运行框架来满足业务的需要。

### **四大模块组件**

districted lucene directory之上就是一些es的模块，Index Module是索引模块，就是对数据建立索引也就是通常所说的建立一些倒排索引等；Search Module是搜索模块，就是对数据进行查询搜索；Mapping模块是数据映射与解析模块，就是你的数据的每个字段可以根据你建立的表结构通过mapping进行映射解析，如果你没有建立表结构，es就会根据你的数据类型推测你的数据结构之后自己生成一个mapping，然后都是根据这个mapping进行解析你的数据；River模块在es2.0之后应该是被取消了，它的意思表示是第三方插件，例如可以通过一些自定义的脚本将传统的数据库（mysql）等数据源通过格式化转换后直接同步到es集群里，这个river大部分是自己写的，写出来的东西质量参差不齐，将这些东西集成到es中会引发很多内部bug，严重影响了es的正常应用，所以在es2.0之后考虑将其去掉。

### **Discovery、Script**

es4大模块组件之上有 Discovery模块：es是一个集群包含很多节点，很多节点需要互相发现对方，然后组成一个集群包括选主的，这些es都是用的discovery模块，默认使用的是 Zen，也可是使用EC2；es查询还可以支撑多种script即脚本语言，包括mvel、js、python等等。

### **Transport协议层**

再上一层就是es的通讯接口Transport，支持的也比较多：Thrift、Memcached以及Http，默认的是http，JMX就是java的一个远程监控管理框架，因为es是通过java实现的。

### **RESTful接口层**

最上层就是es暴露给我们的访问接口，官方推荐的方案就是这种Restful接口，直接发送http请求，方便后续使用nginx做代理、分发包括可能后续会做权限的管理，通过http很容易做这方面的管理。如果使用java客户端它是直接调用api，在做负载均衡以及权限管理还是不太好做。









# 术语

Index索引: 类似于 mysql 中的数据库

Type: 索引中的数据类型, 类似 mysql 中的表

Document文档数据类似, 具体存在 es 中的一条数据, 类似 mysql 的一条记录

Field: 字段, 文档的属性, 类似 mysql 的字段

QueryDSL: 查询语法



# CRUD

 创建一个文档

```json
POST /accounts/person/1
{
	"name": "John",
	"lastName": "Doe",
	"job_desciption": "System administrator and Linux specialit"
}
```

在 index 为 accounts 下 type 为 person 的类型中创建一个 id 为 1 的 Document, 执行后会返回一个结果

```json
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```



获取一个文档

```
GET /accounts/person/1
```

```
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "John",
    "lastName" : "Doe",
    "job_desciption" : "System administrator and Linux specialit"
  }
}
```



更新一个文档

```
POST /accounts/person/1/_update
{
  "doc": {
    "lastName": "miaomiaoqi"
  }
}
```



删除一个文档

```
DELETE accounts/person/1
DELETE accounts
```



# ElasticsearchQuery

创建测试数据

```
POST /accounts/person/2
{
  "name": "miaomiao",
  "lastName": "qi",
  "job_description": "teacher"
}
```

## QueryString

```
GET /accounts/person/_search?q=miaomiao
```



## QueryDSL

```
GET /accounts/person/_search
{
	"query": {
		"match": {
			"job_description": "teacher"
		}
	}
}
```



# 索引

es 有专门的 Index API, 用于创建, 更新, 删除索引配置等

## 创建

`PUT /test_index`

## 查看

`GET _cat/indices`

## 删除

`DELETE /test_index`



# 文档

es 有专门的 DocumentAPI

