---
layout: post
title:  "mysql函数group_concat"
date:   2018-01-08 16:27:12
categories: RDBMS
tags: MySQL
author: miaoqi
---

* content
{:toc} 
## Mysql函数group_concat

* 我们在写统计sql的时候, 有时候需要根据某个字段进行group by操作, 当我们进行group by操作后, 多条数据会合并成一条, 这时候想要展示出其他字段的值, 就可以使用group_concat函数, 假设有如下数据

	username	score
	zhangsan	10
	zhangsan	20
	zhangsan	30
	lisi			50

	根据username进行group by 多条数据就会合成一条

	```
	select * from tbl group by username;
	```

	执行结果如下:

	username	score

	zhangsan	10

	lisi			50

	如果我们想要看下zhangsan和lisi的全部得分怎么办呢? group_concat函数就出场了, 修改sql如下

	```
	select username, group_concat(score) from tbl group by username;
	```

	执行结果如下:

	username	group_concat(score)

	zhangsan	10, 20, 30

	lisi			50

	到此group_concat的用法就介绍完了

* group_concat存在的问题

	在做项目中, 遇到了一个很奇葩的问题, 应该给用户返钱, 但总是漏掉一些用户, 通过查看日志发现, group_concat后的数据少了一部分, 通过查找发现mysql的group_concat默认连接长度为1024字符, 也就是说如果超过1024字符, 它只会显示这么多, 其余部分会被截取丢掉, 导致我们漏掉了数据

	* 解决办法一

		修改连接的配置

		```
		SET GLOBAL group_concat_max_len=102400;
		SET SESSION group_concat_max_len=102400; 
		```

	* 解决办法二

		修改mysql配置文件, 在mysql配置文件中添加如下这句，修改配置文件后记得需要重启mysql服务

		```
		group_concat_max_len = 102400
		```

	虽然上面的办法可以解决问题, 但是使用函数终究性能不会太好, 所以我决定以后避免使用group_concat函数, 而是通过关键列进行简单的索引查找, 在应用层中进行分组, 这样可以避免一些不可预知的问题, 也可以减少mysql的压力
