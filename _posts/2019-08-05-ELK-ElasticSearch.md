---
layout: post
title:  "ELK-ElasticSearch"
date:   2019-08-05 13:54:25
categories: BigData
tags: ElasticSearch
author: miaoqi
---

* content
{:toc}     
## ElasticSearch

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTfulweb接口。ElasticSearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。构建在全文检索开源软件Lucene之上的Elasticsearch，不仅能对海量规模的数据完成分布式索引与检索，还能提供数据聚合分析。据国际权威的数据库产品评测机构DBEngines的统计，在2016年1月，Elasticsearch已超过Solr等，成为排名第一的搜索引擎类应用

https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

## 安装

### 安装 ES

我的是 mac, 使用命令 `brew install elasticsearch` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/elasticsearch/6.8.3` 配置文件目录为`/usr/local/etc/elasticsearch`

mac 上使用 brew 安装的 es 的配置文件目录位于 `/usr/local/etc/elasticsearch`

- elasticsearch.yml: es 相关配置
    - cluster.name: 集群名称, 以此作为是否同一集群的判断条件
    - node.name: 节点名称, 以此作为集群中不同节点的区分条件
    - network.host/http.port: 网络地址和端口, 用于 http 和 transport 服务使用
    - path.data: 数据存储地址
    - path.log: 日志存储地址
- jvm.options: jvm 相关配置
- log4j2.properties: 日志相关配置

**使用`brew services start elasticsearch` 可以将 es 作为服务启动, 我在启动项目的时候遇到了两个问题**

* **Cannot open file logs/gc.log due to No such file or directory**

    这个问题是因为日志文件的路径有问题, 可以修改配置文件 `jvm.options` 配置文件, 指定 `8:-Xloggc:/Users/miaoqi/Documents/elasticsearch/logs/gc.log` 日志文件的路径

* **Plugin [analysis-ik] was built for Elasticsearch version 6.7.2 but version 6.8.3 is running**

    es 插件的目录是`/usr/local/var/elasticsearch/plugins`

    这个问题是因为安装了 IKAnalysis 插件, 但是插件的版本与 es 的版本不对应

    可以使用 `elasticsearch-plugin list` 查看已经安装的插件

    然后使用 `elasticsearch-plugin remove plugin-name` 删除已有的插件
    
    最后使用 `elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.3/elasticsearch-analysis-ik-6.8.3.zip` 安装对应版本的插件



**linux 安装**

1. 到 [ES 官网](https://www.elastic.co/cn/downloads/elasticsearch) 下载对应操作系统的压缩包
2. 解压缩到安装目录
3. 运行 `bin/elasticsearch` 即可



### 安装中文分词器插件

`elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.4/elasticsearch-analysis-ik-6.8.4.zip`

### 安装 Kibana

Kibana 是一个针对 Elasticsearch 的开源分析及可视化平台，使用Kibana可以查询、查看并与存储在ES索引的数据进行交互操作，使用Kibana能执行高级的数据分析，并能以图表、表格和地图的形式查看数据

我的是 mac, 使用命令 `brew install kibana` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/kibana/6.8.1` 配置文件目录为`/usr/local/etc/kibana`

`http://localhost:5601 ` 访问 kibana



### 安装 cerebro 插件

