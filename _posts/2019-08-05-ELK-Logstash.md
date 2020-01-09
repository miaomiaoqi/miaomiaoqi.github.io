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
## Logstash 架构简介

Data Shipper, 与 Beat 不同, Logstash 是比较重的数据传送者, 但功能更加强大

**数据收集处理引擎**

**ETL 工具(Extract, Transform, Load)**

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_1.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_1.png)

## 安装

到[官网](https://www.elastic.co/cn/downloads/logstash)下载对应版本上传到服务器解压即可

## 处理流程

Input 数据采集: file, reids, beats, kafka

Filter 数据解析/转换: grok, mutate, drop, date, geoip, useragent

Output 数据输出: stdout, elasticsearch, redis, kafka

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_2.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_2.png)

### Input 配置

input {file{path => "/tmp/abc.log"}}

### Filter 配置

Grok
* 基于正则表达式提供了丰富的可重用的模式(pattern)
* 基于此可将非结构化数据作结构化处理

Date

* 将字符串类型的时间字段转换为时间戳类型, 方便后续数据处理

Mutate

* 进行增加, 修改, 删除, 替换等字段相关的处理

### Output 配置

Output{stdout{codec => rubydebug}}

### 简单示例

编写 codec.conf 配置文件

**line codec 插件会按行读取, json codec 会以 json 格式输出**

```
input {
	stdin {
		codec => line
	}
}

filter {}

output {
	stdout {
		codec => json
	}
}
```

**命令行执行如下命令, 模拟换行输入**

```shell
echo "foo
bar 
"|bin/logstash -f codec.conf
```

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_3.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_3.png)

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_4.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_4.png)

## Pipeline

input-filter-output 的 3 阶段处理流程

队列管理

插件生命周期管理



## Logstash Event

内部流转的数据表现形式

原始数据在 input 被转换为 Event, 在 output event 被转换为目标格式数据

在配置文件中可以对 Event 中的属性进行增删改查











