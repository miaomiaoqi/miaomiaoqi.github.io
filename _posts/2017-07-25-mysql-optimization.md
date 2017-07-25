---
layout: post
title:  "MySQL性能调优"
date:   2017-03-01 17:28:00
categories: database
tags: MySQL
author: miaoqi
---

## 常用Linux命令

    ps -ef|grep mysql 
    rpm -qa|grep -i mysql qa:query all;rpm:redhat package manage
    rpm -ivh mysql i:install v:日志 h:hash(进度条)
    
    id z3 查看用户
    
    cat /etc/passwd|grep mysql
    cat /etc/group|grep mysql
    
    chkconfig mysql on:开机自启动
    chkconfig --list|grep mysql:查看mysql自启情况
    
    /etc/inittab 启动状态
    
    第三方软件目录:/opt


## 解决中文乱码

    mysql配置文件:/usr/share/mysql/my-huge.cnf	/etc/my.cnf
    [client]
    default-character-set=utf8
    [mysqld]
    character_set_server=utf8
    character_set_client=utf8
    collation-server=utf8_general_ci
    
    lower_case_table_names=1
    max_connections=1000
    
    [mysql]
    default-character-set=utf8

## MySQL配置文件
    
    1. 二进制日志log-bin:主从复制
    2. 错误日志log-error:默认是关闭的,记录严重的警告和错误信息,每次启动和关闭的详细信息
    3. 查询日志log:默认关闭, 记录查询的sql语句,如果开启会降低mysql的整体性能
    4. 数据文件 存放路径:/var/lib/myqsl
        frm文件-存放表结构
        myd文件-存放表数据
        myi文件-存放表索引

## MySQL架构

    1.连接层:jdbc
    2.服务层:核心服务功能,如sql接口,缓存查询,sql的分析优化及部分内置函数的执行.
    3.引擎层:存储引擎(插拔式)
    4.存储层:硬盘

    存储引擎:常用MyISAM和InnoDB
    查看引擎:show engines;show variables like '%storage_engine%';
    
    对比项	    MyISAM              InnoDB
    主外键	    不支持                 支持
    事物      不支持                 支持
    行表锁	表锁, 即使操作一条记录也会锁住整个表,不适合高并发操作	行锁,操作时只锁某一行,不对其他行有影响,适合高并发
    缓存	只缓存索引, 不缓存真实数据				不仅缓存索引还缓存真实数据,对内存要求高,内存决定性能
    表空间	小							大
    关注点	性能							事务

## 性能

    性能下降SQL慢: 查询语句写的不好
    执行时间长: 索引失效(单值 create index idx_user_name on user(name),复合 create index idx_user_nameEmail on user(name,email))
    等待时间长: 关联查询太多join(设计缺陷或不得已需求)
    			服务器调优及各个参数设置(缓冲, 线程数等)


