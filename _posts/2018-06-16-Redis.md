---
layout: post
title:  "ElasticSearch学习"
date:   2018-06-14 15:12:38
categories: Work
tags: ElasticSearch
author: miaoqi
---

* content
{:toc}
            

## ElasticSearch笔记

### 全文检索

* 底层是Lucene

* docker run -e ES\_JAVA\_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 5c1e1ecfe33a

* 初始运行会占用2g内存大小, -Xms256m: 指定初始大小256m, -Xmx256m: 指定最大大小256m, 9200: es服务端口, 9300: 分布式通信端口




    
    
    
    
    
    
    
    
    
    