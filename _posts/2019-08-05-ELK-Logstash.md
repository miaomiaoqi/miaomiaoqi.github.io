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
"|bin/logstash -f imooc/codec.conf
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



### 线程配置

![http://www.miaomiaoqi.cn/images/elastic/logstash/ls_10.png](http://www.miaomiaoqi.cn/images/elastic/logstash/ls_10.png)

* pipeline.works \| -w

    pipeline 线程数, 即 filter_output 的处理线程数, 默认是 cpu 核数

    `bin/logstash -f codec.conf -w 2`

* pipeline.batch.size \| -b

    Batcher 一次批量获取的待处理文档树, 默认是 125 个, 可以根据输出进行调整(比如输出到 es, es 建议批量 10mb~20mb, 根据每个文档的大小, 可以反推出 batchsize 的合理值), 越大会占越多的 heap 空间, 可以通过 jvm.options 调整

* pipeline.batch.delay \| -u

    Batcher 的等待时长, 单位为 ms

### 配置文件

logstash 设置相关的配置文件(在 conf 文件夹中, setting files)

* logstash.yml: logstash 相关的配置, 比如 node.name, path.data, pipeline.workers, queue.type 等, 这其中的配置可以被命令行参数中的相关参数覆盖

    ```yaml
    pipeline:
      batch:
        size: 125
        delay: 50
    ```

* jvm.options: 修改 jvm 相关的参数, 比如修改 heap size 等

pipeline 配置文件

* 定义数据处理流程的文件, 以 .conf 结尾

#### logstash.yml 配置项

**`node.name`:** 节点名, 便于识别

**`path.data`:** 持久化存储数据的文件夹, 默认是 logstash home 目录下的 data

**`path.config`:** 设定 pipeline 配置文件的目录

**`path.log`:** 设定 pipeline 日志文件的目录

**`pipeline.workers`:** 设定 pipeline 的线程数(filter + output), **优化的常用项**

**`pipeline.batch.size/delay`:** 设定批量处理数据的数目和延迟

**`queue.type`:** 设定队列类型, 默认是 memory

**`queue.max_bytes`:** 队列总容量, 默认是 1g

#### 命令行配置项

**`--node.name`:** 节点名称

**`-f --path.config.pipeline`:** 路径, 可以使文件或者文件夹

**`--path.settings`:** logstash 配置文件夹路径, 其中要包含 logstash.yml, 集群方案

**`-e --config.string`:** 指明 pipeline 内容, 多用于测试使用

```bash
bin/logstash -e "input{stdin{}}output{stdout{codec=>line}}" -t

Sending Logstash logs to /usr/local/logstash-6.8.4/logs which is now configured via log4j2.properties
[2020-01-16T10:48:13,569][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
Configuration OK
[2020-01-16T10:48:29,921][INFO ][logstash.runner          ] Using config.test_and_exit mode. Config Validation Result: OK. Exiting Logstash
```

**`-w --pipeline.workers`**

**`-b --pipeline.batch.size`**

**`--path.data`:** 数据存储路径

**`--debug`:** 打开调试日志

**`-t --config.test_and_exit`:** 做测试, 只是检验配置文件是否正确

#### logstash 配置方式建议

线上环境推荐采用配置文件的方式来设定 logstash 的相关配置, 这样可以减少犯错机会, 而且文件便于进行版本化管理

命令行形式多用来进行快速的配置测试, 验证, 检查等



### logstash 多实例运行方式

bin/logstash \-\-path.settings config1

bin/logstash \-\-path.settings config1

不同 config 中修改 logstash.yml, 自定义 path.data, 确保其不相同即可















## Pipeline

用于配置 input, filter 和 output 插件, 队列管理, 插件生命周期管理

```json
input{}

filter{}

output{}
```

### 配置语法

主要有如下数据类型

* 布尔类型 Boolean

    isFailed => true

* 数值类型 Number

    port => 33

* 字符串类型 String

    name => "Hello World"

* 数组 Array/List

    users => [{id => 1, name => bob}, {id => 2, name => jane}]

    path => ["/var/log/messages", "/var/log/*.log"]

* 哈希类型 Hash

    match => {

    ​	"field1" => "value1"

    ​	"field2" => "value2"

    }

* 注释 #

    \# this is a comment

* 在配置中可以引用 Logstash Event 的属性(字段), 主要有如下两种方式

    * 直接引用字段值 Field Reference

        使用 [] 即可, 嵌套字段写多层 [] 即可

        ```json
        {
          "agent": "Mozilla/5.0",
          "ip": "192.168.24.44",
          "request", "/index.html",
          "response": {
            "status": 200,
            "bytes": 52353
          },
          "ua": {
            "os": "Mac OS"
          }
        }
        ```

        ```
        ...
        if [request] =~ "index"{}
        ...
        ```

        ```
        ...
        if [ua][os] =~ "Windows"{}
        ...
        ```

    * 在字符串中以 sprintf 方式引用, 使用 %{} 实现

        ```
        req => "request is %{request}"
        ```

        ```
        ua => "ua is %{[ua][us]}"
        ```

* 支持条件判断语法, 从而扩展了配置的多样性

    ```
    if EXPRESSION {
      ...
    } else if EXPRESSION {
      ...
    } else {
      ...
    }
    ```

* 表达式包含如下的操作符

    **比较:** ==, !=, <, >, <=, >=

    **正则是否匹配:** =~, !~

    **包含(字符串或者数组):** in, not in

    **布尔操作符:** and, or, nand, xor, !

    **分组操作符:** ()



## Logstash Event

内部流转的数据表现形式

原始数据在 input 被转换为 Event, 在 output event 被转换为目标格式数据

在配置文件中可以对 Event 中的属性进行增删改查