cerebro 是 ES 集群的管理工具, 可以方便的查看集群状态 [下载地址](https://github.com/lmenezes/cerebro)

下载对应的压缩包解压运行即可

http://localhost:9000



## 启动

### 单节点

单节点直接运行 `elasticsearch` 命令, 会自动加载 elasticsearch.yml 的配置

### 集群启动

```shell
elasticsearch -Ecluster.name=miaoqi_cluster -Epath.data=miaoqi_cluster_node1 -Enode.name=miaoqi_node1 -Ehttp.port=7200 -d

elasticsearch -Ecluster.name=miaoqi_cluster -Epath.data=miaoqi_cluster_node2 -Enode.name=miaoqi_node2 -Ehttp.port=8200 -d
```

**不同的节点, 指定同一个 cluster.name 不同的 node.name 即可成为集群, 这些配置都可以在 elasticsearch.yml 中更改, jvm.options 可以修改 jvm 参数调低内存, 方便大家练习时候使用**

使用 `http://localhost:9200/_cat/nodes?v` 查看集群是否启动成功



## ElasticSearch的基本概念

### 术语介绍

**Index:** 类似于mysql数据库中的 database

**Type:** 类似于mysql数据库中的 table 表，es 中可以在 Index 中建立 type（table），通过 mapping 进行映射, 6.0 以后一个 Index 下只允许创建一个 Type。

**Document:** 由于 es 存储的数据是文档型的，一条数据对应一篇文档即相当于 mysql 数据库中的一行数据 row，一个文档中可以有多个字段也就是 mysql 数据库一行可以有多列。

**Field:** es中一个文档中对应的多个列与 mysql 数据库中每一列对应

**Mapping:** 可以理解为 mysql 或者 solr 中对应的 schema，只不过有些时候 es 中的 mapping 增加了动态识别功能，感觉很强大的样子，其实实际生产环境上不建议使用，最好还是开始制定好了对应的 schema 为主。

**Indexed:** 就是名义上的建立索引。mysql 中一般会对经常使用的列增加相应的索引用于提高查询速度，而在 es 中默认都是会加上索引的，除非你特殊制定不建立索引只是进行存储用于展示，这个需要看你具体的需求和业务进行设定了。

**Query DSL:** 类似于 mysql 的 sql 语句，只不过在 es 中是使用的 json 格式的查询语句，专业术语就叫：QueryDSL

GET/PUT/POST/DELETE: 分别类似与mysql中的select/update/delete......



### 版本控制

ElasticSearch采用了乐观锁来保证数据的一致性，也就是说，当用户对document进行操作时，并不需要对该document作加锁和解锁的操作，只需要指定要操作的版本即可。当版本号一致时，ElasticSearch会允许该操作顺利执行，而当版本号存在冲突时，ElasticSearch会提示冲突并抛出异常（VersionConflictEngineException异常）。

ElasticSearch的版本号的取值范围为1到2^63-1。

内部版本控制：使用的是_version

```json
PUT /lib/user/4?version=3
{
	"first_name": "xixi"
}
如果 version 和文档中的版本号不一致会抛出异常
```

外部版本控制: elasticsearch 在处理外部版本号时会与对内部版本号的处理有些不同。它不再是检查 `_version` 是否与请求中指定的数值相同, 而是检查当前的 `_version` 是否比指定的数值小。如果请求成功，那么外部的版本号就会被存储到文档中的 `_version` 中。

为了保持 `_version` 与外部版本控制的数据一致
使用version_type=external



## ElasticSearch倒排索引

Elasticsearch 使用一种称为 倒排索引 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

假设文档集合包含五个文档，每个文档内容如图所示，在图中最左端一栏是每个文档对应的文档编号。我们的任务就是对这个文档集合建立倒排索引。
![http://www.miaomiaoqi.cn/images/elastic/search/es_2.png](http://www.miaomiaoqi.cn/images/elastic/search/es_2.png)

(2):中文和英文等语言不同，单词之间没有明确分隔符号，所以首先要用分词系统将文档自动切分成单词序列。这样每个文档就转换为由单词序列构成的数据流，为了系统后续处理方便，需要对每个不同的单词赋予唯一的单词编号，同时记录下哪些文档包含这个单词，在如此处理结束后，我们可以得到最简单的倒排索引
![http://www.miaomiaoqi.cn/images/elastic/search/es_3.png](http://www.miaomiaoqi.cn/images/elastic/search/es_3.png)
“单词ID”一栏记录了每个单词的单词编号，第二栏是对应的单词，第三栏即每个单词对应的倒排列表

(3):索引系统还可以记录除此之外的更多信息,下图还记载了单词频率信息（TF）即这个单词在某个文档中的出现次数，之所以要记录这个信息，是因为词频信息在搜索结果排序时，计算查询和文档相似度是很重要的一个计算因子，所以将其记录在倒排列表中，以方便后续排序时进行分值计算。

![http://www.miaomiaoqi.cn/images/elastic/search/es_4.png](http://www.miaomiaoqi.cn/images/elastic/search/es_4.png)

(4):倒排列表中还可以记录单词在某个文档出现的位置信息

(1,<11>,1),(2,<7>,1),(3,<3,9>,2)

有了这个索引系统，搜索引擎可以很方便地响应用户的查询，比如用户输入查询词“Facebook”，搜索系统查找倒排索引，从中可以读出包含这个单词的文档，这些文档就是提供给用户的搜索结果，而利用单词频率信息、文档频率信息即可以对这些候选搜索结果进行排序，计算文档和查询的相似性，按照相似性得分由高到低排序输出，此即为搜索系统的部分内部流程。

### 倒排索引原理

1.The quick brown fox jumped over the lazy dog

2.Quick brown foxes leap over lazy dogs in summer

倒排索引：

|  Term  | Doc_1 | Doc_2 |
| :----: | :---: | :---: |
| Quick  |       |   X   |
|  The   |   X   |       |
| brown  |   X   |   X   |
|  dog   |   X   |       |
|  dogs  |       |   X   |
|  fox   |   X   |       |
| foxes  |       |   X   |
|   in   |       |   X   |
| jumped |   X   |       |
|  lazy  |   X   |   X   |
|  leap  |       |   X   |
|  over  |   X   |   X   |
| quick  |   X   |       |
| summer |       |   X   |
|  The   |   X   |       |

**搜索含有 quick 或者 brown 的文档：**

| Term  | Doc_1 | Doc_2 |
| ----- | ----- | ----- |
| brown | X     | X     |
| quick | X     |       |

---------------------

Total   |   2   |  1  |

计算相关度分数时，文档1的匹配度高，分数会比文档2高



**问题：**

Quick 和 quick 以独立的词条出现，然而用户可能认为它们是相同的词。

fox 和 foxes 非常相似, 就像 dog 和 dogs ；他们有相同的词根。

jumped 和 leap, 尽管没有相同的词根，但他们的意思很相近。他们是同义词。

搜索含有 Quick fox的文档是搜索不到的, 因为是区分大小写的

**ES使用标准化规则(normalization)解决上述问题：**
建立倒排索引的时候，会对拆分出的各个单词进行相应的处理，以提升后面搜索的时候能够搜索到相关联的文档的概率

|  Term  | Doc_1 | Doc_2 |
| :----: | :---: | :---: |
| brown  |   X   |   X   |
|  dog   |   X   |   X   |
|  fox   |   X   |   X   |
|   in   |       |   X   |
|  jump  |   X   |   X   |
|  lazy  |   X   |   X   |
|  over  |   X   |   X   |
| quick  |   X   |   X   |
| summer |       |   X   |
|  the   |   X   |   X   |





## 分词和分词器

分词：从一串文本中切分出一个一个的词条，并对每个词条进行**标准化**

分词器包括三部分：

**character filter：针对原始文本的预处理，过滤掉HTML标签，特殊符号转换等**

**tokenizer：将原始文本按照一定规则切分为单词**

**token filter：标准化, 针对 tokenizer 处理的单词进行再加工, 比如转小写, 删除或新增等处理**

### 内置分词器

**standard 分词器**：(默认的)他会将词汇单元转换成小写形式，并去除停用词和标点符号，支持中文采用的方法为单字切分

![http://www.miaomiaoqi.cn/images/elastic/search/es_8.png](http://www.miaomiaoqi.cn/images/elastic/search/es_8.png)

**simple 分词器**：首先会通过非字母字符来分割文本信息，然后将词汇单元统一为小写形式。该分析器会去掉数字类型的字符。

![http://www.miaomiaoqi.cn/images/elastic/search/es_9.png](http://www.miaomiaoqi.cn/images/elastic/search/es_9.png)

**Whitespace** 分词器：仅仅是去除空格，对字符没有lowcase化,不支持中文；
并且不对生成的词汇单元进行其他的标准化处理。

![http://www.miaomiaoqi.cn/images/elastic/search/es_10.png](http://www.miaomiaoqi.cn/images/elastic/search/es_10.png)

**Stop 分词器**: 

![http://www.miaomiaoqi.cn/images/elastic/search/es_11.png](http://www.miaomiaoqi.cn/images/elastic/search/es_11.png)

**Keyword 分词器**:

![http://www.miaomiaoqi.cn/images/elastic/search/es_12.png](http://www.miaomiaoqi.cn/images/elastic/search/es_12.png)

**Pattern 分词器**:

![http://www.miaomiaoqi.cn/images/elastic/search/es_13.png](http://www.miaomiaoqi.cn/images/elastic/search/es_13.png)

**language 分词器**：特定语言的分词器，不支持中文



### AnalyzerAPI

es 提供了一个测试分词的 api 接口, 方便验证分词效果, endpoint 是 _analyze

**可以直接指定 analyzer 进行测试**

```json
POST _analyze
{
	"analyzer": "standard",
	"text": "hello world"
}

{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```

**可以直接指定索引中的字段进行测试**

```json
POST test_index/_analyze
{
	"field": "username", # 测试字段
	"text": "hello world" # 测试文本
}
```

**可以自定义分词器进行测试**

```json
POST _analyze
{
	"tokenizer": "standard",
	"filter": ["lowercase"],
	"text": "Hello World!"
}
```

### 中文分词

中文分词器是指将一个汉字序列切分成一个一个单独的词语. 在英文中, 单词之间是以空格作为自然分界符, 汉语中没有一个形式上的分界符

上下文不同, 分词结果迥异, 比如交叉歧义问题

* 乒乓球拍/卖/完了
* 乒乓球/拍卖/完了

**常用中文分词系统**

**IK**

实现中英文单词的切分, 支持 ik_smart, ik_maxword 模式

可自定义词库, 支持热更新分词词典

**jieba**

python 中最流行的分词系统, 支持分词和词性标注

支持繁体分词, 自定义词典, 并行分词等

### 自定义分词

当自带的分词无法满足需求时, 可以自定义分词

通过自定义 Character Filters, Tokenizer 和 Token Filter实现

**Character Filter**

在 Tokenizer 之前对原始文本进行处理, 比如增加, 删除或替换字符等

自带的如下

* HTML Strip 去除 html 标签和转换 html 实体

* Mapping 进行字符替换工作
* Pattern Replace 进行正则匹配替换

会影响后续 tokenizer 解析的 position 和 offset 信息

**Tokenizer**

将原始文本按照一定规则切分为单词(term or token)

自带的如下

* standard 按照单词进行分割
* letter 按照非字符类进行分割
* whitespace 按照空格进行分割
* UAX URL Email 按照 standard 分割, 但不会分割邮箱和 url
* NGram 和 Edge NGram 连词分割
* Path Hierarchy 按照文件路径进行分割

**Token Filters**

对于 tokenizer 输出的单词(term)进行增加, 删除, 修改等操作

自带的如下

* lowercase 将所有 term 转为小写
* stop 删除 stop words
* NGram 和 Edge NGram 连词分割
* Synonym 添加近义词的 term

```json
POST _analyze
{
	"test": "a Hello,World!",
	"tokenizer": "standard",
	"filter": {
		"stop",
		"lowercase":{
			"type": "ngram",
			"min_gram": 4,
			"max_gram": 4
		}
	}
}
```

**自定义分词 Api**

```
PUT /test_index
{
  "settings": {
    "analysis": {
      "char_filter": {
      },
      "tokenizer": {
      },
      "filter": {
      },
      "analyzer": {
      }
    }
  }
}
```

**分词使用说明**

创建或更新文档时(Index Time), 会对相应的文档进行分词处理

查询时(Search Time), 会对查询语句进行分词



## 什么是 Mapping

mapping 类似数据库中的表定义

* 定义 index 下的字段名(field name)
* 定义字段类型, 比如数值型, 字符串型, 布尔型等
* 定义倒排索引相关的配置, 比如是否索引, 记录 position 等

```json
PUT /myindex/article/1 
{
  "post_date": "2018-05-10",
  "title": "Java",
  "content": "java is the best language",
  "author_id": 119
}
```

```json
PUT /myindex/article/2
{
  "post_date": "2018-05-12",
  "title": "html",
  "content": "I like html",
  "author_id": 120
}
```

```json
PUT /myindex/article/3
{
  "post_date": "2018-05-16",
  "title": "es",
  "content": "Es is distributed document store",
  "author_id": 110
}
```

```json
GET /myindex/article/_search?q=2018-05

GET /myindex/article/_search?q=2018-05-10

GET /myindex/article/_search?q=html

GET /myindex/article/_search?q=java
```

**查看 mapping**

```json
GET /myindex/_mapping

{
  "myindex": {
    "mappings": {
      "article": {
        "properties": {
          "author_id": {
            "type": "long"
          },
          "content": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "post_date": {
            "type": "date"
          },
          "title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

**es会自动创建 index，type，以及 type 对应的 mapping, 因为是 es 自动创建的所以叫做动态映射(dynamic mapping)**

**mapping 定义了 type 中的每个字段的数据类型以及这些字段如何分词等相关属性**

创建索引的时候**,可以预先定义字段的类型以及相关属性**，这样就能够把日期字段处理成日期，把数字字段处理成数字，把字符串字段处理字符串值等



### 自定义 Mapping

给索引 lib2 创建映射类型, es 默认会给每一个 field 加上倒排索引

```json
PUT /lib2
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
    "books": {
      "properties": {
        "title": {
          "type": "text", # 默认使用 standard 分词器
          "analyzer": "ik"
        },
        "name": {
          "type": "text",
          "index": false
        },
        "publish_date": {
          "type": "date",
          "index": false
        },
        "price": {
          "type": "double"
        },
        "number": {
          "type": "integer",
          "dynamic":true
        }
      }
    }
  }
}
```

**Mapping 中的字段类型一旦设定后, 禁止直接修改, 因为 Lucene 实现的倒排索引生效后不允许修改**

**如果要修改可以重新建立新的索引, 然后做 reindex 操作**

**允许新增字段类型**

**当 ES 在文档中碰到一个以前没见过的字段时，它会利用动态映射来决定该字段的类型，并自动地对该字段添加映射。**

**可以通过 dynamic 设置来控制这一行为，它能够接受以下的选项：**

```json
true：默认值。允许自动新增字段, 所以我们添加文档时候 es 可以自动建立 mapping
false：不允许自动新增字段, 但是文档可以正常写入, 但无法对字段进行查询等操作
strict：如果碰到陌生字段，抛出异常, 不能写入
```

**dynamic 设置可以适用在根对象上或者object类型的任意字段上。**

```json
PUT /my_index
{
	"mapping": {
		"my_type": {
			"dynamic": false,
			"properties": {
				"user": {
					"properties": {
						"name": {
							"type": "text"
						},
						"social_networks": {
							"dynamic": true,
							"properties": {}
						}
					}
				}
			}
		}
	}
}
```

```json
PUT /myindex
{
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}

GET /myindex/_mapping

PUT /myindex/doc/1
{
	"title": "hello world",
	"desc": "nothing here"
}

GET /myindex/doc/_search
{
	"query": {
		"match": {
			"title": "hello" # 可以查到, 因为定义了 mapping
		}
	}
}

GET /myindex/doc/_search
{
	"query": {
		"match": {
			"desc": "here" # 查不到, 因为定义了 dynamic 是 false, desc 字段没法插入
		}
	}
}
```



### 支持的属性

#### **copy_to解析**

```json
DELETE /myindex

PUT /myindex/article/1
{
  "post_date": "2019-05-10",
  "title": "Java",
  "content": "Java is best language",
  "author_id": 119
}

PUT /myindex/article/2
{
  "post_date": "2019-05-12",
  "title": "html",
  "content": "I like html",
  "author_id": 120
}

PUT /myindex/article/3
{
  "post_date": "2019-05-16",
  "title": "es",
  "content": "es is distributed document store",
  "author_id": 120
}

GET /myindex/_mapping

GET /myindex/article/_search

# 日期类型不分词, 精确匹配
GET /myindex/article/_search?q=post_date:2019-05-16

# 查询 content 字段中含有 html 的
GET /myindex/article/_search?q=content:html

# 因为没有指定具体字段, 所以在全部字段中查询含有 html 或document 的文档, 性能低下
GET /myindex/article/_search?q=html,document
```

**copy_to 字段是把其它字段中的值，以空格为分隔符组成一个大字符串，然后被分析和索引，但是不存储，也就是说它能被查询，但不能被取回显示, 可以提高性能。**

**注意: copy_to 指向的字段字段类型要为：text**

当没有指定 field 时，就会从 copy_to 字段中查询, 如果要使用 copy_to 字段, 需要自己创建 mapping

```json
DELETE /myindex

PUT /myindex

PUT /myindex/article/_mapping
{
	"properties": {
		"post_date": {
			"type": "date"
		},
		"title": {
			"type": "text",
			"copy_to": "fullcontents"
		},
		"content": {
			"type": "text",
			"copy_to": "fullcontents"
		},
		"author_id": {
			"type": "integer"
		},
		"fullcontents": {
			"type": "text"
		}
	}
}

GET /myindex/article/_search
{
	"query": {
		"match": {
			"fullcontents": {
				"query": "xxxx"
			}
		}
	}
}
```

#### index 属性

控制当前字段是否索引, 默认为 true, 即记录索引, false 不记录, 即不可搜索

```json
PUT /my_inde
{
	"mappings": {
		"doc": {
			"properties": {
				"cookie": {
					"type": text,
					"index": false
				}
			}
		}
	}
}
```

#### index_options 属性

index_options 用于控制倒排索引记录的内容, 有如下 4 中配置

**docs(索引文档号):** 只记录 doc id

**freqs(文档号+词频):** 记录 doc id 和 term frequencies

**positions(文档号+词频+位置，通常用来距离查询):** 记录 doc id, term frequencies 和 term position

**offsets(文档号+词频+位置+偏移量，通常被使用在高亮字段):** 记录 doc id, term frequencies, term position 和 character offsets

分词字段(text)默认是positions，其他的默认是docs, 记录内容越多, 占用空间越多



#### null_value 属性

当字段遇到 null 值时的处理策略, 默认为 null, 即空值, 此时 es 会忽略该值. 可以通过设定该值设定字段的默认值, 只有string可以使用，分词字段的null值也会被分词



#### 其他属性

`"store": false` // 是否单独设置此字段的是否存储而从 _source 字段中分离，默认是 false，只能搜索，不能获取值

`"analyzer":"ik"` // 指定分词器,默认分词器为 standard analyzer

`"boost":1.23` // 字段级别的分数加权，默认值是1.0

`"doc_values":false` // 对 not_analyzed 字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存

`"fielddata":{"format":"disabled"}` // 针对分词字段，参与排序或聚合时能提高性能，不分词字段统一建议使用 doc_value

`"fields":{"xxxxx":{"type":"text","index":"not_analyzed"}}` // 可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词

`"ignore_above":100` // 超过 100 个字符的文本，将会被忽略，不被索引

`"include_in_all":ture` // 设置是否此字段包含在 _all 字段中，默认是 true，除非 index 设置成 no选项

`"norms":{"enable":true,"loading":"lazy"}` // 分词字段默认配置，不分词字段：默认{"enable":false}，存储长度因子和索引时 boost，建议对需要参与评分字段使用 ，会额外增加内存消耗量

`"position_increament_gap":0` // 影响距离查询或近似查询，可以设置在多值字段的数据上火分词字段上，查询时可指定 slop间隔，默认值是100

`"search_analyzer":"ik"` // 设置搜索时的分词器，默认跟 ananlyzer 是一致的，比如 index 时用 standard+ngram，搜索时用 standard 用来完成自动提示功能

`"similarity":"BM25"` // 默认是 TF/IDF 算法，指定一个字段评分策略，仅仅对字符串型和分词类型有效

`"term_vector":"no"` // 默认不存储向量信息，支持参数 yes（term存储），with_positions（term+位置）,with_offsets（term+偏移量), with_positions_offsets(term+位置+偏移量) 对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html



### **支持的数据类型**

1. 核心数据类型（Core datatypes）

    ```yaml
    字符型：string, 包括 text 和 keyword
    
    text: 类型被用来索引长文本，在建立索引前默认会将这些文本进行分词，转化为词的组合，建立索引。允许es来检索这些词语。text类型不能用来排序和聚合。
    
    Keyword: 类型不进行分词, 会进行索引, 可以被用来检索过滤、排序和聚合。keyword 类型字段只能用本身来进行检索
    
    数字型：long, integer, short, byte, double, float, half_float, scaled_float 默认会建倒排索引, 但是没有分词, 所以只能精确匹配
    日期型：date 默认会建倒排索引, 但是没有分词, 所以只能精确匹配
    布尔型：boolean
    二进制型：binary
    范围类型: integer_range, float_range, long_range, double_range, date_range
    ```

2. 复杂数据类型（Complex datatypes）

    ```yaml
    数组类型（Array datatype）：数组类型不需要专门指定数组元素的type，例如：
        字符型数组: ["one", "two"]
        整型数组：[1, 2]
        数组型数组：[1, [2, 3]] 等价于[1, 2, 3]
        对象数组：[{"name": "Mary", "age": 12}, {"name": "John", "age": 10}]
    对象类型（Object datatype）：_object_ 用于单个JSON对象；
    嵌套类型（Nested datatype）：_nested_ 用于JSON数组
    ```

    对象类型:

    ```json
    # 对象类型底层结构
    PUT /lib5/person/1
    {
      "name": "Tom",
      "age": 25,
      "birthday": "1985-12-12",
      "address": {
        "country": "china",
        "province": "beijing",
        "city": "beijing"
      }
    }
    
    {
      "name": ["Tom"],
      "age": [25],
      "birthday": ["1985-12-12"],
      "address.country": ["china"],
      "address.province": ["beijing"],
      "address.city": ["beijing"]
    }
    
    # 集合类型底层结构
    PUT /lib6/person/1
    {
      "persons": [
        {"name", "zhangsan", "age": 20},
        {"name", "lisi", "age": 25},
        {"name", "wangwu", "age": 30}
      ]
    }
    
    {
      "persons.name": ["zhangsan", "lisi", "wangwu"],
    	"persons.age": [20, 25, 30]
    }
    ```

    

3. 地理位置类型（Geo datatypes）

    ```yaml
    地理坐标类型（Geo-point datatype）：_geo_point_ 用于经纬度坐标；
    地理形状类型（Geo-Shape datatype): _geo_shape_ 用于类似于多边形的复杂形状；
    ```

4. 特定类型（Specialised datatypes）

    ```yaml
    IPv4 类型（IPv4 datatype）：_ ip _ 用于IPv4 地址；
    Completion 类型（Completion datatype）：_ completion _提供自动补全建议；
    Token count 类型（Token count datatype）：_ token_count _ 用于统计做了标记的字段的index数目，该值会一直增加，不会因为过滤条件而减少。
    mapper-murmur3
    类型：通过插件，可以通过 _ murmur3 _ 来计算 index 的 hash 值；
    附加类型（Attachment datatype）：采用 mapper-attachments
    插件，可支持_ attachments _ 索引，例如 Microsoft Office 格式，Open Document 格式，ePub, HTML 等。
    ```




### Dynamic Mapping

es 可以自动识别文档字段类型, 从而降低用户的使用成本

```json
DELETE /test_index

PUT /test_index/doc/1
{
	"username": "alfred",
	"age": 1
}

GET /test_index/_mapping

{
  "test_index" : {
    "mappings" : {
      "doc" : {
        "properties" : {
          "age" : {
            "type" : "long" # es 自动识别 age 为 long 类型
          },
          "username" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

es 是依靠 json 文档的字段类型实现自动识别字段类型, 支持的类型如下:

| JSON 类型 |                           es 类型                            |
| :-------: | :----------------------------------------------------------: |
|   null    |                             忽略                             |
|  boolean  |                           boolean                            |
| 浮点类型  |                            float                             |
|   整数    |                             long                             |
|  object   |                            object                            |
|   array   |                 由第一个非 null 值的类型决定                 |
|  string   | 匹配为日期则设为 date 类型(默认开启)<br />匹配为数字类型的话设为 float 或 long 类型(默认关闭)<br />如果不是以上两种就设为 text 类型, 并附带 keyword 的子字段 |

```json
DELETE /test_index

PUT /test_index/doc/1
{
	"username": "alfred",
	"age": 14,
	"birth": "1998-10-10",
	"married": false,
	"year": "18",
	"tags": ["boy", "fashion"],
	"money": 100.1
}

GET /test_index/_mapping
{
  "test_index" : {
    "mappings" : {
      "doc" : {
        "properties" : {
          "age" : {
            "type" : "long"
          },
          "birth" : {
            "type" : "date"
          },
          "married" : {
            "type" : "boolean"
          },
          "money" : {
            "type" : "float"
          },
          "tags" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "username" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "year" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

#### 日期格式的自动识别

**日期的识别可以自行配置日期格式, 以满足各种需求**

默认是 `["strict_date_optional_time","yyyy/MM/dd HH:mm:ss||yyyy/MM/dd Z"]`

strict_date_optional_time 是 ISO datetime 的格式, 完整格式类似下面:

YYYY-MM-DDThh:mm:ssTZD(eg 1997-07-16T19:20:30+01:00)

dynamic_date_formats 可以自定义日期类型

date_detection 可以关闭日期自动识别的机制

```json
DELETE /my_index

PUT /my_index
{
	"mappings": {
		"my_type":{
			"dynamic_date_formats": ["MM/dd/yyyy"]
		}
	}
}

PUT /my_index/doc/1
{
	"create_date": "09/25/2015" # 默认不会识别为日期, 因为格式错误
}

GET /my_index/_mapping
```

#### 数字格式的自动识别

**字符串是数字时, 默认不会自动识别为整形, 因为字符串中出现数字是完全合理的**

**numeric_detection 可以开启字符串中数字的自动识别**

```json
DELETE /my_index

PUT /my_index
{
	"mappings": {
		"my_type": {
			"numeric_detection": true
		}
	}
}

PUT /my_index/my_type/1
{
	"my_float": "1.0",
	"my_integer": "1"
}

GET /my_index/_mapping
```

#### Dynamic Mapping Templates

允许根据 es 自动识别的数据类型, 字段名等来动态设定字段类型, 可以实现如下效果

* 所有字符串类型都设定为 keyword 类型, 即默认不分词
* 所有以 message 开头的字段都设定为 text 类型, 即分词
* 所有以 long_ 开头的字段都设定为 long 类型
* 所有自动匹配为 double 类型的都设定为 float 类型, 以节省空间

```json
DELETE /my_index

PUT /my_index
{
	"mappings": {
		"doc": {
			"dynamic_templates": [ # 动态模板数组, 可以指定多个匹配规则, 先匹配到的先生效
				{
					"strings": { # template 的名称, 随意指定
						"match_mapping_type": "string", # 匹配规则
            "mapping": { # 设置 mapping 信息
            	"type": "keyword"
             }
					}
				}
      ]
		}
	}
}
```

**匹配规则一般有如下几个参数:**

match_mapping_type: 匹配 es 自动识别的类型, 如 boolean, long, string 等

match, unmatch: 匹配字段名, 如 `*_en: 以 en 结尾的字段`

path_match, path_unmatch: 匹配路径



**字符串默认使用 keyword 类型**

es 默认会为字符串设置为 text 类型, 并增加一个 keyword 的子字段, 我们现在要修改默认 string 全变为 keyword 类型

```json
DELETE /my_index

PUT /my_index
{
	"mappings": {
		"doc": {
			"dynamic_templates": [
				{
					"strings_as_keyword": {
						"match_mapping_type": "string",
						"mapping": {
							"type": "keyword"
						}
					}
				}
			]
		}
	}
}

PUT /my_index/doc/1
{
	"name": "alfred"
}

GET /my_index/_mapping
```



**以 message 开头的字段都设置为 text 类型, 其余的 string 类型的字段设置为 keyword 类型**

```json
DELETE /my_index

PUT /my_index
{
	"mappings": {
		"doc": {
    	"dynamic_templates": [
        {
          "message_as_text": {
            "match_mapping_type": "string",
            "match": "message*",
            "mapping": {
              "type": "text"
            }
          }
        },
        {
					"strings_as_keyword": {
						"match_mapping_type": "string",
						"mapping": {
							"type": "keyword"
						}
					}
				}
      ]
    }
	}
}

PUT /my_index/doc/1
{
  "title": "学好 java",
  "message_xxx": "哈哈哈哈哈哈"
}

GET /my_index/_mapping
```



https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html

### 自定义 Mapping 的建议

自定义 Mapping 的操作步骤如下

1. 写入一条文档到 es 的临时索引中, 获取 es 自动生成的 mapping
2. 修改步骤 1 得到的 mapping, 自定义相关配置
3. 使用步骤 2 的 mapping 创建实际所需索引

```json
# 临时索引
DELETE /test_index
PUT /test_index/doc/1
{
  "referrer": "-",
  "response_code": "200",
  "remote_ip": "171.221.139.157",
  "method": "POST",
  "user_name": "-",
  "http_version": "1.1",
  "body_sent": {
    "bytes": "0"
  },
  "url": "/analyzeVideo"
}

GET /test_index/_mapping

# 修改临时索引的 mapping 设置, 创建实际要插入的索引
DELETE /my_product_index
PUT /my_product_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "strings": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ],
      "properties": {
        "body_sent": {
          "properties": {
            "bytes": {
              "type": "long"
            }
          }
        },
        "url": {
          "type": "text"
        }
      }
    }
  }
}

PUT /my_product_index/doc/1
{
  "referrer": "-",
  "response_code": "200",
  "remote_ip": "171.221.139.157",
  "method": "POST",
  "user_name": "-",
  "http_version": "1.1",
  "body_sent": {
    "bytes": "0"
  },
  "url": "/analyzeVideo"
}

GET /my_product_index/_mapping
```

### 索引模板

索引模板, 英文名称 Index Template, **主要用于在新建索引时自动应用预先设定的配置**

简化索引创建的操作步骤

* 可以设定索引的配置和 mapping
* 可以有多个模板, 根据 order 设置, order 大的覆盖小的配置

索引模板 API, endpoint 为 _template

```json
PUT /_template/test_template # template 的名称
{
  # 匹配的索引名称
	"index_patterns": ["te*", "bar*"],
  # order 的顺序配置, order 越大优先级越高
	"order": 0,
	# 索引的配置
	"settings": {
		"number_of_shards": 1
	},
	"mappings": {
		"doc": {
			"_source": {
        # 不记录原始数据
				"enabled": false
			},
			"properties": {
				"name": {
					"type": "keyword"
				}
			}
		}
	}
}

DELETE /test_index
PUT /test_index
GET /test_index/_mapping

# 查看所有索引模板
GET _template
# 查看指定名称的索引模板
GET _template/test_template
# 删除索引模板
DELETE _template/test_template
```



## 使用 Kibana 进行操作

### 索引操作

**添加索引, 指定配置信息**

```json
PUT /lib/
{
  "settings": {
    "index": {
      # 默认 5
      "number_of_shards": 3,
      # 默认 1
      "number_of_replicas": 0
    }
  }
}
```

**number_of_shards: 分片, 确定后不能修改**

**number_of_replicas: 备份数量**

**使用默认配置添加索引**

```
PUT lib2
```

**删除索引**

```
DELETE /lib
```

**查看现有索引**

```
GET _cat/indices
```

**查看索引配置信息**

```
GET /lib/_settings
GET /lib2/_settings
```

**查看所有索引配置信息**

```
GET /_all/_settings
```

### 文档操作

**添加文档, 指定 id 为 1, 使用 PUT 方式, 如果索引不存在, es 会自动创建对应的 index 和 type**

```json
PUT /lib/user/1
{
  "first_name": "Jane",
  "last_name": "Smith",
  "age": 32,
  "about": "I like to collect rock albums",
  "interests": [
    "music"
  ]
}
```

**添加文档, 随机分配 id, 使用 POST 方式**

```json
POST /lib/user/
{
  "first_name": "Douglas",
  "last_name": "Fir",
  "age": 23,
  "about": "I like to build cabinets",
  "interests": [
    "forestry"
  ]
}
```

**查看文档**

```yaml
GET /lib/user/1
```

```yaml
GET /lib/user/hS72qW0B7Wijl9prHIPm
```

**查看文档, 指定查看的 field**

```yaml
GET /lib/user/1?_source=age,interests
```

**更新文档, 使用覆盖的方式, 相当于删除从建, id 需要相同**

```json
PUT /lib/user/1
{
  "first_name": "Jane",
  "last_name": "Smith",
  "age": 36,
  "about": "I like to collect rock albums",
  "interests": [
    "music"
  ]
}
```

**更新文档, 使用修改的方式, id 需要相同**

```json
POST /lib/user/1/_update
{
  "doc": {
    "age": 33
  }
}
```

**删除一个文档**

```
DELETE /lib/user/1
```

### 使用 Multi Get API 批量获取文档

使用es提供的Multi Get API：

使用Multi Get API可以通过索引名, 类型名, 文档id一次得到一个文档集合，**文档可以来自同一个索引库，也可以来自不同索引库**

**使用 curl 的方式**

```bash
curl 'http://localhost:9200/_mget' -d '{
  "docs"：[
    {
      "_index": "lib",
      "_type": "user",
      "_id": 1
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id": 2
    }
  ]
}'
```

**使用 kibana 的方式**

```json
GET /_mget
{
  "docs": [
    {
      "_index": "lib",
      "_type": "user",
      "_id": 1
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id": 2
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id": 3
    }
  ]
}

可以指定具体的字段：

GET /_mget
{
  "docs": [
    {
      "_index": "lib",
      "_type": "user",
      "_id": 1,
      "_source": "interests"
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id": 2,
      "_source": [
        "age",
        "interests"
      ]
    },
    {
      "_index": "lib",
      "_type": "user",
      "_id": 3
    }
  ]
}

获取同索引同类型下的不同文档, 上边的简化写法

GET /lib/user/_mget
{
  "docs": [
    {
      "_id": 1
    },
    {
      "_id": 2
    }
  ]
}

GET /lib/user/_mget
{
  "ids": ["1","2"]
}
```

### 使用Bulk API实现批量操作

**bulk的格式**

```json
{action:{metadata}}\n
{requstbody}\n

action:(行为)
  create: 文档不存在时创建
  update: 更新文档
  index: 创建新文档或替换已有文档
  delete: 删除一个文档

metadata：_index,_type,_id

create 和 index 的区别
  如果数据存在，使用 create 操作失败，会提示文档已经存在，使用index则可以成功执行。

示例：
{"delete":{"_index":"lib","_type":"user","_id":"1"}}
```

**批量添加**

```json
POST /lib2/books/_bulk

{"index": {"_id": 1}}
{"title": "Java", "price": 55}

{"index": {"_id": 2}}
{"title": "Html5", "price": 45}

{"index": {"_id":3}}
{"title": "Php", "price": 35}

{"index": {"_id": 4}}
{"title": "Python", "price": 50}
```

**批量获取**

```json
GET /lib2/books/_mget
{
	"ids": ["1","2","3","4"]
}
```

**删除, 没有请求体**

```json
POST /lib2/books/_bulk

{"delete": {"_index": "lib2", "_type": "books", "_id": 4}}

{"create": {"_index": "tt", "_type": "ttt", "_id": "100"}}
{"name": "lisi"}

{"index": {"_index": "tt", "_type": "ttt"}}
{"name": "zhaosi"}

{"update": {"_index": "lib2", "_type": "books", "_id": "4"}}
{"doc": {"price": 58}}
```

bulk一次最大处理多少数据量:

　　bulk会把将要处理的数据载入内存中，所以数据量是有限制的，最佳的数据量不是一个确定的数值，它取决于你的硬件，你的文档大小以及复杂性，你的索引以及搜索的负载。

　　一般建议是1000-5000个文档，大小建议是5-15MB，默认不能超过100M，可以在 es 的配置文件（即$ES_HOME下的config下的elasticsearch.yml）中。

## ElasticSearch查询

实现对 es 中存储的数据进行查询分析, endpoint 为 _search

```json
GET /_search
GET /my_index/_search
GET /my_index1,my_index2/_search
GET /my_*/_search
```

查询主要有两种形式

* URI Search

  操作简便, 方便通过命令行做测试

  仅包含部分查询语法

  ```json
  GET /my_index/_search?q=user:alfred
  ```

* Request Body Search

  es 提供的完备查询语法 Query DSL(Domain Specific Language)

  ```json
  GET /my_index/_search
  {
  	"query": {
  		"term": {
  			"name": ["user", "alfred"]
  		}
  	}
  }
  ```



### URI Search

通过 url query 参数来实现搜索, 常用参数如下

* q 指定查询语句, 语法为 Query String Syntax
* df q 中不指定字段时默认查询的字段, 如果不指定, es 会查询所有字段
* sort 排序
* timeout 指定超时时间, 默认不超时
* from, size 分页, 从 0 开始

查询 user 字段包含 alfred 的文档, 结果按照 age 升序排序, 返回第 5~14 个文档, 如果超过 1s 没有结束, 则以超时结束

```
GET /my_index/_search?q=alfred&df=user&sort=age:asc&from=4&size=10&timeout=1s
```

#### Query String Syntax

* term 与 phrase

  q=alfred way 等效于 alfred OR way

  q="alfred way" 词语查询, 要求先后顺序

* 泛查询

  q=alfred 等效于在所有字段中

* 指定字段

  q=name:alfred

* Group 分组设定, 使用括号指定匹配的规则

  (quick OR brown) AND fox

  q=status:(active OR pending) title:(full text search), 如果不加括号就是查询 status 是 active 或者在所有字段中查找是 pending 的文档

* 布尔操作符

  AND(&&), OR(||), NOT(!)

  q=name:(tom NOT lee)

  注意大写, 不能小写

  \+ - 分别对应 must 和 must_not

  q=name:(tom +lee -alfred)

  q=name:((lee && !alfred) || (tom && lee && !alfred))

  \+ 在 url 中会被解析为空格, 要使用 encode 后的结果才可以, 为 %2B

  布尔查询可以理解为先分词进行查询, 再将布尔条件组合起来筛选符合条件的文档

* 范围查询

  区间写法, 必须见用[], 开区间用{}

  q=age[1 TO 10], 1 <= age <= 10

  q=age[1 TO 10}, 1 <= age < 10

  q=age[1 TO ], age >= 1

  q=age[* TO 10], age <= 10

  算数符号写法

  q=age: >=1

  q=age:(>=1 && <=10) 或者 age:(+ >=1 + <=10)

* 通配符

  ? 代表 1 个字符, * 代表 0 个或多个字符

  q=name:t?m

  q=name:tom*

  q=name:t*m

  通配符匹配执行效率低, 且占用内存多, 不建议使用

  如无特殊需求, 不要将 ?/* 放在最前面

* 正则表达式

  q=name:/[mb]oat/

* 模糊匹配 fuzzy query

  name:roam~1

  匹配与 roam 差一个character 的词, 比如 foam, roams 等

* 近似度查询 proximity search

  "fox quick"~5

  以 term 为单位进行差异比较, 比如 "quick fox" "quick brown fox" 都会被匹配

```json
DELETE test_search_index

PUT test_search_index
{
  "settings": {
    "index":{
        "number_of_shards": "1"
    }
  }
}

POST test_search_index/doc/_bulk
{"index":{"_id":"1"}}
{"username":"alfred way","job":"java engineer","age":18,"birth":"1990-01-02","isMarried":false}
{"index":{"_id":"2"}}
{"username":"alfred","job":"java senior engineer and java specialist","age":28,"birth":"1980-05-07","isMarried":true}
{"index":{"_id":"3"}}
{"username":"lee","job":"java and ruby engineer","age":22,"birth":"1985-08-07","isMarried":false}
{"index":{"_id":"4"}}
{"username":"alfred junior way","job":"ruby engineer","age":23,"birth":"1989-08-07","isMarried":false}

# 泛查询(不指定查询字段), 在所有字段中查找包含 alfred 的文档
GET /test_search_index/_search?q=alfred
# 查看真正执行的查询语句
GET /test_search_index/_search?q=alfred
{
	"profile": true
}
# 指定字段查询
GET /test_search_index/_search?q=username:alfred

# term 查询, 相当于 username:alfred OR 在所有字段中查找包含 way 的文档, 从左到右分组
GET /test_search_index/_search?q=username:alfred way

# term 查询真正执行的语句
GET /test_search_index/_search?q=username:alfred way
{
  "profile": "true"
}

# phrase 查询, 相当于在 username 字段中查找 alfred way 这个词语
GET /test_search_index/_search?q=username:"alfred way"

GET /test_search_index/_search?q=username:"alfred way"
{
  "profile": "true"
}

# group 查询
GET /test_search_index/_search?q=username:(alfred way)

# 对日期范围查询
GET /test_search_index/_search?q=birth:(>1980 AND <1999)
```





### 基本查询(Query查询)(英文)

#### 数据准备

```json
DELETE /lib3

PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {"type":"integer"},
            "interests": {"type":"text"},
            "birthday": {"type":"date"}
        }
      }
     }
}

PUT /lib3/user/1
{
	"name": "zhaoliu",
	"address": "hei long jiang sheng tie ling shi",
	"age": 50,
	"birthday": "1970-12-12",
	"interests": "xi huan hejiu,duanlian,lvyou"
}

PUT /lib3/user/2
{
	"name": "zhaoming",
	"address": "bei jing hai dian qu qing he zhen",
	"age": 20,
	"birthday": "1998-10-12",
	"interests": "xi huan hejiu,duanlian,changge"
}

PUT /lib3/user/3
{
	"name": "lisi",
	"address": "bei jing hai dian qu qing he zhen",
	"age": 23,
	"birthday": "1998-10-12",
	"interests": "xi huan hejiu,duanlian,changge"
}

PUT /lib3/user/4
{
	"name": "wangwu",
	"address": "bei jing hai dian qu qing he zhen",
	"age": 26,
	"birthday": "1995-10-12",
	"interests": "xi huan biancheng,tingyinyue,lvyou"
}

PUT /lib3/user/5
{
	"name": "zhangsan",
	"address": "bei jing chao yang qu",
	"age": 29,
	"birthday": "1988-10-12",
	"interests": "xi huan biancheng,tingyinyue,tiaowu"
}
```

#### 简单查询

```json
GET /lib3/user/_search?q=name:lisi

GET /lib3/user/_search?q=interests:changge&sort=age:desc



{
  "took" : 2, # 耗时 2 毫秒
  "timed_out" : false, # 没有超时
  "_shards" : {
    "total" : 3, # 向 3 个分片都发送了请求
    "successful" : 3, # 3 个分片都成功了
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1, # 命中几条记录
    "max_score" : 0.6931472, # 相关度匹配, 最大是 1, 由 es 计算
    "hits" : [
      {
        "_index" : "lib3",
        "_type" : "user",
        "_id" : "3",
        "_score" : 0.6931472, # 相关度匹配, 最大是 1, 由 es 计算
        "_source" : {
          "name" : "lisi",
          "address" : "bei jing hai dian qu qing he zhen",
          "age" : 23,
          "birthday" : "1998-10-12",
          "interests" : "xi huan hejiu,duanlian,changge"
        }
      }
    ]
  }
}
```

#### term查询和terms查询

term query 会去倒排索引中寻找确切的 term，**它并不知道分词器的存在即搜索词不进行分词**。这种查询适合 keyword 、numeric、date。

term: 查询某个字段里含有某个关键词的文档

```json
GET /lib3/user/_search/
{
  "query": {
    "term": {
      "name": "zhaoliu"
    }
  }
}
```

terms:查询某个字段里含有多个关键词的文档

```json
GET /lib3/user/_search
{
  "query": {
    "terms": {
      "interests": [
        "hejiu",
        "changge"
      ]
    }
  }
}
```

#### 控制查询返回的数量

from：从哪一个文档开始
size：需要的个数

```json
GET /lib3/user/_search
{
  "from": 0,
  "size": 2,
  "query": {
    "terms": {
      "interests": [
        "hejiu",
        "changge"
      ]
    }
  }
}
```

#### 返回版本号

```json
GET /lib3/user/_search
{
  "version": true,
  "query": {
    "terms": {
      "interests": [
        "hejiu",
        "changge"
      ]
    }
  }
}
```

#### match查询

**match query知道分词器的存在，会对搜索词进行分词操作，然后再查询**

```json
GET /lib3/user/_search
{
  "query": {
    "match": {
      "name": "zhaoliu"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "match": { # 这里进行分词会查到 2 条, 如果使用 term 就不会进行分词, 就查不到
      "name": "zhaoliu zhaoming"
    }
  }
}
```

**match_all:查询所有文档**

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  }
}
```

**multi_match:可以指定多个字段进行查询, term 和 match 只能指定一个字段**

```json
GET /lib3/user/_search
{
  "query": {
    "multi_match": {
      "query": "changge",
      "fields": [
        "interests",
        "name"
      ]
    }
  }
}
```

**match_phrase:短语匹配查询**

ElasticSearch引擎首先分析（analyze）查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短语中的所有分词，并且保证各个分词的相对位置不变：

```json
GET lib3/user/_search
{
  "query": {
    "match_phrase": {
      "interests": "duanlian，shuoxiangsheng"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "match_phrase": {
      "interests": {
        "query": "hejiu, lvyou",
        "slop": "1" # 允许短语间有 1 个其他词语
      }
    }
  }
}
```

#### 指定返回的字段

```json
GET /lib3/user/_search
{
  "_source": [
    "address",
    "name"
  ],
  "query": {
    "match": {
      "interests": "changge"
    }
  }
}
```

#### 控制加载的字段

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "_source": {
    "includes": [
      "name",
      "address"
    ],
    "excludes": [
      "age",
      "birthday"
    ]
  }
}
```

**使用通配符\***

```json
GET /lib3/user/_search
{
  "_source": {
    "includes": "addr*",
    "excludes": [
      "name",
      "bir*"
    ]
  },
  "query": {
    "match_all": {
    }
  }
}
```

#### 排序

使用sort实现排序: desc:降序，asc升序

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}

GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}
```

#### 前缀匹配查询

```json
GET /lib3/user/_search
{
  "query": {
    "match_phrase_prefix": {
      "name": {
        "query": "zhao"
      }
    }
  }
}
```

#### 范围查询

range:实现范围查询, 主要针对数值和日期类型

参数：gt, gte, lt, lte

```json
GET /lib3/user/_search
{
  "query": {
    "range": {
      "birthday": {
        "gte": "1990-10-10"
      }
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 25
      }
    }
  }
}
```

#### wildcard查询

允许使用通配符* 和 ?来进行查询

\* 代表0个或多个字符

? 代表任意一个字符

```json
GET /lib3/user/_search
{
  "query": {
    "wildcard": {
      "name": "zhao*"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "wildcard": {
      "name": "li?i"
    }
  }
}
```



#### fuzzy实现模糊查询

value：查询的关键字

boost：查询的权值，默认值是1.0

min_similarity:设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1(包括0和1);对于数值，取值可能大于1;对于日期型取值为1d,1m等，1d就代表1天

prefix_length:指明区分词项的共同前缀长度，默认是0

max_expansions:查询中的词项可以扩展的数目，默认可以无限大

```json
GET /lib3/user/_search
{
  "query": {
    "fuzzy": {
      "name": "zholiu"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "fuzzy": {
      "interests": {
        "value": "chagge"
      }
    }
  }
}
```



#### 高亮搜索结果

```json
GET /lib3/user/_search
{
  "query": {
    "match": {
      "interests": "changge"
    }
  },
  "highlight": {
    "fields": {
      "interests": {
      }
    }
  }
}
```



### 基本查询(Query查询)(中文)

ik 带有两个分词器

ik_max_word: 会将文本做最细粒度的拆分; 尽可能多的拆分出词语

ik_smart: 会做最粗粒度的拆分; 已被分出的词语将不会在被其他词语占有

#### 数据准备

```json
DELETE /lib3

PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text", "analyzer": "ik_max_word"},
            "address": {"type":"text", "analyzer": "ik_max_word"},
            "age": {"type":"integer"},
            "interests": {"type":"text", "analyzer": "ik_max_word"},
            "birthday": {"type":"date"}
        }
      }
     }
}
PUT /lib3/user/1
{
	"name": "赵六",
	"address": "黑龙江省铁岭",
	"age": 50,
	"birthday": "1970-12-12",
	"interests": "喜欢喝酒,锻炼,说相声"
}

PUT /lib3/user/2
{
	"name": "赵明",
	"address": "北京海淀区清河",
	"age": 20,
	"birthday": "1998-10-12",
	"interests": "喜欢喝酒,锻炼,唱歌"
}

PUT /lib3/user/3
{
	"name": "lisi",
	"address": "北京海淀区清河",
	"age": 23,
	"birthday": "1998-10-12",
	"interests": "喜欢喝酒,锻炼,唱歌"
}

PUT /lib3/user/4
{
	"name": "王五",
	"address": "北京海淀区清河",
	"age": 26,
	"birthday": "1995-10-12",
	"interests": "喜欢编程,听音乐,旅游"
}

PUT /lib3/user/5
{
	"name": "张三",
	"address": "北京海淀区清河",
	"age": 29,
	"birthday": "1988-10-12",
	"interests": "喜欢摄影,听音乐,跳舞"
}
```

#### term查询和terms查询

term query会去倒排索引中寻找确切的term，**它并不知道分词器的存在即搜索词不进行分词**。这种查询适合keyword 、numeric、date。

term:查询某个字段里含有某个关键词的文档

```json
GET /lib3/user/_search/
{
  "query": {
    "term": {
      "name": "赵"
    }
  }
}
```

terms:查询某个字段里含有多个关键词的文档

```json
GET /lib3/user/_search
{
  "query": {
    "terms": {
      "interests": ["喝酒","唱歌"]
    }
  }
}
```

#### 控制查询返回的数量

from：从哪一个文档开始
size：需要的个数

```json
GET /lib3/user/_search
{
  "from": 0,
  "size": 2,
  "query": {
    "terms": {
      "interests": ["喝酒","唱歌"]
    }
  }
}
```

#### 返回版本号

```json
GET /lib3/user/_search
{
  "version": true,
  "query": {
    "terms": {
      "interests": ["喝酒","唱歌"]
    }
  }
}
```

#### match查询

**match query知道分词器的存在，会对搜索词进行分词操作，然后再查询**

```json
GET /lib3/user/_search
{
  "query": {
    "match": {
      "name": "赵六"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "match": { # 这里进行分词会查到 2 条, 如果使用 term 就不会进行分词, 就查不到
      "name": "赵六赵明"
    }
  }
}
```

**match_all:查询所有文档**

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  }
}
```

**multi_match:可以指定多个字段进行查询, term 和 match 只能指定一个字段**

```json
GET /lib3/user/_search
{
  "query": {
    "multi_match": {
      "query": "唱歌",
      "fields": [
        "interests",
        "name"
      ]
    }
  }
}
```

**match_phrase:短语匹配查询**

ElasticSearch引擎首先分析（analyze）查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短语中的所有分词，并且保证各个分词的相对位置不变：

```json
GET lib3/user/_search
{
  "query": {
    "match_phrase": {
      "interests": "锻炼,说相声"
    }
  }
}
```

#### 指定返回的字段

```json
GET /lib3/user/_search
{
  "_source": [
    "address",
    "name"
  ],
  "query": {
    "match": {
      "interests": "唱歌"
    }
  }
}
```

#### 控制加载的字段

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "_source": {
    "includes": [
      "name",
      "address"
    ],
    "excludes": [
      "age",
      "birthday"
    ]
  }
}
```

**使用通配符\***

```json
GET /lib3/user/_search
{
  "_source": {
    "includes": "addr*",
    "excludes": [
      "name",
      "bir*"
    ]
  },
  "query": {
    "match_all": {
    }
  }
}
```

#### 排序

使用sort实现排序: desc:降序，asc升序

```json
GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}

GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}
```

#### 前缀匹配查询

```json
GET /lib3/user/_search
{
  "query": {
    "match_phrase_prefix": {
      "name": {
        "query": "赵"
      }
    }
  }
}
```

#### 范围查询

range:实现范围查询

参数：from,to,include_lower,include_upper,boost

include_lower:是否包含范围的左边界，默认是true

include_upper:是否包含范围的右边界，默认是true

```json
GET /lib3/user/_search
{
  "query": {
    "range": {
      "birthday": {
        "from": "1990-10-10",
        "to": "2018-05-01"
      }
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "range": {
      "age": {
        "from": 20,
        "to": 25,
        "include_lower": true,
        "include_upper": false
      }
    }
  }
}
```

#### wildcard查询

允许使用通配符* 和 ?来进行查询

\* 代表0个或多个字符

? 代表任意一个字符

```json
GET /lib3/user/_search
{
  "query": {
    "wildcard": {
      "name": "赵*"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "wildcard": {
      "name": "李?"
    }
  }
}
```



#### fuzzy实现模糊查询

value：查询的关键字

boost：查询的权值，默认值是1.0

min_similarity:设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1(包括0和1);对于数值，取值可能大于1;对于日期型取值为1d,1m等，1d就代表1天

prefix_length:指明区分词项的共同前缀长度，默认是0

max_expansions:查询中的词项可以扩展的数目，默认可以无限大

```json
GET /lib3/user/_search
{
  "query": {
    "fuzzy": {
      "name": "赵"
    }
  }
}

GET /lib3/user/_search
{
  "query": {
    "fuzzy": {
      "interests": {
        "value": "喝酒"
      }
    }
  }
}
```



#### 高亮搜索结果

```json
GET /lib3/user/_search
{
  "query": {
    "match": {
      "interests": "唱歌"
    }
  },
  "highlight": {
    "fields": {
      "interests": {
      }
    }
  }
}
```







### Filter查询

filter是不计算相关性的，同时可以cache。因此，filter速度要快于query。

```json
POST /lib4/items/_bulk
{"index": {"_id": 1}}
{"price": 40,"itemID": "ID100123"}
{"index": {"_id": 2}}
{"price": 50,"itemID": "ID100124"}
{"index": {"_id": 3}}
{"price": 25,"itemID": "ID100124"}
{"index": {"_id": 4}}
{"price": 30,"itemID": "ID100125"}
{"index": {"_id": 5}}
{"price": null,"itemID": "ID100127"}
```

#### 简单的过滤查询

```json
GET /lib4/items/_search
{
  "post_filter": {
    "term": {
      "price": 40
    }
  }
}

GET /lib4/items/_search
{
  "post_filter": {
    "terms": {
      "price": [25, 40]
    }
  }
}

GET /lib4/items/_search
{
  "post_filter": {
    "term": {
      "itemID": "ID100123" # 默认分词会转成小写, 所以查不到
    }
  }
}
```

查看分词器分析的结果：

```
GET /lib4/_mapping
```

不希望商品 itemID 字段被分词，则重新创建映射

```
DELETE lib4
```

```json
PUT /lib4
{
  "mappings": {
    "items": {
      "properties": {
        "itemID": {
          "type": "text",
          "index": false
        }
      }
    }
  }
}
```



#### bool过滤查询

可以实现组合过滤查询

格式：

```json
{
  "bool": {
    "must": [ # 数据库中的 and
    ],
    "should": [ # or
    ],
    "must_not": [ # not
    ]
  }
}
```

must:必须满足的条件---and

should: 可以满足也可以不满足的条件--or

must_not: 不需要满足的条件--not

```json
GET /lib4/items/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "price": 25
          }
        },
        {
          "term": {
            "itemID": "id100123"
          }
        }
      ],
      "must_not": {
        "term": {
          "price": 30
        }
      }
    }
  }
}
```

嵌套使用bool：

```json
GET /lib4/items/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "itemID": "id100123"
          }
        },
        {
          "bool": {
            "must": [
              {
                "term": {
                  "itemID": "id100124"
                }
              },
              {
                "term": {
                  "price": 40
                }
              }
            ]
          }
        }
      ]
    }
  }
}
```

#### 范围过滤

```json
gt: >

lt: <

gte: >=

lte: <=

GET /lib4/items/_search
{
  "post_filter": {
    "range": {
      "price": {
        "gt": 25,
        "lt": 50
      }
    }
  }
}
```

#### 过滤非空

```json
GET /lib4/items/_search
{
  "query": {
    "bool": {
      "filter": {
        "exists": {
          "field": "price"
        }
      }
    }
  }
}

GET /lib4/items/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "exists": {
          "field": "price"
        }
      }
    }
  }
}
```

#### 过滤器缓存

ElasticSearch提供了一种特殊的缓存，即过滤器缓存（filter cache），用来存储过滤器的结果，被缓存的过滤器并不需要消耗过多的内存（因为它们只存储了哪些文档能与过滤器相匹配的相关信息），而且可供后续所有与之相关的查询重复使用，从而极大地提高了查询性能。

注意：ElasticSearch并不是默认缓存所有过滤器，
以下过滤器默认不缓存：

```
numeric_range
script
geo_bbox
geo_distance
geo_distance_range
geo_polygon
geo_shape
and
or
not
```

exists,missing,range,term,terms默认是开启缓存的

开启方式：在filter查询语句后边加上
"_catch":true

### 聚合查询

#### sum

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_of_sum": {
      "sum": {
        "field": "price"
      }
    }
  }
}
```

#### min

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_of_min": {
      "min": {
        "field": "price"
      }
    }
  }
}
```

