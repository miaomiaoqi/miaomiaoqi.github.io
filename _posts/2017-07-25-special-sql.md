---
layout: post
title:  "MySQL疑难SQL"
date:   2016-12-07 19:57:05
categories: database
tags: MySQL
author: miaoqi
---

* content
{:toc}

## 补充自增主键

    alter table `表格名` add column `列名` int not null auto_increment primary key comment '主键' first;
