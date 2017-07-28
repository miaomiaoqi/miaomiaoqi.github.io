---
layout: post
title:  "MySQL疑难SQL"
date:   2016-12-07 19:57:05
categories: Database
tags: MySQL
author: miaoqi
---

## 补充自增主键

    alter table `表名` add column `列名` int not null auto_increment primary key comment '主键' first;
    
## 中文字段排序

    ORDER BY CONVERT(column USING gbk) ASC