## JOIN查询

    1.SQL执行顺序:  
    手写
        select distinct <select_list> from <left_table> <join_type> join <right_table> on <join_condition>
    	where <where_condition> group by <group_by_list> having <having_condition> order by <order_by_condition> limit <limit number>
    机读
        from <left_table> on <join_condition> <join_type> join <right_table> where <where_condition> group by <group_by_list>
    	having <having_condition> select distinct <select_list> order by <order_by_condition> limit <limit_number>
    	
    http://www.cnblogs.com/qanholas/archive/2010/10/24/1859924.html
    2.Join图

    练习:
    create table tbl_dept(
        id int(11) not null auto_increment,
        dept_name varchar(30) default null,
        ioc_add varchar(40) default null,
        primary key(id)
    )engine=innodb auto_increment=1 default charset=utf8;
    
    create table tbl_emp(
        id int(11) not null auto_increment,
        name varchar(20) default null,
        dept_id int(11) default null,
        primary key(id),
        key fk_dept_id(dept_id)
        #constraint fk_dept_id foreign key (dept_id) references tbl_dept(id)
    )engine=innodb auto_increment=1 default charset=utf8;
    
    insert into tbl_dept(dept_name,ioc_add) values('RD',11);
    insert into tbl_dept(dept_name,ioc_add) values('HR',12);
    insert into tbl_dept(dept_name,ioc_add) values('MK',13);
    insert into tbl_dept(dept_name,ioc_add) values('MIS',14);
    insert into tbl_dept(dept_name,ioc_add) values('FD',15);
    
    insert into tbl_emp(name,dept_id) values('z3',1);
    insert into tbl_emp(name,dept_id) values('z4',1);
    insert into tbl_emp(name,dept_id) values('z5',1);
    insert into tbl_emp(name,dept_id) values('w5',2);
    insert into tbl_emp(name,dept_id) values('w6',2);
    insert into tbl_emp(name,dept_id) values('s7',3);
    insert into tbl_emp(name,dept_id) values('s8',4);
    insert into tbl_emp(name,dept_id) values('s9',51);
    
    
    select * from tbl_dept, tbl_emp;
    select * from tbl_emp e inner join tbl_dept d on e.dept_id = d.id;
    select * from tbl_emp e, tbl_dept d on e.dept_id = d.id;
    select * from tbl_emp e left join tbl_dept d on e.dept_id = d.id;
    select * from tbl_emp e right join tbl_dept d on e.dept_id = d.id;
    select * from tbl_emp e left join tbl_dept d on e.dept_id = d.id where d.id is null;
    select * from tbl_emp e right join tbl_dept d on e.dept_id = d.id where e.dept_id is null;
    #select * from tbl_emp e full outer join tbl_dept d on e.dept_id = d.id; oracle
    select * from tbl_emp e left join tbl_dept d on e.dept_id = d.id union select * from tbl_emp e right join tbl_dept d on e.dept_id = d.id;
    select * from tbl_emp e left join tbl_dept d on e.dept_id = d.id where d.id is null union select * from tbl_emp e right join tbl_dept d on e.dept_id = d.id where e.dept_id is null;

    union,union all合并结果集
    union:去重,按照字段排序,效率低
    union all:不去重,不排序,效率高


## 索引

    索引:索引(index)是帮助mysql高效获取数据的数据结构,本质:数据结构
    "排好序的快速查找数据结构"(排序+查找)order by+where
    B树索引:索引单独维护了一个索引文件, 索引文件使用二叉树结构, 指向对象的数据

    优势:
        1. 提高数据检索的效率, 降低数据库的IO
        2. 降低数据排序成本, 降低了CPU消耗
    劣势:
        1. 实际上索引也是一张表, 保存了主键与索引字段, 并指向实体表记录, 所以索引列也是占空间的
        2. 大大提高了查询速度, 会降低更新表的速度, 如对表insert, update和delete.
            因为更新表时,mysql不仅要保存数据, 还要保存一下索引文件每次更新添加了索引列的字段.
    		都会调整因为更新所带来得键值变化后的索引信息
        3. 索引只是提高效率的一个因素, 如果你的mysql有大数据的表, 就需要花时间研究建立最优秀的索引, 或优化
    分类:
        1. 单值索引: 即一个索引只包含单列, 一个表可以有多个单列索引
        2. 唯一索引: 索引列的值必须唯一, 但允许有空值
        3. 复合索引: 即一个索引包含多个列
    语法:
        创建 
        create [unique] index indexName on mytable(columnname(length));
        alter mytable add [unique] index [indexName] on (columnname(length));
        删除 
        drop index [indexName] on mytable;
        查看 
        show index from table_name;

    alter table tbl_name add primary_key(column_list)	该语句添加一个主键, 这意味着索引值必须唯一, 且不能为空
    alter table tbl_name add unique index index_name(column_list) 这条语句创建索引的值必须是唯一的, 但可以为NULL, NULL可能出现多次
    alter table tbl_name add idnex index_name(column_list) 添加普通索引, 索引值可以出现多次
    alter table tbl_name add fulltext index_name(column_list) 添加全文索引

    mysql索引结构:
    	1.BTree索引 3层的b+树可以表示上百万条数据
    	2.Hash索引
    	3.full-text索引
    	4.R-Tree索引

    哪些情况建索引:
    	1. 主键自动建立唯一索引
    	2. 频繁作为查询条件的字段
    	3. 查询中与其他表关联的字段,外键关系建立索引
    	4. 频繁更新得字段不适合创建索引 - 更新时不单单更新了记录还会更新索引
    	5. where条件里用不到的字段不要创建索引
    	6. 单值索引和组合索引的选择?(高并发情况下倾向创建组合索引)
    	7. 查询中排序的字段, 排序字段若通过索引会大大提高查询速度
    	8. 查询中统计或者分组的字段
	      
    哪些情况不要建索引:
    	1. 表记录太少
    	2. 经常增删改的表 - 提高了查询速度, 同时却会降低更新表的速度
    	3. 如果某个数据列包含许多重复的内容, 为它建立索引就没有太大的实际效果