#### max

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_of_max": {
      "max": {
        "field": "price"
      }
    }
  }
}
```

#### avg

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_of_avg": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

#### cardinality:求基数(互不相同的值的个数, 比如"男","女"就是 2)

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_of_cardi": {
      "cardinality": {
        "field": "price"
      }
    }
  }
}
```

#### terms: 分组 

```json
GET /lib4/items/_search
{
  "size": 0,
  "aggs": {
    "price_group_by": {
      "terms": {
        "field": "price"
      }
    }
  }
}
```

**对那些有唱歌兴趣的用户按年龄分组**

```json
GET /lib3/user/_search
{
  "query": {
    "match": {
      "interests": "changge"
    }
  },
  "size": 0,
  "aggs": {
    "age_group_by": {
      "terms": {
        "field": "age",
        "order": {
          "avg_of_age": "desc"
        }
      },
      "aggs": {
        "avg_of_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
```



### 复合查询

将多个基本查询组合成单一查询的查询

#### 使用bool查询

接收以下参数：

must：
    文档 必须匹配这些条件才能被包含进来。 

must_not：
    文档 必须不匹配这些条件才能被包含进来。 

should：
    如果满足这些语句中的任意语句，将增加 _score，否则，无任何影响。它们主要用于修正每个文档的相关性得分。 

