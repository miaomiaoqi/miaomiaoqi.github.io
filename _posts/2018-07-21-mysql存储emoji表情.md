
---
layout: post
title:  "MySQL存储emoji"
date:   2018-07-21 16:43:23
categories: RDBMS
tags: MySQL
author: miaoqi
---

* content
{:toc}

## 问题

* 在存储用户昵称时遇到如下错误

        java.sql.SQLException: Incorrect string value: ‘\xF0\x9F\x92\x94’ for colum n ‘name’ at row 1 

    使用mysql数据库的时候，如果字符集是UTF-8并且在java服务器上，当存储emoji表情的时候，会抛出以上异常（比如微信开发获取用户昵称，有的用户的昵称用的是emoji的图像）

    这是由于mysql字符集不支持的异常导致的, 在mysql中的utf-8字符集最多只支持3个字节的存储, 如果一个字符的utf8编码占用4个字节(最常见的就是ios中的emoji表情字符），那么在写入数据库时就会报错。

    mysql从5.5.3版本开始，才支持4字节的utf8编码，编码名称为utf8mb4（mb4的意思是max bytes 4），这种编码方式最多用4个字节存储一个字符。
    
    因此，要解决上述异常的发生，需要使用utf8mb4编码。
  
    解决数据库编码后，还需要解决客户端Connection连接对象使用的编码问题。
    
## 解决方式(三种)

1. 从数据库层面进行解决（mysql支持utf8mb4的版本是5.5.3+，必须升级到较新版本）

    1. 修改database, table, column字符集

            ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci; 
            ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; 
            ALTER TABLE table_name CHANGE column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

    1. 修改mysql配置文件my.cnf

            [client] 
            default-character-set = utf8mb4 
            [mysql] 
            default-character-set = utf8mb4 
            [mysqld] 
            character-set-client-handshake = FALSE 
            character-set-server = utf8mb4 
            collation-server = utf8mb4_unicode_ci 
            init_connect=’SET NAMES utf8mb4’

    1. 用的是java服务器，升级或者确保mysql connection版本高于5.1.13否则仍然不能试用utf8mb4 

    1. 服务器端的db配置文件

            jdbc.driverClassName=com.mysql.jdbc.Driver 
            jdbc.url=jdbc:mysql://localhost:3306/database?useUnicode=true&characterEncoding=utf8&autoReconnect=true&rewriteBatchedStatements=TRUE 
            jdbc.username=root 
            jdbc.password=password

        如果升级了mysql-connector，其中的characterEncoding=utf8可以自动被识别为utf8mb4（兼容原来的utf8）, 而 autoReconnection（当数据库连接异常中断时，是否自动重新连接？默认为false）强烈建议配上，忽略这个属性，可能导致缓存缘故, 没有读取到DB最新的配置，导致一直无法试用utf8mb4字符集；

1. 修改数据库连接池配置

    * 如果项目中使用了DataSource数据源，只需要对数据源进行相关配置即可，这里以apache的DBCP数据源为例讲解，在spring框架下配置如下：

            <!-- 数据源 -->
            <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        		<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        		<property name="url" value="jdbc:mysql://${${data-source.prefix}.data-source.host-name}:3306/${${data-source.prefix}.data-source.db-name}?characterEncoding=utf8&amp;autoReconnect=true&amp;failOverReadOnly=false&amp;maxReconnects=10&amp;allowMultiQueries=true" />
        		<property name="username" value="${${data-source.prefix}.data-source.username}" />
        		<property name="password" value="${${data-source.prefix}.data-source.password}" />
        		<property name="maxActive" value="150" />
        		<property name="maxIdle" value="2" />
        		<property name="testOnBorrow" value="true" />
        		<property name="testOnReturn" value="true" />
        		<property name="testWhileIdle" value="true" />
        		<property name="validationQuery" value="select 1" />
        		<!-- 此配置用于在创建Connection对象时执行指定的初始化sql -->
        		<property name="connectionInitSqls">
        			<list>
        				<value>set names 'utf8mb4'</value>
        			</list>
        		</property>
        	</bean>

    * 在springcloud项目中进行配置

            spring:
              datasource:
                connection-init-sqls: set names 'utf8mb4'

    * 该设置的解释引用自mysql参考手册:

            SET NAMES 'charset_name'

            SET NAMES显示客户端发送的SQL语句中使用什么字符集。
            
            因此，SET NAMES 'utf8mb4'语句告诉服务器：“将来从这个客户端传来的信息采用字符集utf8mb4”。它还为服务器发送回客户端的结果指定了字符集。（例如，如果你使用一个SELECT语句，它表示列值使用了什么字符集。）
            
            SET NAMES 'x'语句与这三个语句等价：
            
            mysql> SET character_set_client = x;
            
            mysql> SET character_set_results = x;
            
            mysql> SET character_set_connection = x;

1. 从应用层的方面进行解决 

    1. 在获得数据之后往数据库存之前先进行编码:

            URLEncoder.encode(nickName, “utf-8”);

    1. 当从数据库中取出准备显示的时候进行解码，

            URLDecoder.decode(nickname, “utf-8”); 

    从应用层进行解决的时候建议不要在对象getter，setter方法中直接编码，因为放入对象的时候setter方法将nickname进行编码，当插入数据库的时候相当于从对象中调用getter方法将你参考取出这就将之前setter编码过的nickname又重新解码了，等于未对Nickname进行任何操作。依然会出现以上问题。




