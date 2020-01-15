---
layout: post
title: "MySQL 的 binlog"
categories: [RDBMS]
description:
keywords:
---

* content
{:toc}
## 定义

binlog是记录所有数据库表结构变更（例如CREATE、ALTER TABLE…）以及表数据修改（INSERT、UPDATE、DELETE…）的二进制日志

binlog不会记录SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改，但你可以通过查询通用日志来查看MySQL执行过的所有语句。

**多说一句，如果update操作没有造成数据变化，也是会记入binlog。**

## 误解

binlog只是一类记录操作内容的日志文件

因为binlog称之为二进制日志，很多研发会把这个二进制日志和我们平时在代码里写的代码日志联系在一起。因为我们的代码日志，只有一类记录操作容的文件，并不包含索引文件。然而，这个二进制日志包括两类文件：

* 索引文件（文件名后缀为.index）用于记录哪些日志文件正在被使用
* 日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句)语句事件

假设文件my.cnf中有这么三条配置

```
log_bin：on 打开binlog日志

log_bin_basename：bin文件路径及名前缀（/var/log/mysql/mysql-bin）

log_bin_index：bin文件index（/var/log/mysql/mysql-bin.index）
```

那么你会在文件目录/var/log/mysql/下面发现两个文件mysql-bin.000001和mysql-bin.index。
mysql-bin.index就是我们所说的索引文件，打开瞅瞅，内容是下面这样,记录哪些文件是日志文件。

```
./mysql-bin.000001
```

那么说到日志文件。在innodb里其实又可以分为两部分，一部分在缓存中，一部分在磁盘上。这里业内有一个词叫做刷盘，就是指将缓存中的日志刷到磁盘上。跟刷盘有关的参数有两个个:sync_binlog和binlog_cache_size。这两个参数作用如下

```
binlog_cache_size: 二进制日志缓存部分的大小，默认值32k

sync_binlog=[N]: 表示写缓冲多少次，刷一次盘,默认值为0
```

注意两点:

* binlog_cache_size设过大，会造成内存浪费。binlog_cache_size设置过小，会频繁将缓冲日志写入临时文件。具体怎么设，有兴趣自行查询，我觉得研发大大根本没机会去设这个值的，了解即可。
* sync_binlog=0:表示刷新binlog时间点由操作系统自身来决定，操作系统自身会每隔一段时间就会刷新缓存数据到磁盘，这个性能最好。sync_binlog=1，代表每次事务提交时就会刷新binlog到磁盘。sync_binlog=N,代表每N个事务提交会进行一次binlog刷新。

另外，这里存在一个一致性问题，sync_binlog=N，数据库在操作系统宕机的时候，可能数据并没有同步到磁盘，于是再次重启数据库，会带来数据丢失问题。 
当sync_binlog=1，事务在Commit的时候，数据写入binlog，但是还没写入事务日志(redo log和undo log)。此时宕机，重启数据库，数据被回滚。但是binlog里已经记录，这里存在不一致问题。这个事务日志和binlog一致性的问题，大家可以查询mysql的内部XA协议，该协议就是解决这个一致性问题的。

**binlog是InnoDb独有的**

binlog是以事件形式记录的，这句话通俗点说，就是binlog的内容都是一个个的事件。这块具体的我会在下一篇讲，这篇记住binlog的内容就是一个个事件就行。
注意了，这里的用词，是一个个事件，而不是事务。大家应该知道Innodb和mysiam最显著的区别就是一个支持事务，一个不支持事务。
因此你可以说，binlog是基于事务来记录二进制日志，比如sync_binlog=1,每提交一次事务，就写入binlog。你却不能说binlog是事务日志，binlog不仅记录innodb日志，在myisam中，也一样存在binlog。

## 数据库常用语句

|                        sql 语句                         |                             含义                             |
| :-----------------------------------------------------: | :----------------------------------------------------------: |
|             show variables like 'bin_log';              |                     查看是否开启 binlog                      |
|          show variables like 'binlog_format';           |                       查看 binlog 格式                       |
|                    show master logs;                    |                  查看所有的 binlog 日志列表                  |
|                   show master status;                   | 查看最后一个 Binlog 日志的编号名称, 及最后一个事件结束的位置(pos) |
|                       flush logs;                       |      刷新 Binlog, 此刻开始产生一个新编号的 Binlog 文件       |
|                      reset master;                      |                    清空所有的 Binlog 日志                    |
|                   show binlog events;                   |                    查看第一个 Binlog 日志                    |
|          show binlog events in 'binlog.000030'          |                    查看指定的 Binlog 日志                    |
|     show binlog events in 'binlog.000030' from 931;     |             从指定位置开始, 查看指定 Binlog 日志             |
| show binlog events in 'binlog.000030' from 931 limit 2; |     从指定位置开始, 查看指定 Binlog 日志, 限制查询的条数     |