filter：
    必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， bool 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

下面的查询用于查找 title 字段匹配 how to make millions 并且不被标识为 spam 的文档。那些被标识为 starred 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 _两者_ 都满足，那么它排名将更高：

```json
GET /lib3/user/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "how to make millions"
        }
      },
      "must_not": {
        "match": {
          "tag": "spam"
        }
      },
      "should": [
        {
          "match": {
            "tag": "starred"
          }
        },
        {
          "range": {
            "date": {
              "gte": "2014-01-01"
            }
          }
        }
      ]
    }
  }
}
```

如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。但，如果存在至少一条 must 语句，则对 should 语句的匹配没有要求。 
如果我们不想因为文档的时间而影响得分，可以用 filter 语句来重写前面的例子：

```json
{
  "bool": {
    "must": {
      "match": {
        "title": "how to make millions"
      }
    },
    "must_not": {
      "match": {
        "tag": "spam"
      }
    },
    "should": [
      {
        "match": {
          "tag": "starred"
        }
      }
    ],
    "filter": {
      "range": {
        "date": {
          "gte": "2014-01-01"
        }
      }
    }
  }
}
```

通过将 range 查询移到 filter 语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对 filter 查询有效的优化手段来提升性能。

bool 查询本身也可以被用做不评分的查询。简单地将它放置到 filter 语句中并在内部构建布尔逻辑：

```json
{
  "bool": {
    "must": {
      "match": {
        "title": "how to make millions"
      }
    },
    "must_not": {
      "match": {
        "tag": "spam"
      }
    },
    "should": [
      {
        "match": {
          "tag": "starred"
        }
      }
    ],
    "filter": {
      "bool": {
        "must": [
          {
            "range": {
              "date": {
                "gte": "2014-01-01"
              }
            }
          },
          {
            "range": {
              "price": {
                "lte": 29.99
              }
            }
          }
        ],
        "must_not": [
          {
            "term": {
              "category": "ebooks"
            }
          }
        ]
      }
    }
  }
}
```



#### constant_score查询

它将一个不变的常量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。

```json
{
  "constant_score": {
    "filter": {
      "term": {
        "category": "ebooks"
      }
    }
  }
}
```

term 查询被放置在 constant_score 中，转成不评分的filter。这种方式可以用来取代只有 filter 语句的 bool 查询。 



## 分布式特性

es 支持集群模型, 是一个分布式系统, 其好处主要有两个

* 增大系统容量, 如内存, 磁盘, 使得 es 集群可以支持 PB 级的数据
* 提高系统可用性, 即时部分节点停止服务, 整个集群依然可以正常服务

es 集群由多个 es 实例组成

* 不同集群**通过集群名称来区分**, 可以通过 cluster.name 进行修改, 默认为 elasticsearch
* 每个 es 实例本质上是一个 JVM 进程, 且有自己的名字, 通过 node.name 进行修改

```
nohup ./elasticsearch &
```

### Cluster State

es 集群相关的数据称为 cluster state, 主要记录如下信息

* 节点信息, 比如节点名称, 连接地址等
* 索引信息, 比如索引名称, 配置等

### Master Node

**可以修改 cluster state 的节点称为 master 节点**, **一个集群只能有一个**

cluster state 可以存在每个节点上, master 维护最新版本并同步给其他节点

mater 节点是通过集群中所有节点选举产生的, 可以被选举的节点称为 master-eligible 节点

设置 **node.master: true** 将节点设置为可选举节点, 默认为 true

### Coordinating Node(协调节点)

**处理请求的节点即为 corrdinating node**, 该节点为所有节点的默认角色, 不能取消, 主要的作用是路由请求到正确的节点处理, 比如创建索引的请求到 master 节点, 因为只有 master 节点才能修改 cluster state, 创建文档的请求会路由到相应的 primary shard 节点上

### Data Node

**存储数据的节点即为 data 节点**, 默认节点都是 data 类型, 相关配置如下

**node.date: true**

### 提高系统可用性

服务可用性

* 两个节点的情况下, 允许其中 1 个节点停止服务

数据可用性

* 引入副本(replication)解决
* 每个节点上都有完备的数据

### 增大系统容量

如何将数据分布于所有节点上

* 引入分片(shard)解决问题

分片是 es 支持 PB 级存储的基石

* 分片存储了部分数据, 可以分布于任意节点上
* 主分片数在索引**创建时指定且后续不允许修改**, 默认为 5 个
* 分片有主分片和副本分片之分, 以实现数据的高可用
* 副本分片的数据由主分片同步, 可以有多个, 从而提高读取的吞吐量



### Cluster Health

**ES 集群的三种状态:**

1. **Green:** 所有主分片和备份分片都准备就绪(分配成功), 即时有一台机器挂了(假设一台机器一个实例), 数据都不会丢失, 但会变成 Yellow 状态
2. **Yellow:** 所有主分片和备份分片都准备就绪, 但至少存在一个主分片(假设是 A)对应的备份分片没有就绪, 此时集群属于警告状态, 意味着集群高可用和容灾能力下降, 如果刚好 A 所在的机器挂了, 并且你只设置了一个备份(已处于未就绪状态), 那么 A 的数据机会丢失(查询结果不完整), 此时集群进入 Red 状态
3. **Red:** 至少有一个主分片没有就绪(直接原因是找不到对应的备份分片成为新的主分片), 此时查询的结果会出新数据丢失(不完整)

```
GET _cluster/health
```



### 故障转移(容错机制)

假设集群由 3 个节点组成, node1 所在的机器突然宕机, 此时 node2 和 node3 发现 node1 无法响应超过一段时间后会发起 master 选举, 比如选择 node2 为 master 节点, 此时由于主分片 P0 下线, 集群状态变为 Red

以9个shard，3个节点为例：

