---
layout: post
title: "ELK-Kibana"
categories: [BigData]
description:
keywords:
---

* content
{:toc}     


## Kibana

ElasticSearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene/6753302)的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于[云计算](https://baike.baidu.com/item/云计算/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

## 安装

我的是 mac, 使用命令 `brew install kibana` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/kibana/6.8.1` 配置文件目录为`/usr/local/etc/kibana`

## 配置

mac 上使用 brew 安装的 es 的配置目录位于 `/usr/local/etc/kibana`

- kibana.yml: kibana 相关配置

    * server.host/server.port: 访问 kibana 用的地址和端口

    - elasticsearch.hosts: 待访问的 elastic search 的地址

## 常用功能

**Discover:** 数据搜索查看

**Visualize: **图表只做

**Dashboard:**仪表盘制作

**Timelion:**  时序数据的高级可视化分析

**DevTools:** 开发者工具

**Management: **配置