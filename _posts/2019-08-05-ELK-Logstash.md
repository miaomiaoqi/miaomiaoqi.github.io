---
layout: post
title: "ELK-Logstash"
categories: [BigData]
description:
keywords:
---

* content
{:toc}     
## Logstash 简介

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

传输过程中数据都会被封装成 Logstash Event

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

## 架构

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_5.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_5.png)



### Life_of_an_Event

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_6.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_6.png)

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_7.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_7.png)

### Queue 的分类

**In Memory**

* 固定大小, 无法修改
* 无法处理进程 Crash, 机器宕机等情况, 会导致数据丢失

**Persistent Queue In Disk**

* 可以处理进行 Crash 等情况, 保证数据不丢失
* 保证数据至少消费一次
* 充当缓冲区, 可以替代 Kafka 等消息队列的作用

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_8.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_8.png)

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_9.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_9.png)

* queue.type: persisted 默认是 memory
* queue.max_bytes: 4gb 队列存储最大数据量



### 线程

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_10.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_10.png)

* pipeline.works | -w

    pipeline 线程数, 即 filter_output 的处理线程数, 默认是 cpu 核数

    `bin/logstash -f codec.conf -w 2`

* pipeline.batch.size | -b

    Batcher 一次批量获取的待处理文档树, 默认是 125 个, 可以根据输出进行调整(比如输出到 es, es 建议批量 10mb~20mb, 根据每个文档的大小, 可以反推出 batchsize 的合理值), 越大会占越多的 heap 空间, 可以通过 jvm.options 调整

* pipeline.batch.delay | -u

    Batcher 的等待时长, 单位为 ms

### 配置文件

logstash 设置相关的配置文件(在 conf 文件夹中, setting files)

* logstash.yml logstash 相关的配置, 比如 node.name, path.data, pipeline.workers, queue.type 等, 这其中的配置可以被命令行参数中的相关参数覆盖
* jvm.options 修改 jvm 相关的参数, 比如修改 heap size 等

pipeline 配置文件

* 定义数据处理流程的文件, 以 .conf 结尾

















## Pipeline

input-filter-output 的 3 阶段处理流程

队列管理

插件生命周期管理



## Logstash Event

内部流转的数据表现形式

原始数据在 input 被转换为 Event, 在 output event 被转换为目标格式数据

在配置文件中可以对 Event 中的属性进行增删改查