![http://www.miaomiaoqi.cn/images/elastic/search/es_5.png](http://www.miaomiaoqi.cn/images/elastic/search/es_5.png)

如果 master node 宕机，此时不是所有的 primary shard 都是 Active status，所以此时的集群状态是 red。

1. 容错处理的第一步:是选举一台服务器作为 master
2. 容错处理的第二步:新选举出的 master 会把挂掉的 primary shard 的某个 replica shard 提升为 primary shard,此时集群的状态为 yellow，因为少了一个 replica shard，并不是所有的 replica shard 都是 active status

3. 容错处理的第三步：新的 master 为 primary shard 重新分配 replica shard, 集群重新变为 green 状态



### 文档分布式存储

Document1 是如何存储到分片 P1 的? 选择 P1 的依据是什么?

* 需要文档到分片的映射算法, 使得文档均匀分布在所有分片上, 以充分利用资源

算法

* 根据文档值实时计算对应的分片

* es 通过如下公式计算文档对应的分片

  shard = hash(**routing**) % **number_of_primary_shards**

  hash 算法可以保证将数据均匀的分散在分片中

  routing 是一个关键参数, 默认是文档 id, 也可以自行指定

  number_of_primary_shards 是主分片数

  该算法与主分片数相关, 这也是分片数一旦确定后便不能修改的原因

**分档创建流程**

1. Client 向 node3 发起创建文档请求, 此时 node3 就是协调节点(Coordinating Node)
2. node3 通过 routing 计算该文档应该存储在 shard1 上, 查询 cluster state 后确认主分片 P1 在 node2 上, 然后转发创建文档的请求到 node2
3. P1 接收并执行创建文档请求后, 将同样的请求发送到副本分片 R1
4. R1 接收并执行创建文档请求后, 通知 P1 成功的结果
5. P1 接收副本分片结果后, 通知 node3 创建成功
6. node3 返回结果到 Client

![http://www.miaomiaoqi.cn/images/elastic/search/es_15.png](http://www.miaomiaoqi.cn/images/elastic/search/es_15.png)

**文档读取流程**

1. Client 向 node3 发起获取文档 1 的请求
2. node3 通过 routing 计算该文档在 Shard1 上, 查询 cluster state 后获取 Shard1 的主副分片列表, 然后以轮询的机制获取一个 Shard, 比如这次获取到的是 R1, 然后转发读取文档的请求到 R1 所在的节点 node1 上(下次轮询就有可能是 P1)
3. R1 接收并执行读取文档请求后, 将结果返回给 node3
4. node3 返回结果给 Client

**文档批量创建流程**

1. Client 向 node3 发起批量创建文档的请求 (bulk)
2. node3 通过 routing 计算所有文档对应的 shard, 然后按照主 shard 分组, 将对应的文档一次性发送到涉及的主 shard, 比如这里 3 个主 shard 都需要参与
3. 主 shard 接收并执行请求后, 将同样的请求同步到对应的副本 shard
4. 副本 shard 执行结果后返回结果到主 shard, 主 shard 再返回到 nodes
5. node3 整合结果返回给 Client

**文档批量读取流程**

1. Client 向 node3 发起批量获取文档的请求(mget)
2. node3 通过 routing 计算所有文档对应的 shard, 然后以轮询的机制获取要参与的 shard, 按照 shard 构建 mget 请求, 同时发送请求到涉及的 shard, 比如这里有 2 个 shard 需要参与
3. R1, R2 返回文档结果
4. node3 返回结果给 Client

### 脑裂问题(split-brain)

脑裂问题是分布式系统中经典的网络问题

3 个节点组成的集群, 突然 node1 的网络和其他两个节点中断, node2 和 node3 会重新选举 master, 比如 node2 成为了新的 master, 此时会更新 cluster state, node1 自己组成集群后, 也会更新 cluster state, 同一个集群由两个master, 而且维护不同的cluster state, 网络恢复后无法选择正确的 master

![http://www.miaomiaoqi.cn/images/elastic/search/es_16.png](http://www.miaomiaoqi.cn//elastic/search/es_16.png)

解决方案为仅在可选举的 master-eligible 节点数大于等于 quorum 时才可以进行 master 选举

* quorum = master-eligible节点数 / 2 + 1, 例如 3 个 master-eligible 节点时, quorum 为 2,
* 设定 discovery.zen.minimum_master_nodes 为 quorum 即可避免脑裂

### 倒排索引的不可变更

倒排索引一旦生成, 不能更改

其好处如下:

* 不用考虑并发写文件的问题, 杜绝了锁机制带来的性能问题
* 由于文件不再更改, 可以充分利用文件系统缓存, 只需载入一次, 只要内存足够, 对该文件的读取都会从内存中读取, 性能高
* 利于生成缓存数据, 因为数据不变
* 利于对文件进行压缩存储, 节省磁盘和内存存储空间

坏处为需要写入新文档时, 必须重新构建倒排索引, 然后替换老文件后, 新文档才能被检索, 导致文档实时性差

![http://www.miaomiaoqi.cn/images/elastic/search/es_17.png](http://www.miaomiaoqi.cn/images/elastic/search/es_17.png)

解决方案是新文档直接生成新的倒排索引文件, 查询的时候查询所有的倒排索引文件, 然后做结果的汇总即可

![http://www.miaomiaoqi.cn/images/elastic/search/es_18.png](http://www.miaomiaoqi.cn/images/elastic/search/es_18.png)

Lucene 便是采用了这种方案, 它构建的单个倒排索引称为 segment, 合在一起称为 Index, 与 ES 中的 Index 概念不同, ES 中的一个 Shard 对应一个 Lucene Index

Lucene 会有一个专门的文件来记录所有的 segment 信息, 称为 commit point

![http://www.miaomiaoqi.cn/images/elastic/search/es_19.png](http://www.miaomiaoqi.cn/images/elastic/search/es_19.png)

### 文档搜索实时性

**refresh**

segment 写入磁盘的过程依然很耗时, 可以借助文件系统缓存的特性, 先将 segment 在缓存中创建并开放查询来进一步提升实时性, 该过程在 es 中称为 refresh(**将文档构建的倒排索引在缓存中创建一份**)

在 refresh 之前文档会先存储在一个 buffer 中, refresh 时将 buffer 中的所有文档清空并生成 segment

es 默认每 1 秒执行一次 refresh, 因此文档的实时性被提高到 1 秒, 这也是 es 被称为近实时(Near Real Time)的原因

![http://www.miaomiaoqi.cn/images/elastic/search/es_20.png](http://www.miaomiaoqi.cn/images/elastic/search/es_20.png)

refresh 发生的时机主要有如下几种情况

* 间隔时间达到, 通过 `index.settings.refresh_interval` 来设定, 默认是 1s
* index.buffer 占满时, 其大小通过 `indices.memory.index_buffer_size` 设置, 默认为 jvm heap 的 10%, **所有的 shard 共享**
* flush 发生时也会 refresh

**translog**

如果在内存中的 segment 还没有写入磁盘前发生了宕机, 那么其中的文档就无法恢复了, 如何解决这个问题?

es 引入了 translog 机制, 写入文档到 buffer 时, 同时将该操作写入 translog, **translog 会即时写入磁盘(fsync) 生成 translog file**, 6.x 默认每个请求都会落盘, 可以修改为每 5 秒写一次, 这样的风险便是丢失 5 秒内的数据, 相关配置为 `index.translog.*`, es 启动时会检查 translog 文件, 并从中恢复数据

```
"index.translog.durability": "request" # 每个请求都落盘

PUT /my_index/_settings
{
    "index.translog.durability": "async",
    "index.translog.sync_interval": "5s"	# 每 5 秒落盘一次
}
```

![http://www.miaomiaoqi.cn/images/elastic/search/es_21.png](http://www.miaomiaoqi.cn/images/elastic/search/es_21.png)

**flush**

将 translog 写入磁盘

将 index buffer 清空, 其中的文档生成一个新的 segment, 相当于一个 refresh 操作

执行 fsync 操作, 将内存中的 segment 写入磁盘

更新 commit point 写入磁盘

删除旧的 translog 文件

![http://www.miaomiaoqi.cn/images/elastic/search/es_22.png](http://www.miaomiaoqi.cn/images/elastic/search/es_22.png)

flush 发生时机

* 间隔时间达到时, 默认是 30 分钟, 5.x 之前可以通过 `index.translog.flush_threshold_period` 修改, 之后无法修改
* translog 占满时, 其大小可以通过 `index.translog.flush_threshold_size` 控制, 默认是 512m, 每个 index 有自己的 translog

![http://www.miaomiaoqi.cn/images/elastic/search/es_23.png](http://www.miaomiaoqi.cn/images/elastic/search/es_23.png)

**图中的每个 shard 对应一个 lucene index**

**删除与更新文档**

segment 一旦生成就不能更改, 那么如果你要删除文档该如何操作?

* lucene 专门维护了一个.del 的文件, 记录所有已经删除的文档, 注意 .del 上记录的是文档在 Lucene 内部的 id
* 在查询结果返回前会过滤掉 .del 中的所有文档

更新文档如何进行呢?

* 首先删除文档, 然后再创建新文档

**segment merging**

随着 segment 的增多, 由于一次查询的 segment 数增多, 查询速度会变慢

es 会定时在后台进行 segment merge 操作, 减少 segment 的数量

通过 force_merge api 可以手动强制做 segment merge 操作



## 深入 Search 运行机制

Search 执行的时候实际分两个步骤运作的

* Query 阶段
* Fetch 阶段

Query-Then-Fetch

### Query 阶段

1. node3 接收到用户的 search 请求**(这里的请求不是根据 id 的请求, 无法准确路由到某个 shard)**, 会先进行 Query 阶段(此时是 Coordinating Node 角色)
2. node3 在 6 个主副分片中随机选择 3 个分片, 发送 search request
3. 被选中的 3 个分片会分别执行查询并排序, 返回 from + size 个文档 id 和排序值(这里只返回 id, 不返回文档本身内容)
4. node3 整合 3 个分片返回的 from + size 个文档 id, **根据排序值排序后选取 from 到 from + size 的文档 id**

![http://www.miaomiaoqi.cn/images/elastic/search/es_24.png](http://www.miaomiaoqi.cn/images/elastic/search/es_24.png)

### Fetch 阶段

1. node3 根据 Query 阶段获取的文档 id 列表去对应的 shard 上获取文档详情数据
2. node3 向相关 shard 发送 multi_get 请求
3. 3 个分片返回文档的详细数据
4. node3 拼接返回的结果并返回给客户

![http://www.miaomiaoqi.cn/images/elastic/search/es_25.png](http://www.miaomiaoqi.cn/images/elastic/search/es_25.png)

### 相关性算分问题

**相关性算分在 shard 和 shard 之间是相互独立的**, 也就意味着同一个 Term 的 IDF 在等值在不同 shard 上是不同的. 文档的相关性算分和它所处的 shard 有关

**在文档数量不多时, 会导致相关性算分严重不准的情况发生**, 比如 shard0 有 10 个文档, shard1 只有 1 个文档, 有可能那 1 个文档的相关性算分远远高于其他 10 个文档

```json
DELETE test_search_relevance
PUT test_search_relevance
{
  "settings": {
    "index":{
      # "number_of_shards": 1
      "number_of_shards":5 # 通过改变分片数, 使文档集中到一个 shrad 上
    }
  }
}

POST test_search_relevance/doc
{
  "name":"hello"
}

POST test_search_relevance/doc
{
  "name":"hello,world"
}

POST test_search_relevance/doc
{
  "name":"hello,world!a beautiful world"
}

# 理论上 hello 的评分应该是最高的
GET test_search_relevance/_search

GET test_search_relevance

# 查看相关性算分
GET test_search_relevance/_search
{
  "explain": true, 
  "query": {
    "match":{
      "name":"hello"
    }
  }
}
```

解决该问题的思路有两个

* 一是设置分片数为 1 个, 从根本上排除问题, 在文档数量不多的时候可以考虑该方案, 比如百万到千万级别的文档数量

* 二是使用 DFS Query-then-Fetch 查询方式

  DFS Query-then-Fetch 是在拿到所有文档后再重新完整的计算一次相关性算分, 耗费更多的 cpu 和内存, 执行性能也比较低下, 一般不建议使用, 指定 search_type=dfs_query_then_fetch

  ```json
  GET test_search_relevance/_search?search_type=dfs_query_then_fetch
  {
    "query": {
      "match":{
        "name":"hello"
      }
    }
  }
  ```



### 排序

es 默认会采用相关性算分排序, 用户可以通过设定 sorting 参数来自行设定排序规则

```json
GET test_search_index/_search
{
	"sort": { # 关键词
		"birth": "desc"
	}
}

GET test_search_index/_search
{
  "sort": [
    {
      "birth": "desc" # 关键词排序
    },
    {
      "_score": "desc" # 相关性得分排序
    },
    {
      "_doc": "desc" # 按照文档内部 id, 和索引的顺序相关(分片内唯一)
    }
  ]
}
```

```json
DELETE test_search_index

PUT test_search_index
{
  "settings": {
    "index":{
        "number_of_shards": "1"
    }
  }
}

POST test_search_index/doc/_bulk
{"index":{"_id":"1"}}
{"username":"alfred way","job":"java engineer","age":18,"birth":"1990-01-02","isMarried":false}
{"index":{"_id":"2"}}
{"username":"alfred","job":"java senior engineer and java specialist","age":28,"birth":"1980-05-07","isMarried":true}
{"index":{"_id":"3"}}
{"username":"lee","job":"java and ruby engineer","age":22,"birth":"1985-08-07","isMarried":false}
{"index":{"_id":"4"}}
{"username":"alfred junior way","job":"ruby engineer","age":23,"birth":"1989-08-07","isMarried":false}



# score is null here
GET test_search_index/_search
{
  "query":{
    "match": {
      "username": "alfred"
    }
  },
  "sort":{
    "birth":"desc"
  }
}

GET test_search_index/_search
{
  "query":{
    "match": {
      "username": "alfred"
    }
  },
  "sort": [
    {
      "birth": "desc"
    },
    {
      "_score": "desc"
    },
    {
      "_doc": "desc"
    }
  ]
}

PUT test_search_index/doc/0
{
  "username":"aaa"
}

DELETE test_search_index/doc/2

GET test_search_index

GET test_search_index/_search
{
  "sort": [
    {
      "username": "asc"
    }
  ]
}

GET test_search_index/_search
{
  "sort": [
    {
      "username": "desc"
    },
    {
      "_id": "desc"
    }
  ]
}

DELETE test_search_index/doc/5

PUT test_search_index/doc/5
{
  "username":"alfred junior zoo"
}

DELETE test_search_index/doc/5

GET test_search_index/_search
{
  "sort":{
    "username.keyword":"desc"
  }
}
```

**es 按照字符串排序比较特殊, 因为 es 有 text 和 keyword 两种类型, 针对 text 类型排序会报错误, 针对 keyword 类型是可以排序的, 因为默认开启了 docvalues**

```json
GET test_search_index/_search
{
  "sort": {
    "username": "desc"
  }
}

{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [username] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "test_search_index",
        "node": "0yILyYBVQ1Cqw-7-3tLHkg",
        "reason": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [username] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
        }
      }
    ],
```

排序的过程实质是对原始内容排序的过程, 这个过程中**倒排索引无法发挥作用**, 需要用到正排索引, 也就是通过文档 id 和字段可以快速得到字段原始内容

es 对此提供了两种实现方式

* fielddata 默认禁用, 可以通过如下 API 开启, **此时字符串是按照分词后的 term 排序的**, 往往结果很难符合预期, 一般是在对分词做聚合分析的时候开启, **fielddata 只针对 text 类型生效**

  ```json
  PUT test_search_index/_mapping/doc
  {
  	"properties": {
  		"username": {
  			"type": "text", # 类型不可以修改
  			"fielddata": true # fielddate 可以随时开启和关闭
  		}
  	}
  }
  
  GET test_search_index/_search
  {
    "sort": {
      "username": "desc"
    }
  }
  ```

  

* doc values 默认启用, 除了 text 类型, 可以在创建索引时就选择关闭, **如果后边要开启 doc values, 需要做 reindex 操作**

  ```json
  DELETE test_doc_values
  
  PUT test_doc_values
  
  GET test_doc_values
  
  PUT test_doc_values/_mapping/doc
  {
    "properties": {
      "username": {
        "type": "keyword",
        "doc_values": false # 关闭 doc_values, 就不能排序了, 后续也不能修改该字段
      },
      "hobby": {
        "type": "keyword"
      }
    }
  }
  
  PUT test_doc_values/doc/1
  {
    "username":"alfred",
    "hobby":"basketball"
  }
  
  GET test_doc_values/_search
  {
    "sort":"username"
  }
  
  GET test_doc_values/_search
  {
    "sort":"hobby"
  }
  ```

  ```json
  # can be used to get original field value for not stored field
  PUT test_search_index/_mapping/doc
  {
    "properties": {
      "username":{
        "type":"text",
        "fielddata": false
      }
    }
  }
  
  GET test_search_index/_search
  {
    "docvalue_fields": [ # 通过 docvalue_fields 可以查看具体的排序内容
      "username",
      "username.keyword",
      "age"
    ]
  }
  
  PUT test_search_index/_mapping/doc
  {
    "properties": {
      "username":{
        "type":"text",
        "fielddata": true
      }
    }
  }
  ```

  

|   对比   |                      FieldData                       |               DocValues                |
| :------: | :--------------------------------------------------: | :------------------------------------: |
| 创建时机 |                      搜索时创建                      |   索引时创建, 与倒排索引创建时机一致   |
| 创建位置 |                       JVM Heap                       |                  磁盘                  |
|   优点   |                不会占用额外的磁盘资源                |      不会占用 Heap 内存, 减少 GC       |
|   缺点   | 文档过多时, 即时创建会花过多时间, 占用过多 Heap 内存 | 减慢索引的创建速度, 占用额外的磁盘资源 |



### 分页与遍历

es 提供了 3 种方式来解决分页与遍历的问题

* from/size
* scroll
* search_after

#### From/Size

最常用的分页方案

**from:** 指明开始位置, 从 0 开始

**size:** 指明获取总数

```json
GET test_search_index/_search
{
	"from": 1,
	"size": 2
}
```

```json
# pagination
GET test_search_index/_search
{
  "from":0,
  "size":2
}

# total_page=(total+page_size-1)/page_size
GET test_search_index/_search
{
  "from":9998, # 因为分片的原因, 每个 shard 都要获取 9998 个文档再汇总排序, 浪费性能
  "size":2
}
```

**深度分页(deep paging)是一个经典的问题**: 查询的很深，比如一个索引有三个 primary shard，分别存储了 6000 条数据，我们要得到第 100 页的数据(每页 10条)，类似这种情况就叫deep paging

如何得到第100页的10条数据？

在每个 shard 中搜索990到999这10条数据，然后用这30条数据排序，排序之后取10条数据就是要搜索的数据，这种做法是错的，因为3个 shard 中的数据的_score分数不一样，可能这某一个 shard 中第一条数据的 _score 分数比另一个 shard 中第1000条都要高，所以在每个shard中搜索990到999这10条数据然后排序的做法是不正确的。

正确的做法是每个shard把0到999条数据全部搜索出来（按排序顺序），然后全部返回给coordinate node，由coordinate node按_score分数排序后，取出第100页的10条数据，然后返回给客户端。


deep paging性能问题

1. 耗费网络带宽，因为搜索过深的话，各 shard 要把数据传送给 coordinate node，这个过程是有大量数据传递的，消耗网络，

2. 消耗内存，各 shard 要把数据传送给 coordinate node，这个传递回来的数据，是被 coordinate node 保存在内存中的，这样会大量消耗内存。

3. 消耗 cpu, coordinate node 要把传回来的数据进行排序，这个排序过程很消耗 cpu

**鉴于 deep paging 的性能问题，所以应尽量减少使用分页查询, es 通过 `index.max_result_window` 限定最多搜索 10000 条数据**



#### Scroll

遍历文档集的 api, **以快照的方式来避免深度分页的问题**

* 不能用来做实时搜索, 因为数据不是实时的
* 尽量不要使用复杂的 sort 条件, 使用 _doc 最高效
* 使用稍显复杂

如果一次性要查出来比如10万条数据，那么性能会很差，此时一般会采取用 scroll 滚动查询，一批一批的查，直到所有数据都查询完为止。

1. scoll 搜索会在第一次搜索的时候，保存一个当时的视图快照, 并返回一个快照的 id，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的
2. 采用基于 \_doc(不使用_ score)进行排序的方式，性能较高
3. 每次发送 scroll 请求，我们还需要指定一个scoll参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了
4. 当 hits 为 [] 的时候, 就代表最后一条了

```json
# scroll
GET test_search_index/_search?scroll=5m
{
  "size":1
}


GET _search/scroll
GET test_search_index/_search

# new doc can not be searched
PUT test_search_index/doc/10
{
  "username":"doc10"
}


DELETE test_search_index/doc/10

POST _search/scroll
{
  "scroll" : "5m", 
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABbTEWMHlJTHlZQlZRMUNxdy03LTN0TEhrZw=="
}

# 删除指定 id 的快照
DELETE /_search/scroll
{
  "scroll_id": [
    "DXFSFASM...",
    "MGGPASmoasM..."
  ]
}

# 删除内存中的所有快照
DELETE _search/scroll/_all
```



#### Search After

避免深度分页的性能问题, 提供实时的下一页文档获取功能

* 缺点是不能使用 from 参数, 即不能指定页数
* 只能下一页, 不能上一页
* 使用简单

```json
# search_after
GET test_search_index/_search
{
  "size":1,
  "sort":{ # 要保障 sort 值唯一
    "age":"desc",
    "_id":"desc"
  }
}


GET test_search_index/_search
{
  "size":1,
  "search_after":[23,"4"], # 作为下一次查询的参数
  "sort":{
    "age":"desc",
    "_id":"desc"
  }
}
```

通过唯一排序值定位将每次要处理的文档数都控制在 size 内



|     类型     |                    场景                    |
| :----------: | :----------------------------------------: |
|  From/Size   | 需要实时获取顶部的部分文档, 且需要自由翻页 |
|    Scroll    |     需要全部文档, 如导出所有数据的功能     |
| Search_After |        需要全部文档, 不需要自由分页        |



## 聚合分析(Aggregation)

搜索引擎用来回答如下问题:

* 请告诉我地址为上海的所有订单
* 请告诉我最近 1 天内创建但没有付款的所有订单

聚合分析可以回答如下问题

* 请告诉我最近 1 周每天的订单成交量是多少
* 请告诉我最近 1 个月每天的平均订单金额是多少
* 亲告诉我最近半年卖的最火的前 5 个商品是哪些

聚合分析英文为 Aggregation, **是 es 除搜索功能外提供的针对 es 数据做统计分析的功能**, 功能丰富, 提供 Bucket, Metric, Pipeline 等多种分析方式, 可以满足大部分的分析需求

实时性高, 所有的计算结果都是即时返回, 而 hadoop 等大数据系统一般都是 T+1 级别的

聚合分析作为 search 的一部分, api 如下所示

```json
GET test_search_index/_search
{
    "size": 0,
    "aggs": { # 关键词与 query 同级
        "<aggregation_name>": { # 自定义聚合名称
            "<aggregation_type>": {
                "<aggregation_body>"
            }
        [,"aggs": {[<sub_aggregation>]+}]? # 子查询
        }
        [,"<aggregation_name_2>": {...}]* # 可以包含多个聚合分析
    }
}
```

请告诉我公司目前在职人员工作岗位的分布情况?

```json
GET test_search_index/_search
{
    "size": 0,
    "aggs": {
        "pepole_per_job": {
            "terms": {
                "field": "job.keyword"
            }
        }
    }
}
```

为了便于理解, es 将聚合分析主要分为如下 4 类

* Metric, 指标分析类型, 如计算最大值, 最小值, 平均值等
* Bucket, 分桶类型, 类似 SQL 中的 GROUP BY 语法
* Pipeline, 管道分析类型, 基于上一级的聚合分析结果进行再分析
* Matrix, 矩阵分析类型

### 数据准备

```json
DELETE test_search_index
POST test_search_index/doc/_bulk
{"index":{"_id":"1"}}
{"username":"alfred way","job":"java engineer","age":18,"birth":"1990-01-02","isMarried":false,"salary":10000}
{"index":{"_id":"2"}}
{"username":"tom","job":"java senior engineer","age":28,"birth":"1980-05-07","isMarried":true,"salary":30000}
{"index":{"_id":"3"}}
{"username":"lee","job":"ruby engineer","age":22,"birth":"1985-08-07","isMarried":false,"salary":15000}
{"index":{"_id":"4"}}
{"username":"Nick","job":"web engineer","age":23,"birth":"1989-08-07","isMarried":false,"salary":8000}
{"index":{"_id":"5"}}
{"username":"Niko","job":"web engineer","age":18,"birth":"1994-08-07","isMarried":false,"salary":5000}
{"index":{"_id":"6"}}
{"username":"Michell","job":"ruby engineer","age":26,"birth":"1987-08-07","isMarried":false,"salary":12000}
```

### Metric 聚合分析

#### **min**

返回数值类字段的最小值

```json
GET /test_search_index/_search
{
  "size": 0, # 不需要返回文档就写 0
  "aggs": {
    "min_age": { # 聚合分析的名称, 可任意指定
      "min": { # 关键词
        "field": "age" # 对哪个字段进行聚合分析
      }
    },
        "max_age": { # 一条语句可以写多个聚合分析
      "max": {
        "field": "age"
      }
    }
  }
}
```

#### max

```json
GET /test_search_index/_search
{
  "size": 0, # 不需要返回文档就写 0
  "aggs": {
    "max_age": { # 聚合分析的名称, 可任意指定
      "max": { # 关键词
        "field": "age" # 对哪个字段进行聚合分析
      }
    }
  }
}
```



#### avg

```json
GET /test_search_index/_search
{
  "size": 0, # 不需要返回文档就写 0
  "aggs": {
    "avg_age": { # 聚合分析的名称, 可任意指定
      "avg": { # 关键词
        "field": "age" # 对哪个字段进行聚合分析
      }
    }
  }
}
```



#### sum

```json
GET /test_search_index/_search
{
  "size": 0, # 不需要返回文档就写 0
  "aggs": {
    "sum_age": { # 聚合分析的名称, 可任意指定
      "sum": { # 关键词
        "field": "age" # 对哪个字段进行聚合分析
      }
    }
  }
}
```



#### cardinality

意为集合的势, 或者基数, 是指不同数值的个数, 类似 SQL 中的 distinct count

```json
GET /test_search_index/_search
{
  "size": 0, # 不需要返回文档就写 0
  "aggs": {
    "count_of_job": { # 聚合分析的名称, 可任意指定
      "cardinality": { # 关键词
        "field": "job.keyword" # 对哪个字段进行聚合分析
      }
    }
  }
}
```



#### stats

返回一系列数值类型的统计值, 包含 min, max, avg, sum 和 count

```json
GET /test_search_index/_search
{
    "size": 0,
    "aggs": {
        "stats_age": {
            "stats": {
                "field": "age"
            }
        }
    }
}
```

#### extended_stats

对 status 的扩展, 包含了更多的统计数据, 如方差, 标准差

```json
GET /test_search_index/_search
{
    "size": 0,
    "aggs": {
        "stats_age": {
            "extended_stats": {
                "field": "age"
            }
        }
    }
}
```

#### percentiles

百分位数统计

```json
GET /test_search_index/_search
{
    "size": 0,
    "aggs": {
        "per_salary": {
            "percentiles": {
                "field": "salary"
            }
        }
    }
}
```

#### percentile_ranks

百分位数统计排名

```json
# 30 和 50 在当前年龄里边处于一个怎样的百分位
GET /test_search_index/_search
{
    "size": 0,
    "aggs": {
        "per_age": {
            "percentile_ranks": {
                "field": "age",
                "values": [
                  30,
                  50
                ]
            }
        }
    }
}
```

#### top_hits

一般用于分桶后获取该桶内最匹配的顶部文档列表, 即详情数据

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "top_employee": {
          "top_hits": {
            "size": 10,
            "sort": [
              {
                "age": {
                  "order": "desc"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

### Bucket 聚合分析

Bucket 意为桶, 即按照一定规则将文档分配到不同的桶中, 达到分类分析的目的

按照 Bucket 的分桶策略, 常见的 Bucket 聚合分析如下

* Terms
* Range
* Date Range
* Histogram
* Date Histogram

#### terms

该分桶策略最简单, 直接按照 term 来分桶, 如果是 text 类型, 则按照分词后的结果分桶, 要打开 fielddata: true

```json
GET /test_search_index/_search
{
    "size": 0,
    "aggs": {
        "jobs_terms": {
            "terms": {
                "field": "job.keyword", # 指定 term 字段
                "size": 5 # 指定返回数目
            }
        }
    }
}
```

#### range

通过指定数值的范围来设定分桶规则

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "key":"<10", # key 就是个名字
            "to": 20
          },
          {
            "from": 20,
            "to": 30
          },
          {
            "key":">30",
            "from": 30
          }
        ]
      }
    }
  }
}
```

#### date_range

通过指定日期的范围来设定分桶规则

```json
{
  "size": 0,
  "aggs": {
    "date_range": {
      "range": {
        "field": "birth",
        "format":"yyyy",
        "ranges": [
          {
            "to": "now-30y/y"
          },
          {
            "from": "1990",
            "to": "2000"
          },
          {
            "from": "now-20y/y"
          }
        ]
      }
    }
  }
}
```

#### histogram

直方图, 以固定间隔的策略来分割数据

```json
GET test_search_index/_search
{
  "size":0,
  "aggs":{
    "age_hist":{
      "histogram": {
        "field": "age",
        "interval": 10, # 固定的间隔
        "extended_bounds": {
          "min": 0, # 最小范围
          "max": 100 # 最大范围
        }
      }
    }
  }
}
```

#### date_histogram

针对日期的直方图或柱状图, 是时序数据分析中常用的聚合分析类型

```json
GET test_search_index/_search
{
  "size":0,
  "aggs":{
    "by_year":{
      "date_histogram": {
        "field": "birth",
        "interval": "year",
        "format":"yyyy"
      }
    }
  }
}
```

### Bucket + Metric 聚合分析

Bucket 聚合分析允许通过添加子分析来进一步进行分析, 该子分析可以使 Bucket 也可以是 Metric, 这使得 es 的聚合分析功能异常强大

#### 分桶后在分桶

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "age_range": {
          "range": {
            "field": "age",
            "ranges": [
              {
                "to": 20
              },
              {
                "from": 20,
                "to": 30
              },
              {
                "from": 30
              }
            ]
          }
        }
      }
    }
  }
}
```

### Pipeline 聚合分析

针对聚合分析的结果再次进行聚合分析, 而且支持链式调用

Pipeline 的分析结果会输出到原结果中, 根据输出的位置不同, 分为如下两类

* Parent 结果, 内嵌到现有的聚合分析结果中
    * Derivative 导数, 看发展趋势
    * Moving Average 移动平均值
    * Cumulative Sum 值的累计加和
* Sibling 结果, 与现有聚合分析结果同级
    * Max/Min/Avg/Sum Bucket
    * Stats/Extended Stats Bucket
    * Percentiles Bucket

#### Sibling Min Bucket

找出所有 Bucket(上一步的结果) 中值最小的 Bucket 名称和值

```json
GET test_search_index/_search
{
  "size":0,
  "aggs":{
    "jobs":{
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs":{
        "avg_age":{
          "avg": {
            "field": "age"
          }
        }
      }
    },
    "min_age_by_job":{
      "min_bucket": {
        # 取相同工作下平均年龄值中最小的 bucket
        "buckets_path": "jobs>avg_age"
      }
    }
  }
}
```

#### Parent Derivative 求导

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "birth": {
      "date_histogram": {
        "field": "birth",
        "interval": "year",
        "min_doc_count": 0
      },
      "aggs": {
        "avg_age": {
          "avg": {
            "field": "age"
          }
        },
        "derivative_avg_age": {
          "derivative": {
            "buckets_path": "avg_age" # 这里要注意没有">"因为是与 avg_age 同级
          }
        }
      }
    }
  }
}
```

### 作用范围

es 聚合分析默认作用范围是 `query` 结果集, 可以通过如下方式改变其作用范围

* filter

* post_filter

* global

#### filter

为某个聚合分析设定过滤条件, 从而在不更改整体 `query` 语句的情况下修改了作用范围

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs_salary_small": {
      "filter": {
        "range": {
          "salary": {
            "to": 10000
          }
        }
      },
      "aggs": {
        "jobs": {
          "terms": {
            "field": "job.keyword"
          }
        }
      }
    },
    "jobs": {
      "terms": {
        "field": "job.keyword"
      }
    }
  }
}
```

#### post_filter

作用于**文档过滤**, 但在聚合分析后生效

```json
GET test_search_index/_search
{
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword"
      }
    }
  },
  "post_filter": {
    "match":{
      "job.keyword":"java engineer"
    }
  }
}
```

#### global

无视 `query` 过滤条件, 基于全部文档进行分析

java 工程师的平均工资与所有工程师的平均工资

```json
GET test_search_index/_search
{
  "query": {
    "match": {
      "job.keyword": "java engineer"
    }
  },
  "aggs": {
    "java_avg_salary": {
      "avg": {
        "field": "salary"
      }
    },
    "all": {
      "global": {},
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    }
  }
}
```

### 排序

#### 按照关键字

可以使用自带的关键数据进行排序

* _count 文档数
* _key 按照 key 值排序

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "order": [
          {
            "_count": "asc" # 文档数排序
          },
          {
            "_key": "desc" # 按照 key 排序
          }
        ]
      }
    }
  }
}
```

#### 按照子聚合分析数值

利用子聚合分析数值进行排序

```json
GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10,
        "order": [
          {
            "avg_salary": "desc"
          }
        ]
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    }
  }
}

GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10,
        "order": [
          {
            "stats_salary.sum": "desc"
          }
        ]
      },
      "aggs": {
        "stats_salary": {
          "stats": {
            "field": "salary"
          }
        }
      }
    }
  }
}