## 性能分析:

    1. MySql Query Optimizer:
        通过计算分析系统中收集的统计信息, 为客户端提供请求的Query提高它认为最优的执行计划(不见得是DBA认为最优的)
        当客户端向MySql请求一条Query, 命令解析器模块完成请求分类, 区别出是select并转发给mysql query optimizer, 首先会对整条Query进行优化,处理掉一些常量表达式的预算, 直接换算成常量值, 并对Query中的查询进行简化和转换, 如去掉一些无用或显而易见的条件, 结构调整,等, 然后分析Hint
	(如果有), 看Hint信息是否可以完全确定该Query的执行计划, 如果没有Hint或Hint信息不足以完全确定执行计划, 则会读取所涉及对象的统计信息, 根据
	Query进行写相应的计算分析, 最后得出执行计划.
	2. MySql的常见瓶颈:
        CPU: CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
        IO: 磁盘I/O瓶颈发生装入数据远大于内存容量的时候
        服务器硬件性能: top, free, iostat和vmstat来查看系统性能
	3. Explain
        是什么: 使用explain关键字可以模拟优化器执行SQL查询语句, 从而知道MySql是如何处理你的SQL语句, 分析你的查询语句或是表结构的性能瓶颈.
        干什么: 表的读取顺序, 数据读取操作的操作类型, 哪些索引可以使用, 哪些索引被实际使用, 表之间的引用, 每张表有多少行被优化器查询
        怎么用: explain + sql语句
		
		id select_type table type possible_keys key key_len ref rows Extra
		名词解释:
			id*: select查询的序列号, 包含一组数字, 表示查询中执行select子句或操作表的顺序
            三种情况: 
                id相同, 执行顺序由上至下
                id不同, 如果是子查询, id的序号会递增, id值越大优先级越高, 越先被执行
                id相同不同,同时存在

			select_type: 查询类型
                SIMPLE: 简单的select查询, 查询中不包含子查询或union
                PRIMARY: 查询中包含任何复杂的子部分, 最外层查询则被标记为
                SUBQUERY: 子查询
                DERIVED: 在from列表中包含的子查询被标记为derived, mysql会递归执行这些子查询, 把结果放在临时表
                UNION: 若第二个select出现在union之后, 标记为union, 若union包含在from子句的子查询中, 外层select被标记为derived
                UNION RESULT: 从union表获取结果的select

			table: 显示这一行的数据是关于哪张表

			type*: 显示查询使用了何种类型
                最好到最差
                system > const > eq_ref > ref > range > index > all
                system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_query > index_subquery > range > idnex > all
                一般来说,保证查询能到range级别,最好能到ref级别
                system: 表只有一行记录(等于系统表), 这是const类型的特例, 平时不会出现
                const: 表示通过索引一次就找到了, 用于比较primary key或者unique索引, 因为只匹配一行数据, 所以速度很快.(主键)
                eq_ref: 唯一索引扫描, 对于每个索引键, 表中只有一条记录与之匹配
                ref: 非唯一索引扫描, 返回匹配某个单独值的所有行本质上也是一种索引访问, 但是会匹配多个值
                rang: 只检索给定范围的行, 使用一个索引来选择行. key列显示使用了哪个索引
                index: full index scan, index与all的区别为indx类型只遍历索引树, 而all是遍历数据文件.
                all: 全数据文件扫描

            possible_keys: 显示可能应用在这张表的索引, 一个或多个, 查询涉及的字段上若存在索引, 则该索引将被列出, 但不一定被查询实际使用

            key*: 实际使用的索引, 若为NULL, 则没有使用索引查询中若使用了覆盖索引, 则该索引只出现在key列表中覆盖索引: 查询列与复合索引的个数顺序一致, 则自动按照复合索引去查询

            key_len: 表示索引中使用的字节数, 可通过该列计算查询中使用索引的长度, 在不损失精度的情况下, 长度越短越好,
				 key_len显示的值为索引字段的最大可能长度, 并非实际使用长度, 即key_len是根据表定义计算而得, 不是通过表内检索得出.

            ref: 显示索引的哪一列被使用了, 如果可能的话, 是一个常数, 哪些列或常量被用于查找索引列上的值

            rows*: 根据表统计信息及索引选用情况, 大致估算出找到所需的记录所需读取的行数

            Extra*: 包含不适合在其他列显示但十分重要的信息
                1. using filesort: 说明mysql会对数据使用一个外部的索引排序, 而不是按照表内的索引顺序进行读取, 称为"文件排序"
                2. using temporary: 使用了临时表保存中间结果, mysql在对查询结果排序时使用临时表. 常见于排序order by和分组查询group by
                3. using index: 表示相应的select操作使用了覆盖索引, 避免访问了表的数据行.
    				如果同时出现using where 表明索引被用来执行索引键值的查找
    				如果没有出现using where 表明索引用来读取数据而非执行查找操作
    				覆盖索引: select的数据列只用从索引表就能够取得, 不用读取数据表.条件是查询列的顺序要和索引顺序相同
                4. using where: 表明使用了where过滤
                5. using join buffer: 使用了连接缓存
                6. impossible where: where子句的值总是false, 不能用来获取任何元组
                7. select table optimized away: 
                8. distinct: 

