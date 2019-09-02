---
layout: post
title:  "ELK-Logstash"
date:   2019-08-05 19:15:25
categories: BigData
tags: Logstash
author: miaoqi
---

* content
{:toc}     
# Logstash

Data Shipper, 与 Beat 不同, Logstash 是比较重的数据传送者, 但功能更加强大

* Extract
* Transform
* Load

# 处理流程

Input 输入数据: file, reids, beats, kafka

Filter 过滤数据: grok, mutate, drop, date

Output 输出数据: stdout, elasticsearch, redis, kafka

## Input配置

input {file{path => "/tmp/abc.log"}}

## Filter配置

* Grok
    * 基于正则表达式提供了丰富的可重用的模式(pattern)
    * 基于此可将非结构化数据作结构化处理
* Date
    * 将字符串类型的时间字段转换为时间戳类型, 方便后续数据处理
* Mutate
    * 进行增加, 修改, 删除, 替换等字段相关的处理

## Output配置

Output{stdout{codec => rubydebug}}