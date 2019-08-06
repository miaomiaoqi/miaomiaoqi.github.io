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
# ElasticSearch

ElasticSearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene/6753302)的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于[云计算](https://baike.baidu.com/item/云计算/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

# 安装

我的是 mac, 使用命令 `brew install elasticsearch` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/elasticsearch/6.8.2` 配置文件目录为`/usr/local/etc/elasticsearch`

使用`brew services start elasticsearch` 可以将 es 作为服务启动, 我在启动项目的时候遇到了两个问题

* **Cannot open file logs/gc.log due to No such file or directory**

    这个问题是因为日志文件的路径有问题, 可以修改配置文件 `jvm.options` 配置文件, 指定 `8:-Xloggc:/Users/miaoqi/Documents/elasticsearch/logs/gc.log` 日志文件的路径

* **Plugin [analysis-ik] was built for Elasticsearch version 6.7.2 but version 6.8.2 is running**

    这个问题是因为安装了 IKAnalysis 插件, 但是插件的版本与 es 的版本不对应

    可以使用 `elasticsearch-plugin list` 查看已经安装的插件

    然后使用 `elasticsearch-plugin remove plugin-name` 删除已有的插件

    最后使用 `elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.2/elasticsearch-analysis-ik-6.8.2.zip` 安装对应版本的插件

# 配置

mac 上使用 brew 安装的 es 的配置目录位于 `/usr/local/etc/elasticsearch`

* elasticsearch.yml: es 相关配置
    * cluster.name: 集群名称, 以此作为是否同一集群的判断条件
    * node.name: 节点名称, 以此作为集群中不同节点的区分条件
    * network.host/http.port: 网络地址和端口, 用于 http 和 transport 服务使用
    * path.data: 数据存储地址
    * path.log: 日志存储地址
* jvm.options: jvm 相关配置
* log4j2.properties: 日志相关配置

# 启动

## 单节点

单节点直接运行 `elasticsearch` 命令

## 集群启动

elasticsearch

elasticsearch -Ehttp.port=8200 -Epath.data=node2

elasticsearch -Ehttp.port=7200 -Epath.data=node3



使用 http://localhost:9200/_cat/nodes?v 查看集群是否启动成功



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