## 索引优化:
    索引分析:
        单表:
        sql:
        create table if not exists `article`(
            id int(10) unsigned not null primary key auto_increment,
            author_id int(10) unsigned not null,
            category_id int(10) unsigned not null,
            views int(10) unsigned not null,
            comments int(10) unsigned not null,
            title varchar(255) not null,
            content text not null
        );
        insert into article(author_id, category_id, views, comments, title, content) values
        (1,1,1,1,'1','1'),
        (2,2,2,2,'2','2'),
        (1,1,3,3,'3','3');
        案例:
        1. category_id为1且comments大于1的情况下,views最多的article_id
			
        EXPLAIN SELECT id, author_id from article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1; 
        #ALTER TABLE article ADD INDEX idx_article_ccv(category_id,comments,views);
        #CREATE INDEX idx_article_ccv ON article(category_id,comments,views);
        #DROP INDEX idx_article_ccv ON article;
        #CREATE INDEX idx_article_cv ON article(category_id,views);

        两表:
        sql:
        create table if not exists `class`(
            id int not null auto_increment,
            card int not null,
            primary key(id)
        );
        create table if not exists book(
            bookid int not null auto_increment,
            card int not null,
            primary key(bookid)
        );
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        insert into class(card)values(floor(1+(rand()*20)));
        
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));
        insert into book(card)values(floor(1+(rand()*20)));

        explain select * from class left join book on class.card = book.card;

        alter table book add index Y(card);

        drop index Y on book;
        
        alter table class add index Y(card);

        三表:
        sql:
        create table if not exists phone(
            phoneid int not null auto_increment,
            card int not null,
            primary key(phoneid)
        );

        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));
        insert into phone(card) values(floor(1+(rand()*20)));

        explain select * from class inner join book on class.card = book.card inner join phone on book.card = phone.card;
        explain select * from class left join book on class.card = book.card left join phone on book.card = phone.card;

        alter table phone add index z(card);
        alter table book add index x(card);

        结论:
            尽可能减少join语句中的nestedloop的循环总次数:"永远用小结果集驱动大结果集"
            优先优化嵌套循环的内层循环
            保证join语句中被驱动表上join条件字段已经被索引
            当无法保证被驱动表的join条件字段被索引且内存资源充足的条件下, 不要吝啬joinbuffer的设置

        create table staffs(
            id int primary key auto_increment,
            name varchar(24) not null default '',
            age int not null default 0,
            pos varchar(20) not null default '',
            add_time timestamp default current_timestamp
        );

        insert into staffs(name, age, pos, add_time)values('z3',22,'manager',now());
        insert into staffs(name, age, pos, add_time)values('july',23,'dev',now());
        insert into staffs(name, age, pos, add_time)values('2000',23,'dev',now());
        alter table staffs add index idx_staffs_nameAgePos(name, age, pos);

        索引失效原因:
            1. 全值匹配
                explain select * from staffs where name = 'july';
                explain select * from staffs where name = 'july' and age = 25;
                explain select * from staffs where name = 'july' and age = 25 and pos = 'dev';
            2. 最佳左前缀法则: 如果索引了多列, 要遵守最左前缀法则, 指的是查询从索引的最左前缀开始并且不跳过索引中的列.
                where name = 'july' and pos = 'dev';
            3. 不在索引列上左任何操作(计算, 函数, 类型转换), 会导致索引失效而转向全表扫描
                explain select * from staffs where left(name,3) = 'july';
            4. 存储引擎不能使用索引中范围条件右边的列
                explain select * from staffs where name = 'july' and age > 23 and pos = 'dev';
            5. 尽量使用覆盖索引, 减少select *
                explain select * from staffs where name = 'july' and age = 23 and pos = 'dev';		
            6. mysql在使用不等于(!=或者<>)的时候无法使用索引会导致全表扫描
            7. is null和is not null也无法使用索引
            8. like以通配符开头('%abc')mysql索引会失效变成全表扫描
                explain select * staffs where name like '%july%';
                explain select * staffs where name like '%july';
                explain select * staffs where name like 'july%';

                create table tbl_user(
                    id int auto_increment,
                    name varchar(20) default null,
                    age int default null,
                    email varchar(20) default null,
                    primary key(id)
                );
                insert into tbl_user(name, age, email) values('1aa1',21,'a@qq.com');
                insert into tbl_user(name, age, email) values('2aa2',221,'b@qq.com');
                insert into tbl_user(name, age, email) values('3aa3',231,'c@qq.com');
                insert into tbl_user(name, age, email) values('4aa4',251,'d@qq.com');
                insert into tbl_user(name, age, email) values('aa',251,'e@qq.com');

                create index idx_user_nameAge on tbl_user(name, age);

                explain select name, age from tbl_user where name like '%aa%';
                
                explain select id from tbl_user where name like '%aa%';
                explain select name from tbl_user where name like '%aa%';
                explain select age from tbl_user where name like '%aa%';
                
                explain select id, name from tbl_user where name like '%aa%';
                explain select id, name, age from tbl_user where name like '%aa%';
                explain select name, age from tbl_user where name like '%aa%';
                
                explain select * from tbl_user where name like '%aa%';
                explain select id, name, age, email from tbl_user where name like '%aa%';

                利用覆盖索引解决like的%问题

            9. 字符串不加单引号索引失效
                select * from staffs where name = '2000';
                select * from staffs where name = 2000;
            10. 少用or, 用它来连接时索引会失效
                explain select * from staffs where name = 'july' or name = 'z3';

            练习: 假设index(a,b,c)
				where a = 3 使用到a
				where a = 3 and b = 5 使用到a,b
				where a = 3 and b = 5 and c = 4 使用到a,b,c
				where b = 3 或者 b = 3 and c = 4 或者 where c = 4 不能
				where a = 3 and c = 5 用到a
				where a = 3 and b > 4 and c = 5 使用到a,b
				where a = 3 and b like 'kk%' and c = 4 使用到a,b,c
				where a = 3 and b like '%kk' and c = 4 使用到a
				where a = 3 and b like '%kk%' and c = 4 使用到a
				where a = 3 and b like 'k%kk%' and c = 4 使用到a,b,c

            create table test03(
                id int primary key auto_increment,
                c1 char(10),
                c2 char(10),
                c3 char(10),
                c4 char(10),
                c5 char(10)
            );

            insert into test03(c1,c2,c3,c4,c5) values('a1','a2','a3','a4','a5');
            insert into test03(c1,c2,c3,c4,c5) values('b1','b2','b3','b4','b5');
            insert into test03(c1,c2,c3,c4,c5) values('c1','c2','c3','c4','c5');
            insert into test03(c1,c2,c3,c4,c5) values('d1','d2','d3','d4','d5');
            insert into test03(c1,c2,c3,c4,c5) values('e1','e2','e3','e4','e5');

            select * from test03;
            
            create index idx_test03_c1234 on test03(c1,c2,c3,c4);
            
            explain select * from test03 where c1 = 'a1';
            explain select * from test03 where c1 = 'a1' and c2 = 'a2';
            explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c3 = 'a3';
            explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c3 = 'a3' and c4 = 'a4';
            
            1)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c4 = 'a4' and c3 = 'a3';查询优化器(都是常量)
            2)explain select * from test03 where c4 = 'a4' and c3 = 'a3' and c2 = 'a2' and c1 = 'a1';
            3)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c3 > 'a3' and c4 = 'a4';用了 c1,c2,c3
            4)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c4 > 'a4' and c3 = 'a3';用了 c1,c2,c3,c4(查询优化器)
            5)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c4 = 'a4' order by c3; c3作用是排序
            6)explain select * from test03 where c1 = 'a1' and c2 = 'a2' order by c3;
            7)explain select * from test03 where c1 = 'a1' and c2 = 'a2' order by c4; 出现了filesort
            8.1)explain select * from test03 where c1 = 'a1' and c5 = 'a5' order by c2,c3; 查找用c1, 排序用c2,c3
            8.2)explain select * from test03 where c1 = 'a1' and c5 = 'a5' order by c3,c2; 查找用c1, 排序没有用到
            9)explain select * from test03 where c1 = 'a1' and c2 = 'a2' order by c2,c3; 查找用c1, c2, 排序用c2,c3
            10)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c5 = 'a5' order by c2,c3; 查找用c1, c2, 排序用c2,c3
            11)explain select * from test03 where c1 = 'a1' and c2 = 'a2' and c5 = 'c5' order by c3,c2;
            12)explain select * from test03 where c1 = 'a1' and c4 = 'a4' group by c2,c3;
            13)explain select * from test03 where c1 = 'a1' and c4 = 'a4' group by c3,c2;分组之前必排序, 会产生临时表
            
            建议: 对于单键索引, 尽量选择针对当前query过滤性更好的条件在组合索引时, 当前query过滤性最好的字段在索引顺序中越靠左越好在组合索引时,   
            尽量选择可以能够包含当前query中where中更多的字段尽可能通过分析统计信息调整sql达到选择合适索引的目的

        分析
        1. 观察, 至少跑1天, 检查生产的慢sql情况
        2. 开启慢查询日志, 设置阙值, 比如超过5秒的就是慢sql, 并将它抓取
        3. explain+慢sql分析
        4. show profile
        5. 运维经理或dba进行sql数据库服务器的参数调优

        查询截取分析:
        1. 慢查询的开启并捕获
        2. explain + 慢sql分析
        3. show profile查询sql在mysql服务器里面的执行细节和生命周期
        4. sql数据库服务器的参数调优

        for(int i = 0; i < 5; i++){
            for(int j = 0; j < 1000; j++){
                
            }
        }
        
        for(int i = 0; i < 1000; i++){
            for(int j = 0; j < 5; j++){
                
            }
        }

        * 小表驱动大表

        select * from A where id in (select id from B)
        等价于:
        for select id from B
        for select * from A where A.id = B.id
        当B表的数据集必须小于A表的数据集时, 用in优于exists
	
        select * from A where exists (select 1 from B where B.id = A.id)
        等价于:
        for select * from A
        for select * from B where B.id = A.id
        当A表的数据集小于B表的数据集时, 用exists优于in

        A表与B表的id字段应建立索引
        
        exists:
        select ... from table where exists(subquery)
        理解为: 将主查询的数据, 放到子查询中做条件验证, 根据验证结果(true或false)来决定主查询的数据结果是否得以保留

        order by关键字优化:
        create table tblA(
            id int primary key auto_increment,
            age int,
            birth timestamp not null
        );

        insert into tblA(age,birth) values(22,now());
        insert into tblA(age,birth) values(23,now());
        insert into tblA(age,birth) values(24,now());
        	
        create index idx_A_ageBirth on tblA(age, birth);
        	
        explain select * from tblA where age > 20 order by age;
        explain select * from tblA where age > 20 order by age,birth;
        explain select * from tblA where age > 20 order by birth;
        explain select * from tblA where age > 20 order by birth,age;
        explain select * from tblA order by birth;
        explain select * from tblA where birth > '2017-01-01 00:00:00' order by birth;
        explain select * from tblA where birth > '2017-01-01 00:00:00' order by age;
        explain select * from tblA order by age asc, birth desc; order by 默认是升序

        index效率高, filesort效率低

        order by满足两种情况, 会使用index排序:
            1. order by语句使用索引最左前列
            2. 使用where子句与order by子句条件组合满足索引最前列

        如果不在索引列上, filesort有两种算法:
        1. 双路排序: mysql4.1之前是使用双路排序, 字面意思是两次扫描磁盘, 最终得到数据
        读取行指针和order by列, 对他们进行排序, 然后扫描已经排序好的列表, 按照列表中的值重新
        从列表中的值从新从列表中读取对应的数据进行输出
        2. 单路排序: 

        * 为排序使用索引:
            mysql两种排序方式: 文件排序和扫描有序索引排序
            mysql能为排序与查询使用相同索引
        
            key a_b_c(a,b,c);
            
            order by a;
            order by a, b;
            order by a, b, c;
            order by a desc, b desc, c desc;
            
            where a = const order by b, c;
            where a = const and b = const order by c;
            where a = const and b > const order by b, c;
            
            不能使用:
            order by a asc, b desc, c desc  排序不一致
            where a = const order by c  丢失b索引
            where a = const order by a, d;  d不是索引的一部分
            where a in (...) order by b, c;  对于排序来说, 多个相等条件也是范围查询

        * group by:实质是先排序后进行分组, 遵照索引建的最佳左前缀
            当无法使用索引列, 增大max_length_for_sort_data, sort_buffer_size
            其他均和order by一样

        * 慢查询日志:
            默认情况下没有开启慢查询日志
            show variables like '%slow_query_log%';
            set global slow_query_log = 1; 只对本次有效
            show variables like 'long_query_time%';
            show global variables like 'long_query_time%';
            set global long_query_time = 3;
            
            select sleep(4);
            
            show global status like '%slow_queries%';
            
        mysqldumpslow --help

        # 批量插入数据
        create table dept(
            id int primary key auto_increment,
            deptno int not null default 0,
            dname varchar(20) not null default '',
            ioc varchar(20) not null default ''
        );

        create table emp(
            id int primary key auto_increment,
            empno int not null default 0,
            ename varchar(20) not null default '',
            job varchar(20) not null default '',
            mgr int not null default 0,
            hiredate date not null,
            sal decimal(7,2) not null,
            comm decimal(7,2) not null,
            deptno int not null default 0
        );

        show variables like 'log_bin_trust_function_creators';
        
        set global log_bin_trust_function_creators = 1;
        
        delimiter $$
        create function rand_string(n int) returns varchar(255)
        begin
        declare chars_str varchar(100) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        declare return_str varchar(255) default '';
        declare i int default 0;
        while i < n do
        set return_str = concat(return_str,substring(chars_str,floor(1+rand()*52),1));
        set i = i + 1;
        end while;
        return return_str;
        end $$
        	
        delimiter $$
        create function rand_num() returns int(5)
        begin
        declare i int default 0;
        set i = floor(100+rand()*10);
        return i;
        end $$
        
        # drop function rand_num;
        
        delimiter $$
        create procedure insert_emp(in start int(10), in max_num int(10))
        begin
        declare i int default 0;
        set autocommit = 0;
        repeat
        set i = i + 1;
        insert into emp(empno, ename, job, mgr, hiredate, sal, comm, deptno) values((start + i), rand_string(6), 'salesman', 0001, curdate(), 2000, 400, rand_num());
        until i = max_num
        end repeat;
        commit;
        end $$
        
        delimiter $$
        create procedure insert_dept(in start int(10), in max_num int(10))
        begin
        declare i int default 0;
        set autocommit = 0;
        repeat
        set i = i + 1;
        insert into dept(deptno, dname, ioc) values((start + i), rand_string(10), rand_string(8));
        until i = max_num
        end repeat;
        commit;
        end $$
        
        delimiter ;
        call insert_dept(100, 10);
    call insert_emp(100001, 500000);

