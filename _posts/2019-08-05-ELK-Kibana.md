---
layout: post
title: "ELK-Kibana"
categories: [BigData]
description:
keywords:
---

* content
{:toc}     


## Kibana

ElasticSearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene/6753302)的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于[云计算](https://baike.baidu.com/item/云计算/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

## 安装

我的是 mac, 使用命令 `brew install kibana` 可以一键安装, 安装后的目录在 `/usr/local/Cellar/kibana/6.8.1` 配置文件目录为`/usr/local/etc/kibana`

linux 下只要到官网下载压缩包上传解压即可

## 配置

mac 上使用 brew 安装的 es 的配置目录位于 `/usr/local/etc/kibana`

- kibana.yml: kibana 相关配置

    * server.host/server.port: 访问 kibana 用的地址和端口

    - elasticsearch.hosts: 待访问的 elastic search 的地址

推荐专门部署一个 Coordinating Only ES Node, 和 Kibana 在同一台机器上

## 常用功能

**Discover:** 数据搜索查看

**Visualize: **图表只做

**Dashboard:**仪表盘制作

**Timelion:**  时序数据的高级可视化分析

**DevTools:** 开发者工具

**Management: **配置





## 实战

### Airbnb

创建 ES 模型

```json
PUT testairbnb
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "analysis": {
        "analyzer": {
          "autosuggest_analyzer": {
            "filter": [
              "lowercase",
              "asciifolding",
              "autosuggest_filter"
            ],
            "tokenizer": "standard",
            "type": "custom"
          },
          "ngram_analyzer": {
            "filter": [
              "lowercase",
              "asciifolding",
              "ngram_filter"
            ],
            "tokenizer": "standard",
            "type": "custom"
          }
        },
        "filter": {
          "autosuggest_filter": {
            "max_gram": "20",
            "min_gram": "1",
            "token_chars": [
              "letter",
              "digit",
              "punctuation",
              "symbol"
            ],
            "type": "edge_ngram"
          },
          "ngram_filter": {
            "max_gram": "9",
            "min_gram": "2",
            "token_chars": [
              "letter",
              "digit",
              "punctuation",
              "symbol"
            ],
            "type": "ngram"
          }
        }
      }
    }
  },
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "accommodates": {
          "type": "integer"
        },
        "bathrooms": {
          "type": "integer"
        },
        "bed_type": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "bedrooms": {
          "type": "integer"
        },
        "beds": {
          "type": "integer"
        },
        "date_from": {
          "type": "date",
          "format": "yyyyMMdd"
        },
        "date_to": {
          "type": "date",
          "format": "yyyyMMdd"
        },
        "has_availability": {
          "type": "boolean"
        },
        "host_image": {
          "type": "keyword",
          "ignore_above": 256,
          "index": false
        },
        "host_name": {
          "type": "text",
          "analyzer": "autosuggest_analyzer",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "image": {
          "type": "keyword",
          "ignore_above": 256,
          "index":false
        },
        "listing_url": {
          "type": "keyword",
          "ignore_above": 256
        },
        "location": {
          "type": "geo_point"
        },
        "name": {
          "type": "text",
          "analyzer": "autosuggest_analyzer",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "price": {
          "type": "float"
        },
        "property_type": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "room_type": {
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
```

导入数据

```
cat demo_data/airbnb/airbnb.csv | bin/logstash -f demo_data/airbnb/ls.conf
```

Management 创建 Index Patterns 即可通过 Discover 查看 Airbnb 的数据



### 慕课网 Nginx 日志分析

创建 ES 模型

```json
PUT _template/nginx_logs
{
  "index_patterns":"nginx_logs_*",
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas":0,
      "refresh_interval":"30s"
    }
  },
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "@version": {
          "type": "keyword",
          "ignore_above": 256
        },
        "agent": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "auth": {
          "type": "keyword",
          "ignore_above": 256
        },
        "bytes": {
          "type": "long"
        },
        "clientip": {
          "type": "keyword",
          "ignore_above": 256
        },
        "geoip": {
          "properties": {
            "city_name": {
              "type": "keyword",
              "ignore_above": 256
            },
            "continent_code": {
              "type": "keyword",
              "ignore_above": 256
            },
            "country_code2": {
              "type": "keyword",
              "ignore_above": 256
            },
            "country_code3": {
              "type": "keyword",
              "ignore_above": 256
            },
            "country_name": {
              "type": "keyword",
              "ignore_above": 256
            },
            "ip": {
              "type": "keyword",
              "ignore_above": 256
            },
            "latitude": {
              "type": "half_float"
            },
            "location": {
              "type": "geo_point"
            },
            "longitude": {
              "type": "half_float"
            },
            "region_code": {
              "type": "keyword",
              "ignore_above": 256
            },
            "region_name": {
              "type": "keyword",
              "ignore_above": 256
            },
            "timezone": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "host": {
          "type": "keyword",
          "ignore_above": 256
        },
        "hostname": {
          "type": "keyword",
          "ignore_above": 256
        },
        "httpversion": {
          "type": "keyword",
          "ignore_above": 256
        },
        "ident": {
          "type": "keyword",
          "ignore_above": 256
        },
        "params": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "port": {
          "type": "keyword",
          "ignore_above": 256
        },
        "referrer": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "referrer_host": {
          "type": "keyword",
          "ignore_above": 256
        },
        "imooc_type": {
          "type": "keyword",
          "ignore_above": 256
        },
        "imooc_res_id": {
          "type": "keyword",
          "ignore_above": 256
        },
        "request": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "response_status_code": {
          "type": "keyword",
          "ignore_above": 256
        },
        "response_time": {
          "type": "float"
        },
        "upstream_host": {
          "type": "keyword",
          "ignore_above": 256
        },
        "upstream_response_status_code": {
          "type": "keyword",
          "ignore_above": 256
        },
        "upstream_response_time": {
          "type": "float"
        },
        "useragent": {
          "properties": {
            "build": {
              "type": "keyword",
              "ignore_above": 256
            },
            "device": {
              "type": "keyword",
              "ignore_above": 256
            },
            "name": {
              "type": "keyword",
              "ignore_above": 256
            },
            "os": {
              "type": "keyword",
              "ignore_above": 256
            },
            "os_major": {
              "type": "keyword",
              "ignore_above": 256
            },
            "os_minor": {
              "type": "keyword",
              "ignore_above": 256
            },
            "os_name": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "verb": {
          "type": "keyword",
          "ignore_above": 256
        },
        "xforwardedfor": {
          "type": "keyword",
          "ignore_above": 256
        }
      }
    }
  }
}
```

导入数据

```
bin/logstash -f demo_data/imooc_log/ls.conf
```

Management 创建 Index Patterns 即可通过 Discover 查看 imooc 的数据

### 数据分析

两个维度进行分析

* Nginx 访问分析
* 慕课网业务数据分析

#### Nginx 访问分析

访问人数? 流量? 请求 QPS? 统计用

访问来源分布? 访问站点分布? 访问页面排名?

请求响应时间分布? 排查问题

请求响应码分布?

访问地图分布?

#### 慕课网业务数据分析

访问量最大的是视频还是文章? 可以针对性做内容

最受欢迎的视频, 文章是哪些? 可以坐推送

最努力的用户是谁, 兴趣最广泛的用户是谁?

用户哪个时间段最活跃? 可以做活动