**Binlog 的 Event_type**

query_event: 与数据操作无关, begin, drop table, truncate table 等

table_map_event: 记录下一个操作所对应的表信息, 存储了数据库名和表名

xid_event: 标记事务提交

write_rows_event: 插入数据, 即 insert 操作

update_rows_event: 更新数据, 即 update 操作

delete_rows_event: 删除数据, 即 delete 操作

## 用途

### 复制

主库有一个单独的log dump线程，将binlog传给从库
从库有两个线程，一个I/O线程，一个SQL线程，I/O线程读取主库传过来的binlog内容并写入到relay log,SQL线程从relay log里面读取内容，写入从库的数据库, 这也是主从的原理。

### 恢复

这里网上有大把的文章指导你，如何利用binlog日志恢复数据库数据。如果你真的觉得自己很有时间，就自己去创建个库，然后删了，再去恢复一下数据，练练手吧。

### 审计

 用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击

## 常识

### binlog常见格式

| format    | 定义                       | 优点                           | 缺点                                                         |
| --------- | -------------------------- | ------------------------------ | ------------------------------------------------------------ |
| Statement | 记录的是修改SQL语句        | 日志文件小, 解决IO, 提高性能   | 准确性差, 对一些系统函数不能准确复制或不能复制, 如now(), uuid()等 |
| row       | 记录的是每行实际数据的变更 | 准确性强, 能准确复制数据的变更 | 日志文件大, 较大的网络IO和磁盘IO                             |
| mixed     | statement和row模式的混合   | 准确性强, 文件大小合适         | 有可能发生主从不一致问题                                     |

**业内目前推荐使用的是row模式**，准确性高，虽然说文件大，但是现在有SSD和万兆光纤网络，这些磁盘IO和网络IO都是可以接受的。那么，大家一定想问，为什么不推荐使用mixed模式，理由如下, 假设master有两条记录，而slave只有一条记录。