GET test_search_index/_search
{
  "size": 0,
  "aggs": {
    "salary_hist": {
      "histogram": {
        "field": "salary",
        "interval": 5000,
        "order": {
          "age>avg_age": "desc"
        }
      },
      "aggs": {
        "age": {
          "filter": {
            "range": {
              "age": {
                "gte": 10
              }
            }
          },
          "aggs": {
            "avg_age": {
              "avg": {
                "field": "age"
              }
            }
          }
        }
      }
    }
  }
}
```

### 原理与精准度问题

#### Min 聚合的执行流程

协调节点从各个分片上获取最小的数据, 再求出最小的数据返回给客户端

![http://www.miaomiaoqi.cn/images/elastic/search/es_26.png](http://www.miaomiaoqi.cn/images/elastic/search/es_26.png)

#### Terms 聚合的执行流程

协调节点从各个分片上获取到 Top5 的数据, 再求出真正的 top5 返回给客户端

![http://www.miaomiaoqi.cn/images/elastic/search/es_27.png](http://www.miaomiaoqi.cn/images/elastic/search/es_27.png)

**Terms 永远不准确**

![http://www.miaomiaoqi.cn/images/elastic/search/es_28.png](http://www.miaomiaoqi.cn/images/elastic/search/es_28.png)

**Terms 不准确的原因**

数据分散在多 shard 上, Coordinating Node 无法得悉数据全貌

**Terms 不准确的解决办法**

* 设置 `Shard` 数为 1, 消除数据分散的问题, 但无法承载大数据量

* 合理设置 `shard_size` 大小, 即每次从 Shard 上多获取数据, 以提升准确度但会降低性能, 默认为 (size x 1.5) + 10

* 设定 `show_term_doc_count_error` 可以查看每个 bucket 误算的最大值

    ```json
    GET test_search_index/_search
    {
      "size": 0,
      "aggs": {
        "jobs": {
          "terms": {
            "field": "job.keyword",
            "size": 2,
            "shard_size": 10,
            "show_term_doc_count_error": true
          }
        }
      }
    }
    ```

* 如何合理设置 `Shard_Size` 的大小

    terms 聚合返回结果中有如下两个统计值

    * doc_count_error_upper_bonud 被遗漏的 term 可能的最大值, 0 表明计算准确
    * sum_other_doc_count 返回结果 bucket 的 term 外其他 term 的文档总数

    ![http://www.miaomiaoqi.cn/images/elastic/search/es_29.png](http://www.miaomiaoqi.cn/images/elastic/search/es_29.png)

### 近似统计算法

在 ES 的聚合分析中, Cardinality 和 Percentile 分析使用的是近似统计算法

* 结果是近似准确的, 但不一定精准
* 可以通过参数的调整使其结果精准, 但同时也意味着更多的计算时间和更大的性能消耗

![http://www.miaomiaoqi.cn/images/elastic/search/es_30.png](http://www.miaomiaoqi.cn/images/elastic/search/es_30.png)

## 数据建模

英文为 Data Modeling, 为创建数据模型的过程

数据模型(Data Model)

* 对现实世界进行抽象描述的一种工具和方法
* 通过抽象的实体及实体之间联系的形式去描述业务规则, 从而实现对现实世界的映射

### 数据建模的过程

* 概念模型

    确定系统的核心需求和范围边界, 设计实体和实体间的关系

* 逻辑模型

    进一步梳理业务需求, 确定每个实体属性, 关系和约束等

* 物理模型

    * 结合具体的数据库产品, 在满足业务读写性能等需求的前提下确定最终的定义
    * Mysql, MongoDB, ElasticSearch 等
    * 第三范式

### ES 中的数据建模

ES 是基于 Lucene 以倒排索引为基础实现的存储体系, 不遵循关系型数据库中的范式约定

![http://www.miaomiaoqi.cn/images/elastic/search/es_31.png](http://www.miaomiaoqi.cn/images/elastic/search/es_31.png)

#### Mapping 字段相关设置

|                    属性名称                    |   可选值    |                             说明                             |
| :--------------------------------------------: | :---------: | :----------------------------------------------------------: |
|                    enabled                     | true\|false |                  仅存储, 不做搜索或聚合分析                  |
|                     index                      | true\|false |                       是否构建倒排索引                       |
| index_options  |      docs\|freqs\|positions\|offsets      |                    存储倒排索引的哪些信息                    |
|               norms                |     true\|false        | 是否存储归一化相关参数, 如果字段仅用于过滤和聚合分析, 可关闭 |
|            doc_values              |        true\|false     |            是否启用 doc_values 用于排序和聚合分析            |
|            field_data              |      false\|true       |      是否为 text 类型启用 fielddata, 实现排序和聚合分析      |
|               store                |       false\|true      |                       是否额外存储该字段值                       |
|              coerce                |    true\|false         | 是否开启自动数据类型转换功能, 比如字符串转为数字, 浮点转为整型等 |
|                  multifields                  |             |        多字段, 灵活使用多字段特性来解决多样的业务需求        |
|          dynamic           |       true\|false\|strict      |                    控制 mapping 自动更新                     |
|          date_detection            |      true\|false       |               是否自动识别日期类型, 建议 false               |



#### Mapping 字段属性的设定流程

![http://www.miaomiaoqi.cn/images/elastic/search/es_32.png](http://www.miaomiaoqi.cn/images/elastic/search/es_32.png)

##### 何种类型

* 字符串类型

    需要分词则设定为 text 类型, 否则设置 keyword 类型

* 枚举类型

    基于性能考虑将其设定为 keyword 类型, 即便该数据为整形

* 数值类型

    尽量选择贴近的类型, 比如 byte 即可表示所有数值时, 即选用 byte, 不要用 long

* 其他类型

    比如布尔类型, 日期, 地理位置数据等

##### 是否需要检索

* 完全不需要检索, 排序, 聚合分析的字段

    enabled 设置为 false

* 不需要检索的字段

    index 设置为 false

* 需要检索的字段时, 可以通过如下配置设定需要的存储粒度

    index_options 结合需要设定

    norms 不需要归一化数据时关闭即可

##### 是否需要排序和聚合分析

* 不需要排序或者聚合分析功能

    doc_values 设定为 false

    fielddata设定为 false

##### 是否需要另行存储

* 是否需要专门存储当前字段的数据

    store 设定为 true, 即可存储该字段的原始内容(与_srouce 中的数据不相关)

    一般结合 _source 的 enabled 设定为 false 时使用

### 实例

博客文章 blog_index

* 标题 title
* 发布日期 publish_date

* 作者 author

* 摘要 abstract
* 网络地址 url
* 大字段内容 content

##### blog_index 的 mapping 设置1

```json
DELETE blog_index
PUT blog_index
{
  "mappings": {
    "doc":{
      "properties": {
        "title":{
          "type": "text", # 标题需要分词检索所以用 text
          "fields": { # 还使用了 multifield, 方便完全匹配搜索
            "keyword":{
              "type":"keyword",
              "ignore_above": 100
            }
          }
        },
        "publish_date":{ # 日期类型
          "type":"date"
        },
        "author":{ # 作者不需要分词
          "type":"keyword",
          "ignore_above": 100
        },
        "abstract":{ # 摘要需要分词检索
          "type": "text"
        },
        "url":{ # 仅做展示, 不需要搜索
          "enabled":false
        },
    		"content": {
          "type": "text"
        }
      }
    }
  }
}

PUT blog_index/doc/1
{
  "title":"blog title",
  "content":"blog content"
}

GET blog_index/_search # 会在 _source 中返回 content 内容

GET blog_index/_search?_source=title # 只会返回 title 字段, 但是在底层内部交换时还是会带 content 字段, 只是返回客户端时做了简单过滤
```

##### blog_index 的 mapping 设置2, 解决大字段问题

```json
DELETE blog_index
PUT blog_index
{
  "mappings": {
    "doc": {
      "_source": {
        "enabled": false # 禁用 _source 属性
      },
      "properties": {
        "title": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 100
            }
          },
          "store": true
        },
        "publish_date": {
          "type": "date",
          "store": true
        },
        "author": {
          "type": "keyword",
          "ignore_above": 100, 
          "store": true
        },
        "abstract": {
          "type": "text",
          "store": true
        },
        "content": {
          "type": "text",
          "store": true
        },
        "url": {
          "type": "keyword",
          "doc_values":false,
          "norms":false,
          "ignore_above": 100, # 如果 term 超过 100 个字符, 只取前面 100 个字符
          "store": true
        }
      }
    }
  }
}

