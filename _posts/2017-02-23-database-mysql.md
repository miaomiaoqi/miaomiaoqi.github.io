---
layout: post
title:  "MySQL 导入导出"
date:   2017-02-23 20:09:00
categories: database
tags: MySQL
author: miaoqi
---

* content
{:toc}

## 导出整个库

    mysqldump -u 用户名 -p 数据库名 > 导出的文件名
    
## 导出一个表
    
    mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名 
    
## 导入

    在mysql命令行界面 source 目标sql文件

    