![http://www.miaomiaoqi.cn/images/mysql/binlog_1.png](http://www.miaomiaoqi.cn/images/mysql/binlog_1.png)

当在master上更新一条从库不存在的记录时，也就是id=2的记录，你会发现master是可以执行成功的。而slave拿到这个SQL后，也会照常执行，不报任何异常，只是更新操作不影响行数而已。并且你执行命令show slave status，查看输出，你会发现没有异常。但是，如果你是row模式，由于这行根本不存在，是会报1062错误的。

### 怎查看binlog

binlog本身是一类二进制文件。二进制文件更省空间，写入速度更快，是无法直接打开来查看的。

因此mysql提供了命令mysqlbinlog进行查看。

* 一般的statement格式的二进制文件，用下面命令就可以

    ```
    mysqlbinlog mysql-bin.000001
    ```

* 如果是row格式，加上-v或者-vv参数就行，如

    ```
    mysqlbinlog -vv mysql-bin.000001
    ```

### 怎么删binlog

删binlog的方法很多，有三种是常见的

* 使用`reset master`,该命令将会删除所有日志，并让日志文件重新从000001开始。

* 使用命令

	```
	PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr }
	```

	例如

	```
	purge master logs to "binlog_name.00000X" 
	```

	将会清空00000X之前的所有日志文件

* 使用`expire_logs_days=N`选项指定过了多少天日志自动过期清空。

### binlog常见参数

常见参数，列举如下，有个印象就好。

| 参数名                                       | 含义                                           |
| -------------------------------------------- | ---------------------------------------------- |
| log_bin = {on \| off\| base_name}            | 指定是否启用记录二进制日志或者指定一个日志路径 |
| Sql_log_bin = {on \| off}                    | 指定是否启用记录二进制日志                     |
| expire_logs_days                             | 指定自动删除二进制日志的时间, 即日志过期时间   |
| log_bin_index                                | 指定mysql-bin.index文件的路径                  |
| binlong_format = {mixed \| row \| statement} | 指定二进制日志基于什么模式记录                 |
| max_binlog_size                              | 指定二进制日志文件最大值                       |
| binlog_cache_size                            | 指定事物日志缓存区大小                         |
| max_binlog_cache_size                        | 指定二进制日志缓存最大大小                     |
| sync_binlog = {0 \| n}                       | 指定写缓冲多少次, 刷一次盘                     |

## 开源工具监听 binlog

```xml
<!-- binlog 监听与解析: https://github.com/shyiko/mysql-binlog-connector-java -->
<dependency>
    <groupId>com.github.shyiko</groupId>
    <artifactId>mysql-binlog-connector-java</artifactId>
    <version>0.13.0</version>
</dependency>
```

```java
public class BinlogServiceTest {
    public static void main(String[] args) throws IOException {
        BinaryLogClient client = new BinaryLogClient("127.0.0.1", 3306, "root", "miaoqi");
        // client.setBinlogFilename();
        // client.setBinlogPosition();
        client.registerEventListener(event -> {
            EventData data = event.getData();
            if (data instanceof UpdateRowsEventData) {
                // update ad_unit_keyword set keyword = '奔驰' where keyword = '标志';
                System.out.println("Update---------------");
                System.out.println(data.toString());
            } else if (data instanceof WriteRowsEventData) {
                // insert into `ad_unit_keyword`(`unit_id`, `keyword`) values(10, '标志');
                System.out.println("Write----------------");
                System.out.println(data.toString());
            } else if (data instanceof DeleteRowsEventData) {
                System.out.println("Delete---------------");
                System.out.println(data.toString());
            }
        });
        client.connect();
    }
}
```

**binlog 文件会监听数据库中所有的变化, 所以我们可以自定义一份模板文件, 指定我们想要监听的内容**

```
{
  "database": "advertisement",
  "tableList": [
    {
      "tableName": "ad_plan",
      "level": 2,
      "insert": [
        {"column": "id"},
        {"column": "user_id"},
        {"column": "plan_status"},
        {"column": "start_date"},
        {"column": "end_date"}
      ],
      "update": [
        {"column": "id"},
        {"column": "user_id"},
        {"column": "plan_status"},
        {"column": "start_date"},
        {"column": "end_date"}
      ],
      "delete": [
        {"column": "id"}
      ]
    },
    {
      "tableName": "ad_unit",
      "level": 3,
      "insert": [
        {"column": "id"},
        {"column": "unit_status"},
        {"column": "position_type"},
        {"column": "plan_id"}
      ],
      "update": [
        {"column": "id"},
        {"column": "unit_status"},
        {"column": "position_type"},
        {"column": "plan_id"}
      ],
      "delete": [
        {"column": "id"}
      ]
    },
    {
      "tableName": "ad_creative",
      "level": 2,
      "insert": [
        {"column": "id"},
        {"column": "type"},
        {"column": "material_type"},
        {"column": "height"},
        {"column": "width"},
        {"column": "audit_status"},
        {"column": "url"}
      ],
      "update": [
        {"column": "id"},
        {"column": "type"},
        {"column": "material_type"},
        {"column": "height"},
        {"column": "width"},
        {"column": "audit_status"},
        {"column": "url"}
      ],
      "delete": [
        {"column": "id"}
      ]
    },
    {
      "tableName": "creative_unit",
      "level": 3,
      "insert": [
        {"column": "creative_id"},
        {"column": "unit_id"}
      ],
      "update": [
      ],
      "delete": [
        {"column": "creative_id"},
        {"column": "unit_id"}
      ]
    },
    {
      "tableName": "ad_unit_district",
      "level": 4,
      "insert": [
        {"column": "unit_id"},
        {"column": "province"},
        {"column": "city"}
      ],
      "update": [
      ],
      "delete": [
        {"column": "unit_id"},
        {"column": "province"},
        {"column": "city"}
      ]
    },
    {
      "tableName": "ad_unit_it",
      "level": 4,
      "insert": [
        {"column": "unit_id"},
        {"column": "it_tag"}
      ],
      "update": [
      ],
      "delete": [
        {"column": "unit_id"},
        {"column": "it_tag"}
      ]
    },
    {
      "tableName": "ad_unit_keyword",
      "level": 4,
      "insert": [
        {"column": "unit_id"},
        {"column": "keyword"}
      ],
      "update": [
      ],
      "delete": [
        {"column": "unit_id"},
        {"column": "keyword"}
      ]
    }
  ]
}
```

**查看表信息**

```sql
select table_schema, table_name, column_name, ordinal_position from information_schema.columns where table_schema = 'advertisement' and table_name = 'ad_unit_keyword';
```