# 添加数据
PUT blog_index/doc/1
{
  "title":"blog title",
  "content":"blog content"
}

# 不会返回任何字段, 因为 _source 禁用了
GET blog_index/_search 

GET blog_index/_search
{	# 通过指定 stored_fields 返回数据, 因为开启了 store=true
  "stored_fields": ["title","publish_date","author","abstract","url"], 
  "query": {
    "match": {
      "content": "blog"
    }
  },
  "highlight": {
    "fields":{
      "content": {}
    }
  }
}
```

### 关联关系处理

ES 不擅长处理关系型数据库中的关联关系, 比如文章表 blog 与评论表 comment 之间通过 blog_id 关联, 在 ES 中可以通过如下两种手段变相解决

* Nested Object
* Parent/Child

评论 Comment

* 文档 id blog_id
* 评论人 username
* 评论日期 date
* 评论内容 content

![http://www.miaomiaoqi.cn/images/elastic/search/es_33.png](http://www.miaomiaoqi.cn/images/elastic/search/es_33.png)

#### Nested Object

```json
DELETE blog_index
# comments 需要保存到 blog 下
PUT blog_index/doc/2
{
  "title": "Blog Number One",
  "author": "alfred",
  "comments": [
    {
      "username": "lee",
      "date": "2017-01-02",
      "content": "awesome article!"
    },
    {
      "username": "fax",
      "date": "2017-04-02",
      "content": "thanks!"
    }
  ]
}

GET blog_index

# 下边的查询会把 fax 的评论也查出来, 我们预期的是评论人是 lee, 评论内容是 thanks
GET blog_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "comments.username": "lee"
          }
        },
        {
          "match": {
            "comments.content": "thanks"
          }
        }
      ]
    }
  }
}


# 使用 nested mapping 解决上述问题
DELETE blog_index_nested
PUT blog_index_nested
{
  "mappings": {
    "doc":{
      "properties": {
        "title":{
          "type": "text",
          "fields": {
            "keyword":{
              "type":"keyword",
              "ignore_above": 100
            }
          }
        },
        "publish_date":{
          "type":"date"
        },
        "author":{
          "type":"keyword",
          "ignore_above": 100
        },
        "abstract":{
          "type": "text"
        },
        "url":{
          "enabled":false
        },
        "comments":{
          "type":"nested", # 这里要注意, 需要关联的数据 type 设置成 nested
          "properties": {
            "username":{
              "type":"keyword",
              "ignore_above":100
            },
            "date":{
              "type":"date"
            },
            "content":{
              "type":"text"
            }
          }
        }
      }
    }
  }
}

PUT blog_index_nested/doc/2
{
  "title": "Blog Number One",
  "author": "alfred",
  "comments": [
    {
      "username": "lee",
      "date": "2017-01-02",
      "content": "awesome article!"
    },
    {
      "username": "fax",
      "date": "2017-04-02",
      "content": "thanks!"
    }
  ]
}


GET blog_index_nested/_search
{
  "query": {
    "nested": {
      "path": "comments", # 查询需要指定 path
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "comments.username": "lee"
              }
            },
            {
              "match": {
                "comments.content": "awesome"
              }
            }
          ]
        }
      }
    }
  }
}
```

#### Parent/Child

ES 还提供了类似关系型数据库中 `join` 的实现方式, 使用 `join` 数据类型实现, 需要 es6.0 以上的版本

![http://www.miaomiaoqi.cn/images/elastic/search/es_34.png](http://www.miaomiaoqi.cn/images/elastic/search/es_34.png)

![http://www.miaomiaoqi.cn/images/elastic/search/es_35.png](http://www.miaomiaoqi.cn/images/elastic/search/es_35.png)

常见 `query` 语法包括如下几种

* parent_id 返回某父文档的子文档

    ![http://www.miaomiaoqi.cn/images/elastic/search/es_36.png](http://www.miaomiaoqi.cn/images/elastic/search/es_36.png)

* has_child 返回包含某子文档的父文档

    ![http://www.miaomiaoqi.cn/images/elastic/search/es_37.png](http://www.miaomiaoqi.cn/images/elastic/search/es_37.png)

* has_parent 返回包含某父文档的子文档

```json
# 使用 parent child 方式完成关联查询
DELETE blog_index_parent_child
PUT blog_index_parent_child
{
  "mappings": {
    "doc": {
      "properties": {
        "join": {
          "type": "join",
          "relations": {
            "blog": "comment"
          }
        }
      }
    }
  }
}

GET blog_index_parent_child

GET blog_index_parent_child/_search

# 创建两个父文档
PUT blog_index_parent_child/doc/1
{
  "title":"blog",
  "join":"blog"
}

PUT blog_index_parent_child/doc/2
{
  "title":"blog2",
  "join":"blog"
}

# 创建一个子文档, 当前父子文档是在一个 index 下, 要保证 id 唯一
# routing 指定父文档的 id, 保证父子文档在一个分片上
PUT blog_index_parent_child/doc/comment-1?routing=1
{
  "comment":"comment world",
  "join":{
    "name":"comment",
    "parent":1
  }
}


PUT blog_index_parent_child/doc/comment-2?routing=2
{
  "comment":"comment hello",
  "join":{
    "name":"comment",
    "parent":2
  }
}

GET blog_index_parent_child/_search

# get all child for parent
GET blog_index_parent_child/_search
{
  "query":{
    "parent_id":{
      "type":"comment",
      "id":"2"
    }
  }
}

GET blog_index_parent_child/_search
{
  "query":{
    "match": {
      "comment": "world"
    }
  }
}


# get parent documents which has child matching following conditions
GET blog_index_parent_child/_search
{
  "query":{
    "has_child": {
      "type": "comment",
      "query": {
        "match": {
          "comment": "world"
        }
      }
    }
  }
}

GET blog_index_parent_child/_search
{
  "query":{
    "has_parent": {
      "parent_type": "blog",
      "query": {
        "match": {
          "title": "blog"
        }
      }
    }
  }
}
```

#### 两种方式对比

| 对比 |          Nested Object           |                    Parent/Child                    |
| :--: | :------------------------------: | :------------------------------------------------: |
| 优点 |  文档存储在一起, 因此读取性能高  |           父子文档可以独立更新, 互不影响           |
| 缺点 | 更新父或子文档时需要更新整个文档 | 为了维护 join 的关系, 需要占用部分内存读取性能较差 |
| 场景 |     子文档偶尔更新, 查询频繁     |                   子文档更新频繁                   |

建议尽量选择 nested object 来解决问题



### Reindex

指重建所有数据的过程, 一般发生在如下情况

* mapping 设置变更, 比如字段类型发生变化, 分词器字典更新等
* index 设置变更, 比如分片数更改等
* 迁移数据

ES 提供了现成的 API 用于完成该工作

#### _update_by_query

在现有索引上重建

![http://www.miaomiaoqi.cn/images/elastic/search/es_38.png](http://www.miaomiaoqi.cn/images/elastic/search/es_38.png)

```json
DELETE blog_index
# 添加数据
PUT blog_index/doc/1
{
  "title": "Blog Number One",
  "author": "alfred",
  "comments": [
    {
      "username": "lee",
      "date": "2017-01-02",
      "content": "awesome article!"
    },
    {
      "username": "fax",
      "date": "2017-04-02",
      "content": "thanks!"
    }
  ]
}

# 可以看出 version 是 1
GET blog_index/doc/1

# 执行 reindex 操作
POST blog_index/_update_by_query?conflicts=proceed

# 再次查看 version 变为了 2
GET blog_index/doc/1
```



#### _reindex

在其他索引上重建

![http://www.miaomiaoqi.cn/images/elastic/search/es_39.png](http://www.miaomiaoqi.cn/images/elastic/search/es_39.png)

![http://www.miaomiaoqi.cn/images/elastic/search/es_40.png](http://www.miaomiaoqi.cn/images/elastic/search/es_40.png)

```json
DELETE blog_index
# 添加数据
PUT blog_index/doc/1
{
  "title": "Blog Number One",
  "author": "alfred",
  "comments": [
    {
      "username": "lee",
      "date": "2017-01-02",
      "content": "awesome article!"
    },
    {
      "username": "fax",
      "date": "2017-04-02",
      "content": "thanks!"
    }
  ]
}

DELETE blog_new_index
# 执行 reindex 操作
POST _reindex
{
  "source": {
    "index": "blog_index"
  },
  "dest": {
    "index": "blog_new_index"
  }
}

GET blog_index
GET blog_new_index
GET blog_new_index/_search
```

#### Reindex-Task

数据重建的时间受源索引文档规模的影响, 当规模越大时, 所需时间越多, 此时需要通过设定 url 参数 `wait_for_completion` 为 `false` 来异步执行, ES 以 Task 来描述此类执行任务

ES 提供了 Task API 来查看任务的执行进度和相关数据

![http://www.miaomiaoqi.cn/images/elastic/search/es_41.png](http://www.miaomiaoqi.cn/images/elastic/search/es_41.png)

```json
POST blog_index/_update_by_query?conflicts=proceed&wait_for_completion=false

GET _tasks/_qKI6E8_TDWjXyo_x-bhmw:11996
```

### 其他建议

#### 对 Mapping 进行版本管理

包含在代码或者以专门的文件进行管理, 添加好注释, 并加入 Git 版本管理仓库中, 方便回顾

为每个文档增加一个 metadata 字段, 在其中维护一些文档相关的元数据, 方便对数据进行管理

![http://www.miaomiaoqi.cn/images/elastic/search/es_42.png](http://www.miaomiaoqi.cn/images/elastic/search/es_42.png)

#### 防止字段过多

字段过多有如下坏处

* 难于维护, 当字段成百上千时, 基本很难有人能明确知道每个字段的含义
* mapping 的信息存储在 cluster state 里面, 过多的字段会导致 mapping 过大, 最终导致更新变慢

通过设置 `index.mapping.total_fields.limit` 可以限定索引中最大字段数, 默认是 `1000`

可以通过 key/value 的方式解决字段过多的问题, 但并不完美

## ElasticSearch 分布式架构

![http://www.miaomiaoqi.cn/images/elastic/search/es_1.png](http://www.miaomiaoqi.cn/images/elastic/search/es_1.png)

### **Gateway层**

es用来存储索引文件的一个文件系统且它支持很多类型，例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的hdfs分布式存储、亚马逊的S3。它的主要职责是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。

### **Distributed Lucene Directory**

Gateway上层就是一个lucene的分布式框架，lucene是做检索的，但是它是一个单机的搜索引擎，像这种es分布式搜索引擎系统，虽然底层用lucene，但是需要在每个节点上都运行lucene进行相应的索引、查询以及更新，所以需要做成一个分布式的运行框架来满足业务的需要。

### **四大模块组件**

districted lucene directory之上就是一些es的模块，Index Module是索引模块，就是对数据建立索引也就是通常所说的建立一些倒排索引等；Search Module是搜索模块，就是对数据进行查询搜索；Mapping模块是数据映射与解析模块，就是你的数据的每个字段可以根据你建立的表结构通过mapping进行映射解析，如果你没有建立表结构，es就会根据你的数据类型推测你的数据结构之后自己生成一个mapping，然后都是根据这个mapping进行解析你的数据；River模块在es2.0之后应该是被取消了，它的意思表示是第三方插件，例如可以通过一些自定义的脚本将传统的数据库（mysql）等数据源通过格式化转换后直接同步到es集群里，这个river大部分是自己写的，写出来的东西质量参差不齐，将这些东西集成到es中会引发很多内部bug，严重影响了es的正常应用，所以在es2.0之后考虑将其去掉。

### **Discovery, Script**

es4大模块组件之上有 Discovery模块：es是一个集群包含很多节点，很多节点需要互相发现对方，然后组成一个集群包括选主的，这些es都是用的discovery模块，默认使用的是 Zen，也可是使用EC2；es查询还可以支撑多种script即脚本语言，包括mvel、js、python等等。

### **Transport协议层**

再上一层就是es的通讯接口Transport，支持的也比较多：Thrift、Memcached以及Http，默认的是http，JMX就是java的一个远程监控管理框架，因为es是通过java实现的。

### **RESTful接口层**

最上层就是es暴露给我们的访问接口，官方推荐的方案就是这种Restful接口，直接发送http请求，方便后续使用nginx做代理、分发包括可能后续会做权限的管理，通过http很容易做这方面的管理。如果使用java客户端它是直接调用api，在做负载均衡以及权限管理还是不太好做。



## 解析es的分布式架构

### 分布式架构的透明隐藏特性

**ElasticSearch是一个分布式系统，隐藏了复杂的处理机制**

**分片机制：**我们不用关心数据是按照什么机制分片的、最后放入到哪个分片中

**分片的副本：** 每个主分片都可以有副本分片

**集群发现机制(cluster discovery)：**比如当前我们启动了一个es进程，当启动了第二个es进程时，这个进程作为一个node自动就发现了集群，并且加入了进去

**shard负载均衡：**比如现在有10shard，集群中有3个节点，es会进行均衡的进行分配，以保持每个节点均衡的负载请求

**扩容机制:**

垂直扩容：购置新的机器，替换已有的机器

水平扩容：直接增加机器

**rebalance:**

增加或减少节点时会自动负载均衡 shard

**master节点**

主节点的主要职责是和集群操作相关的内容，如创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点。稳定的主节点对集群的健康是非常重要的。

**节点对等**

每个节点都能接收请求
每个节点接收到请求后都能把该请求路由到有相关数据的其它节点上
接收原始请求的节点负责采集数据并返回给客户端(请求转发)

### 分片(primary shard)和副本(replica shard)机制

1. index 包含多个 primary shard(主分片), 每个 primary shard 又有与之对应的 replica shard(副分片), ES是分布式搜索引擎, 每个索引有一个或多个分片, 索引的数据被分配到各个分片上, 相当于一桶水用了 N 个杯子装, N 个杯子中的水都是一样多的(rebalance)
2. 每个 shard 都是一个最小工作单元，**承载部分数据**；每个 shard 都是一个 lucene 实例，有完整的建立索引和处理请求的能力, 因为每个shard 都是一个独立的 lucene 实例, 所以一个分片只能存放 Integer.MAX_VALUE - 128 = 2,147,483,519 个 docs
3. 增减节点时，shard 会自动在nodes中负载均衡有助于横向扩展, 假如 A 节点有 2 个 shard, 会分一个 shard 给新节点, A 节点只剩下 1 个 shard, 这个过程叫做 (relocation), ES 感知后自动完成
4. primary shard 和 replica shard，每个 document 肯定只存在于某一个 primary shard 以及其对应的 replica shard 中，不可能存在于多个 primary shard
5. replica shard 是 primary shard的副本，负责容灾(primary 分片丢失, replica 分片会顶上去成为新的主分片, 同时会根据这个新的主分片创建新的 replica 分片)，以及承担读请求负载提高查询性能(对于一个 query 既可以查主分片也可以查询备份分片, 在合适的范围内多个 replica 性能会更优)
6. primary shard 的数量在创建索引的时候就固定了除非新建索引否则不能调整主分片数(number_of_shards)，replica shard的数量可以随时修改
7. primary shard 的默认数量是 5，replica shard 默认是 1，默认有 10 个 shard，5 个 primary shard，5 个 replica shard
8. **primary shard 不能和自己的 replica shard 放在同一个节点上**（否则节点宕机，primary shard 和副本都丢失，起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上, 如果只有一个节点, 那么 5 个 primary shard会分配在同一台机器上, 5 个 replica 都无法分配 (unassigned), 此时 cluster status 会变成 Yellow

```
GET _cat/health # 查询集群的健康状态
```


### 单节点环境下创建索引分析

```
PUT /myindex
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

这个时候，只会将3个primary shard分配到仅有的一个node上去，另外3个replica shard是无法分配的（一个shard的副本replica，他们两个是不能在同一个节点的）。集群可以正常工作，但是一旦出现节点宕机，数据全部丢失，而且集群不可用，无法接收任何请求。

### 两个节点环境下创建索引分析

将 3 个 primary shard 分配到一个node上去，另外 3 个replica shard分配到另一个节点上

**primary shard 和 replica shard 保持同步, primary 处理写请求同步到 replica 上**

**primary shard 和 replica shard 都可以处理客户端的读请求**


### 水平扩容的过程(增加节点)

1. 扩容后 primary shard 和 replica shard 会自动的负载均衡

2. 扩容后每个节点上的 shard 会减少，那么分配给每个 shard 的 CPU，内存，IO资源会更多，性能提高

3. 扩容的极限，如果有 6 个 shard，扩容的极限就是 6 个节点，每个节点上一个 shard，如果想超出扩容的极限，比如说扩容到9 个节点，那么可以增加 replica shard 的个数

4. 6 个shard，3 个节点，最多能承受几个节点所在的服务器宕机？(容错性)
    任何一台服务器宕机都会丢失部分数据

    为了提高容错性，增加shard的个数：
    9 个shard，(3 个 primary shard，6 个 replicashard)，这样就能容忍最多两台服务器宕机了

**总结：扩容是为了提高系统的吞吐量，同时也要考虑容错性，也就是让尽可能多的服务器宕机还能保证数据不丢失**



### 文档的核心元数据

1. _index:

    说明了一个文档存储在哪个索引中

    同一个索引下存放的是相似的文档(文档的field多数是相同的)

    索引名必须是小写的，不能以下划线开头，不能包括逗号

2. _type:

    表示文档属于索引中的哪个类型

    ES6 开始一个索引下只能有一个 type

    类型名可以是大写也可以是小写的，不能以下划线开头，不能包括逗号

3. _id:

    文档的唯一标识，和索引，类型组合在一起唯一标识了一个文档

    可以手动指定值，也可以由es来生成这个值

### 文档id生成方式

1. 手动指定

    ```
    PUT /index/type/66
    ```

    通常是把其它系统的已有数据导入到es时

2. 由 es 生成 id 值

    ```
    POST /index/type
    ```

    es 生成的 id 长度为 20 个字符，使用的是 base64 编码，URL 安全，使用的是 GUID 算法，**分布式下并发生成 id 值时不会冲突**


### _source元数据分析

其实就是我们在添加文档时 `request body` 中的内容, 默认返回全部 field

指定返回的结果中含有哪些字段: get /index/type/1?_source=name


### 改变文档内容原理解析

替换方式：首先查出原有 document 的数据, 将这条 document 标记为 deleted, 再根据用户数据创建一份新的 document, 用户未提交的数据不会保存

