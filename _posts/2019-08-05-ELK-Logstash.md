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

![http://www.milky.show/images/elastic/logstash/ls_1.png](http://www.milky.show/images/elastic/logstash/ls_1.png)

## 安装运行

到[官网](https://www.elastic.co/cn/downloads/logstash)下载对应版本上传到服务器解压, bin/logstash 即可单实例运行



### logstash 多实例运行方式

bin/logstash \-\-path.settings config1

bin/logstash \-\-path.settings config2

**不同 config 中修改 logstash.yml, 自定义 path.data, 确保其不相同即可**



## 架构

多种数据输入源经过 codec 投递到队列中, Batcher 从队列中拉取数据, 当达到等待时间或者数据阈值会将数据流转到 filter,output 中处理

<img src="http://www.milky.show/images/elastic/logstash/ls_5.png" alt="http://www.milky.show/images/elastic/logstash/ls_5.png" style="zoom:50%;" />

### 数据处理流程

Input 数据采集: file, reids, beats, kafka

Filter 数据解析/转换: grok, mutate, drop, date, geoip, useragent

Output 数据输出: stdout, elasticsearch, redis, kafka

传输过程中数据都会被封装成 Logstash Event

<img src="http://www.milky.show/images/elastic/logstash/ls_2.png" alt="http://www.milky.show/images/elastic/logstash/ls_2.png" style="zoom: 67%;" />

#### Input 配置

input {file{path => "/tmp/abc.log"}}

#### Filter 配置

Grok
* 基于正则表达式提供了丰富的可重用的模式(pattern)
* 基于此可将非结构化数据作结构化处理

Date

* 将字符串类型的时间字段转换为时间戳类型, 方便后续数据处理

Mutate

* 进行增加, 修改, 删除, 替换等字段相关的处理

#### Output 配置

Output{stdout{codec => rubydebug}}

#### 简单示例

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

<img src="http://www.milky.show/images/elastic/logstash/ls_3.png" alt="http://www.milky.show/images/elastic/logstash/ls_3.png" style="zoom: 50%;" />

<img src="http://www.milky.show/images/elastic/logstash/ls_4.png" alt="http://www.milky.show/images/elastic/logstash/ls_4.png" style="zoom: 50%;" />



### Logstash Event

Logstash 内部流转的数据表现形式, 数据都会被封装为 logstash event

原始数据在 input 被转换为 Event, 在 output event 被转换为目标格式数据

在配置文件中可以对 Event 中的属性进行增删改查

**event 生命周期**

<img src="http://www.milky.show/images/elastic/logstash/ls_6.png" alt="http://www.milky.show/images/elastic/logstash/ls_6.png" style="zoom: 50%;" />

<img src="http://www.milky.show/images/elastic/logstash/ls_7.png" alt="http://www.milky.show/images/elastic/logstash/ls_7.png" style="zoom:50%;" />

### Queue 的分类

**In Memory**

* 固定大小, 无法修改

* 无法处理进程 Crash, 机器宕机等情况, 会导致数据丢失

**Persistent Queue In Disk**

* 可以处理进行 Crash 等情况, 保证数据不丢失, 通过各种 ack 下图中展示
* 保证数据至少消费一次
* 充当缓冲区, 可以替代 Kafka 等消息队列的作用

<img src="http://www.milky.show/images/elastic/logstash/ls_8.png" alt="http://www.milky.show/images/elastic/logstash/ls_8.png" style="zoom: 50%;" />

<img src="http://www.milky.show/images/elastic/logstash/ls_9.png" alt="http://www.milky.show/images/elastic/logstash/ls_9.png" style="zoom:50%;" />

* queue.type: persisted 默认是 memory
* queue.max_bytes: 4gb 队列存储最大数据量



### 线程配置(调优)

<img src="http://www.milky.show/images/elastic/logstash/ls_10.png" alt="http://www.milky.show/images/elastic/logstash/ls_10.png" style="zoom: 50%;" />

* pipeline.workers(配置文件) \| -w(命令行)

    pipeline 线程数, 即 filter_output 的处理线程数, 默认是 cpu 核数

    `bin/logstash -f codec.conf -w 2`

* pipeline.batch.size \| -b

    Batcher 一次批量获取的待处理文档树, 默认是 125 个, 可以根据输出进行调整(比如输出到 es, es 建议批量 10mb~20mb, 根据每个文档的大小, 可以反推出 batchsize 的合理值), 越大会占越多的 heap 空间, 可以通过 jvm.options 调整

* pipeline.batch.delay \| -u

    Batcher 的等待时长, 单位为 ms

## 配置文件

logstash 设置相关的配置文件(在 conf 文件夹中, setting files)

* logstash.yml: logstash 相关的配置, 比如 node.name, path.data, pipeline.workers, queue.type 等, 这其中的配置可以被命令行参数中的相关参数覆盖

    ```yaml
    pipeline:
      batch:
        size: 125
        delay: 50
    ```

* jvm.options: 修改 jvm 相关的参数, 比如修改 heap size 等

pipeline 配置文件, 定义了 input, filter, output

* 定义数据处理流程的文件, 以 .conf 结尾

### logstash.yml 配置项

**`node.name`:** 节点名, 便于识别

**`path.data`:** 持久化存储数据的文件夹, 默认是 logstash home 目录下的 data

**`path.config`:** 设定 pipeline 配置文件的目录

**`path.log`:** 设定 pipeline 日志文件的目录

**`pipeline.workers`:** 设定 pipeline 的线程数(filter + output), **优化的常用项**

**`pipeline.batch.size/delay`:** 设定批量处理数据的数目和延迟

**`queue.type`:** 设定队列类型, 默认是 memory

**`queue.max_bytes`:** 队列总容量, 默认是 1g

### 命令行配置项

**`--node.name`:** 节点名称

**`-f --path.config.pipeline`:** 路径, 可以使文件或者文件夹

**`--path.settings`:** logstash 配置文件夹路径, 其中要包含 logstash.yml, 集群方案, 默认是 config 目录

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

### logstash 配置方式建议

线上环境推荐采用配置文件的方式来设定 logstash 的相关配置, 这样可以减少犯错机会, 而且文件便于进行版本化管理

命令行形式多用来进行快速的配置测试, 验证, 检查等



## Pipeline

用于配置 input, codec, filter 和 output 插件, 队列管理, 插件生命周期管理, 以 .conf 结尾的文件

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

### Input 插件

input 插件指定数据输入源, 一个 pipeline 可以有多个 input 插件, 我们主要了解下面几个 input 插件

* stdin
* file
* kafka
* http

#### stdin

**最简单的输入, 从标准输入读取数据, 通用配置为**

* codec 类型为 codec
* type 类型为 string, 自定义该事件的类型, 可用于后续判断
* tag 类型为 array, 自定义该事件的 tag, 可用于后续判断
* add_field 类型为 hash, 为该事件添加字段

编辑 input-stdin.conf 文件

```json
input {
  stdin {
    codec => "plain"
    tags => ["test"]
    type => "std"
    add_field => {"key" => "value"}
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}
```

执行该 pipeline

```bash
echo "test"|bin/logstash -f imooc/input-stdin.conf
```

查看输出结果

```bash
[2020-01-16T15:38:46,675][INFO ][logstash.inputs.stdin    ] Automatically switching from plain to line codec {:plugin=>"stdin"}
[2020-01-16T15:38:46,983][INFO ][logstash.pipeline        ] Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0x6536cb97 run>"}
[2020-01-16T15:38:47,263][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2020-01-16T15:38:49,107][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
/usr/local/logstash-6.8.4/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
{
          "tags" => [
        [0] "test"
    ],
      "@version" => "1",
          "type" => "std",
       "message" => "test",
           "key" => "value",
          "host" => "VM_0_9_centos",
    "@timestamp" => 2020-01-16T07:38:47.390Z
}
[2020-01-16T15:38:49,354][INFO ][logstash.pipeline        ] Pipeline has terminated {:pipeline_id=>"main", :thread=>"#<Thread:0x6536cb97 run>"}
[2020-01-16T15:38:49,869][INFO ][logstash.runner          ] Logstash shut down.
```

#### file

**从文件读取数据, 如常见的日志文件, 文件读取通常要解决几个问题**

* 文件内容如何只被读取一次, 即重启 LS 时, 从上次读取的位置继续

    sincedb

* 如何即时读取到文件的新内容

    定时检查文件是否有更新

* 如何发现新文件并进行读取

    可以定时检查新文件

* 如果文件发生了归档(rotation)操作, 是否影响当前的内容读取

    不影响, 被归档的文件内容可以继续被读取

**通用配置如下**

* path 类型为数组, 指明读取的文件路径, 基于 glob 匹配语法

    path => ["/var/log/\*\*/\*.log", "/var/log/message"]

* exclude 类型为数组, 排除不想监听的文件规则, 基于 glob 匹配语法

    exclude => "\*.gz"

* sincedb_path 类型为字符串, 记录 sincedb 文件路径, 上一次读取到的位置

* start_position 类型为字符串, beginning or end, 是否从头(sinced_path)读取文件, 默认 end(从 logstash 启动之后读取)

* start_interval 类型为数值, 单位秒, 定时检查文件是否有更新, 默认 1 秒

* discover_interval 类型为数值, 单位秒, 定时检查是否有新文件待读取, 默认 15 秒

* ignore_older 类型为数值, 单位秒, 扫描文件列表时, 如果该文件上次更改时间超过设定的时长, 则不做处理, 但依然会监控是否有新内容, 默认关闭

* close_older 类型为属猪, 单位秒, 如果正在监听的文件超过该设定时间内没有新内容, 会被关闭文件句柄, 释放资源, 但依然会监控是否有新内容, 默认 3600 秒, 即 1 小时

**glob 匹配语法**

* \* 匹配任意字符, 但不匹配以 \.开头的隐藏文件, 匹配这类文件使用 .\*来匹配
* \*\* 递归匹配子目录
* ? 匹配单一字符
* [] 匹配多个字符, 比如[a-z], \[^a-z\]
* {} 匹配多个单词, 比如{foo, bar, hello}
* \ 转义字符



**"/var/log/*.log":** 匹配 /var/log 目录下以 .log 结尾的文件

**"/var/log/\*\*/\*.log":** 匹配 /var/log 所有子目录下以 .log 结尾的文件

**"/var/log/{app1, app2, app3}/\*.log":** 匹配 /var/log 目录下 app1, app2, app3 目录中以 .log 结尾的文件

```json
input {
  file {
    path => ["/var/log/access_log", "/var/log/err_log"],
    type => "Web",
    start_position => "beginning"
  }
}
```

**调试文件常用配置**

```json
input {
  file {
    path => "/var/log/*.log"
    sincedb_path => "/dev/null"
    start_position => "beginning"
    ignore_older => 0
    close_older => 5
    discover_interval => 1
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}

# /dev/null 特殊文件, 所有的写入内容都不会存储, 那么每次运行该 pipeline 都会从头读取文件
```

#### kafka

kafka 是最流行的消息队列, 也是 Elastic Stack 架构中常用的, 使用相对简单

```json
input {
  kafka {
    zk_connect => "kafka:2181"
    group_id => "logstash"
    topic_id => "apache_logs"
    consumer_threads => 16
  }
}
```

### Codec 插件

Codec Plugin 作用于 input 和 output plugin, 负责将数据在原始内容与 Logstash Event 之间转换, 常见的 codec 有

* plain: 读取原始内容
* dots: 将内容简化为点进行输出
* rubydebug: 将 Logstash Events 按照 ruby 格式输出, 方便调试
* line: 处理带有换行符的内容
* json: 处理 json 格式的内容
* multiline: 处理多行数据的内容

```bash
bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=>rubydebug}}"

bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=>dots}}"

bin/logstash -e "input{stdin{codec=>json}}output{stdout{codec=>rubydebug}}"
```

#### line

每行数据都会被当做一个 message

#### multiline

当一个 Event 的 message 由多行组成时且需要当做一个整体来处理, 需要使用该 codec, 常见的情况是堆栈日志信息的处理

```java
Exception in thread "main" java.lang.NullPointeRException
  at com.miaoqi.myproject.Book.getTitle(Book.java:16)
  at com.miaoqi.myproject.Author.getBookTitles(Author.java:25)
```

主要设置参数如下

* pattern 设置行匹配的正则表达式, 可以使用 grok
* what previous\|next 如果匹配成功, 那么匹配行是归属上一个事件还是下一个事件
* negate true or false 是否对 pattern 的结果取反

编写 codec-multiline.conf 文件

```
input{
  stdin {
    codec => multiline{
      pattern => "^\s"
      what => "previous"
    }
  }
}
output {
  stdout {
    codec => "rubydebug"
  }
}
```

`bin/logstash -e imooc/codec-multiline.conf`



### Filter 插件

Filter 是 logstash 强大的主要原因, 它可以对 Logstash Event 进行丰富的处理, 比如解析数据, 删除字段, 类型转换等, 常见的有如下几个

* date  日期解析
* grok 正则匹配解析
* dissect 分隔符解析
* mutate 对字段作处理, 比如重命名, 删除, 替换等
* json 按照 json 解析字段内容到指定字段中
* geoip 增加地理位置数据
* ruby 利用 ruby 代码来动态修改 Logstash Event



#### date

将日期字符串解析为日期类型, 然后替换@timestamp 字段或者指定的其他字段

编写 imooc/filter-date1.conf

```
input {stdin{codec=>json}}
filter {
  date {
    match => ["logdate", "MMM dd yyyy HH:mm:ss"]
  }
}
output{stdout{codec=>rubydebug}}
```

运行该 pipeline 输入

`{"logdate": "Jan 01 2018 12:02:03"}`

**常用参数**

* match

    类型为数组, 用于指定日期匹配的格式, 可以一次指定多种日期格式

    match => ["logdate", "MMM dd yyyy HH:mm:ss", "MMM d yyyy HH:mm:ss", "ISO8601"]

* target

    类型为字符串, 用于指定赋值的字段名, 默认是@timestamp

* timezone

    类型为字符串, 用于指定时区



#### grok

解析一段日志

```
144.23.4.1 -- [13/Mar/2016:02:38:26-0400] "GET /fancy.html HTTP/1.1" 200 6146 "-" "Mozilla/5.0()"
```

语法如下

* %{SYNTAX:SEMANTIC}

    SYNTAX 为 gork pattern 的名称, SEMANTIC 为赋值字段的名称

    %{NUMBER:duration} 可以匹配数值类型, 但是 grok 匹配出的内容都是字符串类型, 可以通过在最后指定 int 或者 float 来强制转换类型

    %{NUMBER:duration:float}

* 熟悉一些常见的 Pattern 利于编写匹配规则

* https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns

* 自定义 gork pattern

    * (?<service_name>[0-9a-z]{10,11})

        ```
        input{stdin{}}
        filter{
          grok{
            match => {
              "message" -> "(?<service_name>[0-9a-z]{10,11})"
            }
          }
        }
        ```

        如果需要重复使用, 利用率就很低了

    * pattern_definitions 参数, 以键值对的方式定义 pattern 名称和内容

        pattern_dir 参数, 以文件的形式被读取

        ```
        filter {
          grok {
            match => {
              "message" => "%{SERVICE:service}"
            }
            pattern_definitions => {
              "SERVICE" => [a-z0-9]{10,11}
            }
          }
        }
        ```

    

#### dissect

基于分隔符原理解析数据, 解决 grok 解析时消耗过多 cpu 资源的问题

```
%{clientip} %{ident} %{auth} [%{timestamp}] "%{request}" %{response}
```

![http://www.milky.show/images/elastic/logstash/ls_11.png](http://www.milky.show/images/elastic/logstash/ls_11.png)

主要适用于每行格式相似且分隔符明确简单的场景

dissect 语法比较简单, 有一系列字段(field)和分隔符(delimiter)组成

* %{} 字段
* %{} 之间是分隔符

dissect 分割后的字段值都是字符串, 可以使用 convert_datatype 属性进行类型转换

```
filter{
  dissect {
    convert_datatype => {
      age => "int"
    }
  }
}
```



#### mutate

使用最频繁的插件, 可以对字段进行各种操作, 比如重命名, 删除, 替换, 更新等, 主要操作如下

* convert 类型转换

    实现字段类型的转换, 类型为 hash, 仅支持转换为 integer, float, string 和 boolean

    ```
    filter {
      mutate {
        convert => {"age" => "integer"}
      }
    }
    ```

* gsub 字符串替换

    对字段内容进行替换, 类型为数组, 每 3 项为一个替换配置

    ```
    filter {
      mutate {
        gsub => {
          "path", "/", "-",
          "urlparams", "[\\?#-]", "."
        }
      }
    }
    ```

    

* split/join/merge 字符串切割, 数组合并为字符串, 数组合并为数组

    ```
    filter{
      mutate{
        split => {"jobs" => ","}
      }
    }
    ```

    ```
    filter{
      mutate{
        join => {"params" => ","}
      }
    }
    ```

    ```
    filter{
      mutate{
        merge => {"dest_arr" => "source_arr"}
      }
    }
    ```

* rename 字段重命名

    ```
    filter {
      mutate {
        rename => {"HOSTORIP" => "client_ip"}
      }
    }
    ```

    

* update/replace 字段内容更新或替换

* remove_field 删除字段

#### json

将字段内容为 json 格式的数据进行解析

```
filter {
  json {
    source => "message"
    target => "msg_json"
  }
}
```



### Output 插件

负责将 Logstash Event 输出, 常见的插件如下

* stdout
* file
* elasticsearch

#### stdout

输出到标准输出, 多用于调试

```
output {
  stdout {
    codec => rubydebug
  }
}
```

#### file

输出多文件, 实现将分散在多地的文件统一到一处的需求, 比如将所有 web 机器的 web 日志收集到 1 个文件中, 从而方便查阅

```
output {
  file {
    path => "/var/log/web.log"
    codec => line{format => "%{message}"}
  }
}
```

#### elasticseatch

```
output {
  elasticsearch {
    hosts => ["127.0.0.1:9200", "127.0.0.2:9200"]
    index => "nginx-%{+YYYY.MM.dd}"
    template => "./nginx_tempalte.json"
  }
}
```



## Logstash 实战

### 调试的配置建议

http 做 input, 方便输入测试数据, 并且可以结合 reload 特性(stdin 无法 reload)

stdout 做 output, codec 使用 rubydebug, 即时查看结果

测试错误输入的情况下, 以便对错误情况进行处理

### 实例分析之 Apache Logs

收集 Apache 日志

手机 csv 到 elasticsearch



### 监控运维之 API

logstash 提供了丰富的 api 来查看 logstash 当前的状态

* http://localhost:9600
* http://localhost:9600/_node
* http://localhost:9600/_node/stats
* http://localhost:9600/_node/hot_threads