## profile:

    是什么: 是mysql提供可以用来分析当前会话中语句执行的资源消耗情况, 可以用于sql调优的测量
    默认情况: 处于关闭状态, 并保存最近15次的运行结果
    分析步骤: 
        show variables like 'profiling';
        set profiling = on;
    
        select * from emp group by id%10 limit 150000;
        select * from emp group by id%20 order by 5;
            
        show profiles;
        show profile cpu, block io for query 5;
            
        converting HEAP	to MyISAM:查询结果太大, 往磁盘上挪了
        creating tmp table 创建临时表
        copying to tmp table on disk 把内存中临时表复制到磁盘
        locked
        
        全局查询日志
        set global general_log = 1;
        set global log_output = 'TABLE';

## 锁

    对MyISAM表的读操作(加读锁), 不会阻塞其他进程对同一表的读请求, 但会阻塞对同一表的写请求. 只有当读锁释放后, 才会执行其他进行的写操作.
    对MyISAM表的写操作(加写锁), 会阻塞其他进程对同一表读和写操作, 只有当写锁释放后, 才会执行其他进程的读写操作.

    简而言之, 就是读锁会阻塞写, 但是不会阻塞读, 而写锁则会把读和写都阻塞
        
    show status like 'table%';
    immediate: 代表立即获得锁的情况
    waited: 代表等待次数