```
PUT /lib/user/4
{
	"first_name": "Jane",
	"last_name": "Lucy",
	"age": 24,
	"about": "I like to collect rock albums",
	"interests": [ "music" ]
}
```

修改方式(partial update)：只修改部分 field 内容, es 接收用的数据, 更新 document 的部分字段, 再将这个 document 标记为 deleted, 再根据更新后的 document 创建一份新的 document

```
POST /lib/user/2/_update
{
    "doc":{
       "age":26
     }
}
```

删除文档：标记为 deleted，随着数据量的增加，es会选择合适的时间删除掉

1. POST 比 PUT 少了一次查询的网络传输, 效率更高一些, POST 全是在 es 内存中完成的
2. POST 方式发生并发冲突的可能性降低, 因为耗时少

### 基于groovy脚本执行partial update

es有内置的脚本支持，可以基于groovy脚本实现复杂的操作

1. 修改年龄

    ```
    POST /lib/user/4/_update
    {
      "script": "ctx._source.age+=1"
    }
    ```

2. 修改名字

    ```
    POST /lib/user/4/_update
    {
      "script": "ctx._source.last_name+='hehe'"
    }
    ```

3. 添加爱好

    ```
    POST /lib/user/4/_update
    {
      "script": {
        "source": "ctx._source.interests.add(params.tag)",
        "params": {
          "tag":"picture"
        }
      }
    }
    ```

4. 删除爱好

    ```
    POST /lib/user/4/_update
    {
      "script": {
        "source": "ctx._source.interests.remove(ctx._source.interests.indexOf(params.tag))",
        "params": {
          "tag":"picture"
        }
      }
    }
    ```

5. 删除文档

    ```
    POST /lib/user/4/_update
    {
      "script": {
        "source": "ctx.op=ctx._source.age==params.count?'delete':'none'",
        "params": {
            "count":29
        }
      }
    }
    ```

6. upsert

    ```
    POST /lib/user/4/_update
    {
      "script": "ctx._source.age += 1",
    
      "upsert": {
         "first_name" : "Jane",
         "last_name" :   "Lucy",
         "age" :  20,
         "about" :       "I like to collect rock albums",
         "interests":  [ "music" ]
      }
    }
    ```

### POST 更新文档 (partial update) 处理并发冲突

使用的是乐观锁: _version

retry_on_conflict:

```
POST /lib/user/4/_update?retry_on_conflict=3
```

重新获取文档数据和版本信息进行更新，不断的尝试更新操作，最多操作的次数就是 retry_on_conflict 的值


### 文档数据路由原理解析

1. 文档路由到分片上：

     一个索引由多个分片构成，当添加(删除，修改)一个文档时，es就需要决定这个文档存储在哪个分片上，这个过程就称为数据路由(routing)

2. 路由算法：

    ```
    shard=hash(routing) % number_of_pirmary_shards
    ```

    示例：一个索引，3 个 primary shard

    2.1 每次增删改查时，都有一个routing值，默认是文档的_id的值

    2.2 对这个routing值使用哈希函数进行计算

    2.3 计算出的值再和主分片个数取余数

    余数肯定在0 - （number_of_pirmary_shards-1）之间，文档就在对应的shard上

    routing值默认是文档的_id的值，也可以手动指定一个值，手动指定对于负载均衡以及提高批量读取的性能都有帮助

3. primary shard个数一旦确定就不能修改了, 一旦改变就找不到原来的 document 了

### 文档增删改内部原理

![http://www.miaomiaoqi.cn/images/elastic/search/es_6.png](http://www.miaomiaoqi.cn/images/elastic/search/es_6.png)

1. 发送增删改请求时，可以选择任意一个节点，该节点就成了协调节点(coordinating node)

2. 协调节点使用路由算法进行路由，然后将请求转到 primary shard 所在节点，该节点处理请求，并把数据同步到它的 replica shard

3. 协调节点对客户端做出响应


### 写一致性原理和quorum机制

1. 任何一个增删改操作都可以跟上一个参数 `consistency`

    可以给该参数指定的值：

    one: (primary shard)只要有一个primary shard是活跃的就可以执行

    all: (all shard)所有的primary shard和replica shard都是活跃的才能执行

    quorum: (default) 默认值，大部分shard是活跃的才能执行 （例如共有6个shard，至少有3个shard是活跃的才能执行写操作）

2. quorum机制：多数shard都是可用的，

    int((primary+number_of_replica)/2)+1

    例如：3个primary shard，1个replica

    int((3+1)/2)+1=3

    至少3个shard是活跃的

    注意：可能出现shard不能分配齐全的情况

    比如：1个primary shard,1个replica
    int((1+1)/2)+1=2
    但是如果只有一个节点，因为primary shard和replica shard不能在同一个节点上，所以仍然不能执行写操作

    再举例：1个primary shard,3个replica,2个节点

    int((1+3)/2)+1=3

    最后:当活跃的shard的个数没有达到要求时，
    es默认会等待一分钟，如果在等待的期间活跃的shard的个数没有增加，则显示timeout

    ```
    put /index/type/id?timeout=60s
    ```

### 文档查询内部原理

![http://www.miaomiaoqi.cn/images/elastic/search/es_7.png](http://www.miaomiaoqi.cn/images/elastic/search/es_7.png)

1. 查询请求发给任意一个节点，该节点就成了协调节点(coordinating node)，该节点使用路由算法算出文档所在的primary shard

2. 协调节点把请求转发给 primary shard 也可以转发给 replica shard (使用轮询调度算法(Round-Robin Scheduling，把请求平均分配至 primary shard 和 replica shard), 区别于增删改只能转发给 primary shard

3. 处理请求的节点把结果返回给协调节点，协调节点再返回给应用程序

特殊情况：请求的文档还在建立索引的过程中，primary shard上存在，但replica shard上不存在，但是请求被转发到了replica shard上，这时就会提示找不到文档


### bulk批量操作的json格式解析

bulk的格式：

```
{action:{metadata}}\n

{requstbody}\n
```

为什么不使用如下格式：

```
[
	{
		"action": {},
		"data": {}
	}
]
```

这种方式可读性好，但是内部处理就麻烦了：

1. 将 json 数组解析为 JSONArray 对象，在内存中就需要有一份 json 文本的拷贝，另外还有一个 JSONArray 对象。

2. 解析 json 数组里的每个 json ，对每个请求中的 document 进行路由

3. 为路由到同一个 shard 上的多个请求，创建一个请求数组

4. 将这个请求数组序列化

5. 将序列化后的请求数组发送到对应的节点上去

耗费更多内存，增加java虚拟机开销, 相比之下 bulk 的格式有如下优势:

1. 不用将其转换为json对象，直接按照换行符切割json，内存中不需要json文本的拷贝

2. 对每两个一组的json，读取meta，进行document路由

3. 直接将对应的json发送到node上去

### 查询结果分析

```json
GET /lib3/user/_search

{
  "took": 419,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "3",
        "_score": 0.6931472,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "lisi"
        }
      },
      {
        "_index": "lib3",
        "_type": "user",
        "_id": "2",
        "_score": 0.47000363,
        "_source": {
          "address": "bei jing hai dian qu qing he zhen",
          "name": "zhaoming"
        }
      }
    ]
  }
}
```

took：查询耗费的时间，单位是毫秒

_shards：共请求了多少个shard

total：查询出的文档总个数

max_score： 本次查询中，相关度分数的最大值，文档和此次查询的匹配度越高，_score的值越大，排位越靠前

hits：默认查询前10个文档

timed_out：

```
GET /lib3/user/_search?timeout=10ms
{
    "_source": ["address","name"],
    "query": {
        "match": {
            "interests": "changge"
        }
    }
}
```




### 多index，多type查询模式

```
GET _search # 查询所有文档

GET /lib/_search # 查询单个索引

GET /lib,lib3/_search # 查询多个索引

GET /*3,*4/_search # 通配符索引

GET /lib/user/_search # 查询类型

GET /lib,lib4/user,items/_search # 指定多个索引和类型

GET /_all/_search # 查询所有索引下的所有文档

GET /_all/user,items/_search # 查询所有索引下的, user, items 类型的文档
```

### 分页查询中的deep paging问题

```json
GET /lib3/user/_search
{
    "from":0,
    "size":2,
    "query":{
        "terms":{
            "interests": ["hejiu","changge"]
        }
    }
}

GET /_search?from=0&size=3
```




### 字符串排序问题

对一个字符串类型的字段进行排序通常不准确，因为已经被分词成多个词条了

**解决方式：对字段索引两次，一次索引分词（用于搜索），一次索引不分词(用于排序)**

```json
GET /lib3/_search

GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "interests": {
        "order": "desc"
      }
    }
  ]
}

GET /lib3/user/_search
{
  "query": {
    "match_all": {
    }
  },
  "sort": [
    {
      "interests.raw": {
        "order": "asc"
      }
    }
  ]
}

DELETE lib3

PUT /lib3
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
    "user": {
      "properties": {
        "name": {
          "type": "text"
        },
        "address": {
          "type": "text"
        },
        "age": {
          "type": "integer"
        },
        "birthday": {
          "type": "date"
        },
        "interests": {
          "type": "text", # 字符串, 分词, 建立倒排索引, 用于搜索
          "fields": {
            "raw": {
              "type": "keyword" # 字符串, 部分词, 用于排序
            }
          },
          "fielddata": true
        }
      }
    }
  }
}
```




### 如何计算相关度分数

**TF/IDF 算法(Term Frequency&Inverse Document Frequency)**

1. **Term Frequency(TF): **词频, 即单词在该文档中出现的次数. **词频越高, 相关度越高**

    搜索内容： hello world

    Hello，I love china.

    Hello world,how are you!

2. **Document Frequency(DF):** 文档频率, 即单词出现的文档数

3. **Inverse Document Frequency(IDF)：**逆向文档频率, 与文档频率相反, 简单理解为 1/DF. **即单词出现的文档数越少, 相关度越高**

    搜索内容：hello world

    hello，what are you doing?

    I like the world.

    hello 在索引的所有文档中出现了500次，world出现了100次

4. **Field-length norm:** 字段长度归约, **文档越短, 相关度越高**

    搜索内容：hello world

    ```json
    {"title":"hello,what's your name?","content":{"owieurowieuolsdjflk"}}
    
    {"title":"hi,good morning","content":{"lkjkljkj.......world"}}
    ```



**BM25模型, 5.x 之后的默认模型**



查看分数是如何计算的：

```
GET /lib3/user/_search?explain=true
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```

查看一个文档能否匹配上某个查询：

```
GET /lib3/user/2/_explain
{
    "query":{
        "match":{
            "interests": "duanlian,changge"
        }
    }
}
```


​    


### Doc Values 解析(正排索引建立)

DocValues其实是Lucene在构建倒排索引时，会额外建立一个有序的正排索引(基于document => field value的映射列表)

```
{"birthday":"1985-11-11",age:23}

{"birthday":"1989-11-11",age:29}

document     age       birthday

doc1         23         1985-11-11

doc2         29         1989-11-11
```

存储在磁盘上，节省内存 

对排序，分组和一些聚合操作能够大大提升性能 

**注意：默认对不分词的字段是开启的，对分词字段无效（需要把fielddata设置为true）**

```
PUT /lib3
{
    "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 0
    },
     "mappings":{
      "user":{
        "properties":{
            "name": {"type":"text"},
            "address": {"type":"text"},
            "age": {
              "type":"integer",
              "doc_values":false
            },
            "interests": {"type":"text"},
            "birthday": {"type":"date"}
        }
      }
     }
}
```





### dynamic mapping策略

**dynamic**:

1. true:遇到陌生字段就 dynamic mapping

2. false:遇到陌生字段就忽略

3. strict:约到陌生字段就报错

```json
PUT /lib8
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
    "user": {
      "dynamic": strict,
      "properties": {
        "name": {
          "type": "text"
        },
        "address": {
          "type": "object",
          "dynamic": true
        },
      }
    }
  }
}
```

**会报错, 因为添加了 age 字段, 并且 mappings 中设置了 dynamic=strict**

```json
PUT  /lib8/user/1
{
  "name": "lisi",
  "age": 20,
  "address": {
    "province": "beijing",
    "city": "beijing"
  }
}
```

**date_detection**: 默认会按照一定格式识别date，比如yyyy-MM-dd

可以手动关闭某个 type 的 date_detection

```json
PUT /lib8
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  },
  "mappings": {
    "user": {
      "date_detection": false,
    }
  }
}
```

**定制 dynamic mapping template(type)**

```json
PUT /my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "en": {
            "match": "*_en",
            "match_mapping_type": "string",
            "mapping": {
              "type": "text",
              "analyzer": "english"
            }
          }
        }
      ]
    }
  }
}
```

**使用了模板, en 后缀**

```json
PUT /my_index/my_type/3
{
  "title_en": "this is my dog"
}
```

**没有使用模板**

```json
PUT /my_index/my_type/5
{
  "title": "this is my cat"
}
```

```json
GET my_index/my_type/_search
{
  "query": {
    "match": {
      "title": "is"
    }
  }
}
```



### 重建索引

一个 field 的设置是不能修改的，如果要修改一个 field，那么应该重新按照新的 mapping，建立一个 index，然后将数据批量查询出来，重新用 bulk api 写入到 index 中。

批量查询的时候，建议采用 scroll api，并且采用多线程并发的方式来 reindex 数据，每次 scroll 就查询指定日期的一段数据，交给一个线程即可。

```json
PUT /index1/type1/4
{
   "content":"1990-12-12"
}

GET /index1/type1/_search

GET /index1/type1/_mapping
```

**报错**

```json
PUT /index1/type1/4
{
   "content":"I am very happy." # 上边自动映射为了 date 类型, 而这次插入的是 string 类型
}
```

**修改 content 的类型为 string 类型, 报错, 不允许修改**

```json
PUT /index1/type1/_mapping
{
  "properties": {
    "content":{
      "type": "text"
    }
  }
}
```

1. 创建一个新的索引，把 index1 索引中的数据查询出来导入到新的索引中
2. 但是应用程序使用的是之前的索引，为了不用重启应用程序，给 index1 这个索引起个别名

```
PUT /index1/_alias/index2
```

**创建新的索引，把 content 的类型改为字符串**

```json
PUT /newindex
{
  "mappings": {
    "type1":{
      "properties": {
        "content":{
          "type": "text"
        }
      }
    }
  }
}
```

使用scroll批量查询

```json
GET /index1/type1/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": ["_doc"],
  "size": 2
}
```

使用 bulk 批量写入新的索引

```json
POST /_bulk
{"index":{"_index":"newindex","_type":"type1","_id":1}}
{"content":"1982-12-12"}
```

将别名 index2 和新的索引关联，应用程序不用重启

```json
POST /_aliases
{
  "actions": [
    {"remove": {"index":"index1","alias":"index2"}},
    {"add": {"index": "newindex","alias": "index2"}}
	]
}

GET index2/type1/_search
```




### 索引不可变的原因

倒排索引包括：

文档的列表，文档的数量，词条在每个文档中出现的次数，出现的位置，每个文档的长度，所有文档的平均长度

索引不变的原因：

不需要锁，提升了并发性能

可以一直保存在缓存中（filter）

节省 cpu 和 io 开销



## 在Java应用中访问ElasticSearch

### 在Java应用中实现查询文档

pom中加入ElasticSearch6.2.4的依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>transport</artifactId>
        <version>6.2.4</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
<build>
    <plugins><!--java编译插件-->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 在Java应用中实现添加文档

```java
"{" +
"\"id\":\"1\"," +
"\"title\":\"Java设计模式之装饰模式\"," +
"\"content\":\"在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。\"," +
"\"postdate\":\"2018-05-20 14:38:00\"," +
"\"url\":\"csdn.net/79239072\"" +
"}"
```

```java
 XContentBuilder doc1 = XContentFactory.jsonBuilder()
                    .startObject()
                    .field("id","3")
                    .field("title","Java设计模式之单例模式")
                    .field("content","枚举单例模式可以防反射攻击。")
                    .field("postdate","2018-02-03")
                    .field("url","csdn.net/79247746")
                    .endObject();        
```

```java
IndexResponse response = client.prepareIndex("index1", "blog", null)
	.setSource(doc1)
	.get();
System.out.println(response.status());
```

### 在Java应用中实现删除文档

```java
DeleteResponse response=client.prepareDelete("index1","blog","SzYJjWMBjSAutsuLRP_P").get();

//删除成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());
```



### 在Java应用中实现更新文档 

```java
UpdateRequest request=new UpdateRequest();
request.index("index1").type("blog")
       .id("2").doc(XContentFactory.jsonBuilder().startObject()
       .field("title","单例模式解读")
       .endObject());
UpdateResponse response=client.update(request).get();

// 更新成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());

upsert方式：

IndexRequest request1 =new IndexRequest("index1","blog","3")
                .source(
                		XContentFactory.jsonBuilder().startObject()
                                .field("id","3")
                                .field("title","装饰模式")
                                .field("content","动态地扩展一个对象的功能")
                                .field("postdate","2018-05-23")
                                .field("url","csdn.net/79239072")
                                .endObject()
                );
        UpdateRequest request2=new UpdateRequest("index1","blog","3")
                .doc(
                		XContentFactory.jsonBuilder().startObject()
                        .field("title","装饰模式解读")
                        .endObject()
                ).upsert(request1);
        
UpdateResponse response=client.update(request2).get();
        
//upsert操作成功返回OK，否则返回NOT_FOUND

System.out.println(response.status());
```




### 在Java应用中实现批量操作

```java
MultiGetResponse mgResponse = client.prepareMultiGet()
	                .add("index1","blog","3","2")
	                .add("lib3","user","1","2","3")
	                .get();
		    
for(MultiGetItemResponse response:mgResponse){
	            GetResponse rp=response.getResponse();
	            if(rp!=null && rp.isExists()){
	                System.out.println(rp.getSourceAsString());
	            }
	        }
	        
bulk：

BulkRequestBuilder bulkRequest = client.prepareBulk();

bulkRequest.add(client.prepareIndex("lib2", "books", "4")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "python")
                        .field("price", 68)
                        .endObject()
                )
        );
bulkRequest.add(client.prepareIndex("lib2", "books", "5")
                .setSource(XContentFactory.jsonBuilder()
                        .startObject()
                        .field("title", "VR")
                        .field("price", 38)
                        .endObject()
                )
        );
//批量执行
BulkResponse bulkResponse = bulkRequest.get();
        
System.out.println(bulkResponse.status());
if (bulkResponse.hasFailures()) {
   System.out.println("存在失败操作");
}    
```





## 集群调优建议