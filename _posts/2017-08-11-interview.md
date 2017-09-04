---
layout: post
title:  "Python语法"
date:   2017-08-10 15:12:38
categories: python
tags: Python
author: miaoqi
---
                                    
## 面试题

### 非技术
1. 为什么离职


### 业务
1. 在项目开发中遇到的难点有哪些？都是怎么解决的？
2. 前后端分离的优缺点


### 数据结构基础知识
1. 算法的复杂度
2. 双层for循环累加一个二维数组的算法复杂度
3. 一个递归算阶乘的算法复杂度
4. 写一个程序，判断字符串是否对称，并给出复杂度


### Java基础
1. List、Set、Map接口都有哪些实现？

    List:
        |-- ArrayList
        |-- LinkedList
        |-- Vector
    Set:
        |-- HashSet
        |-- TreeSet
    Map:
        |-- HashMap
        |-- LinkedHashMap
2. HashMap中负载因子的作用
3. ConcurrentHashMap是如何实现的线程安全？
4. StringBuffer 和 StringBuilder的区别
5. 接口和抽象类的异同

    接口: 所有方法都是抽象的
    抽象类: 可以有带实现的方法
6. JDK1.8新特性
7. 设计模式
8. 单例模式的使用场景
9. Servlet的生命周期
10. Servlet是否线程安全的？其类内有两个类变量，name和age，会不会有线程问题？如何解决？
11. JSP的九大内置对象
12. Session的生命周期
13. Session如何实现的保持会话
14. Session如何在浏览器端禁用cookie的情况下保持回话
15. synchronized关键字可以使用的场景：类？方法？变量？代码块？
16. 谈谈volatile关键字
17. 泛型的使用场景及优势
18. 对象中equals方法的作用
19. 重定向与请求转发的区别
20. newCachedThreadPool线程池何时回收闲置的线程？若长时间没有任务，所有的线程会被回收吗？
21. 线程的状态切换
22. wait和sleep的区别？对待锁的区别？


### 框架知识
1. SSM框架中，各个框架的作用是什么？
2. Spring默认是单例模式还是多例模式？各自的优缺点是什么？如何切换？
3. Spring的AOP你是如何理解的？
4. Spring如何通过AOP实现的全局事务管理
5. Spring配置事务管理都用到了哪些标签
6. IOC你是如何理解的？
7. Spring的IOC是如何实现的？
8. Spring创建bean时，必须有构造方法？若有多个构造方法，如何匹配具体某一个？
9. bean标签下的constructor-arg标签和property标签的区别？
10. ref属性和value属性的区别？value属性可以传递null么？
11. BeanFactory和FactoryBean的区别
12. @Resource注解和@Autowired注解有什么区别
13. @Component注解的作用
14. SpringMVC前台提交一个表单到数据库，中途都会经过哪些主要的类
15. SpringMVC的拦截器使用的什么设计模式
16. SpringMVC如何返回一个页面？
17. MyBatis取值#和$的区别

    #: 是占位符
    $: 是字符串拼接(会出现sql注入问题)
18. MyBatis有哪几种传参方式
19. MyBatis如何实现的分页查询
20. MyBatis如何实现的数据库字段和类属性的绑定


### 数据库
1. MySQL数据库如何实现的事务？
2. 常用的SQL优化
3. 哪些操作会破坏索引
4. 在使用utf8编码下，varchar(4)可以存储几个汉字？占用多少字节？
5. 在一个有id，name，score三个字段的表tb中，有数据(1,’b’,5),(2,’a’,3),(3,’d’,4),(4,’c’,1),(5,’e’,2)，执行sql：delete from tb order by score，问，删除的顺序是什么？
6. 写一个sql查询第二高分数？
7. MySQL的锁有哪几种
8. 乐观锁和悲观锁的区别
9. 主键和唯一键的区别
10. MySQL的事务隔离级别有哪些？每个级别下有可能导致什么问题？


### Linux
1. 如何挂起一个任务
2. unix socket和TCP哪个通信效率高？


### HTML
1.可以实现行内合并单元格的是？
2.谈谈前端使用bootstrap的优缺点
3.Velocity的使用场景是什么
4.一个表单提交响应速度很慢，如何防止重复提交？
5.JS闭包
6.JS变量的作用域？JS方法内的变量能否在外部访问？
7.Jquery的选择器有哪些？


### Redis
1. Redis支持哪些数据结构
2. Redis持久化的机制
3. Redis是单例还是多例的
4. Redis你们是如何使用的


### Mongodb
1.Mongodb非常吃内存，在生产环境是如何处理的？
2.Mongodb可否同时更改一条记录内一个集合中符合条件的多个值？


### 性能调优
1.JVM性能调优
2.Tomcat性能调优
3.MySQL性能调优
4.Mongodb性能调优


### 其他零零碎碎的
1.git和svn的区别
2.git从本地修改到推送到远程所需要经历的状态和命令
3.maven的打包命令
4.谈谈Jenkins在项目中的意义
5.使用过哪些消息队列























    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    ---
layout: post
title:  "Python语法"
date:   2017-08-10 15:12:38
categories: python
tags: Python
author: miaoqi
---
                                    
## 面试题

### 非技术
1. 为什么离职


### 业务
1. 在项目开发中遇到的难点有哪些？都是怎么解决的？
2. 前后端分离的优缺点


### 数据结构基础知识
1. 算法的复杂度
2. 双层for循环累加一个二维数组的算法复杂度
3. 一个递归算阶乘的算法复杂度
4. 写一个程序，判断字符串是否对称，并给出复杂度


### Java基础
1. List、Set、Map接口都有哪些实现？

    List:
        |-- ArrayList
        |-- LinkedList
        |-- Vector
    Set:
        |-- HashSet
        |-- TreeSet
    Map:
        |-- HashMap
        |-- LinkedHashMap
2. HashMap中负载因子的作用
3. ConcurrentHashMap是如何实现的线程安全？
4. StringBuffer 和 StringBuilder的区别
5. 接口和抽象类的异同

    接口: 所有方法都是抽象的
    抽象类: 可以有带实现的方法
6. JDK1.8新特性
7. 设计模式
8. 单例模式的使用场景
9. Servlet的生命周期
10. Servlet是否线程安全的？其类内有两个类变量，name和age，会不会有线程问题？如何解决？
11. JSP的九大内置对象
12. Session的生命周期
13. Session如何实现的保持会话
14. Session如何在浏览器端禁用cookie的情况下保持回话
15. synchronized关键字可以使用的场景：类？方法？变量？代码块？
16. 谈谈volatile关键字
17. 泛型的使用场景及优势
18. 对象中equals方法的作用
19. 重定向与请求转发的区别
20. newCachedThreadPool线程池何时回收闲置的线程？若长时间没有任务，所有的线程会被回收吗？
21. 线程的状态切换
22. wait和sleep的区别？对待锁的区别？


### 框架知识
1. SSM框架中，各个框架的作用是什么？
2. Spring默认是单例模式还是多例模式？各自的优缺点是什么？如何切换？
3. Spring的AOP你是如何理解的？
4. Spring如何通过AOP实现的全局事务管理
5. Spring配置事务管理都用到了哪些标签
6. IOC你是如何理解的？
7. Spring的IOC是如何实现的？
8. Spring创建bean时，必须有构造方法？若有多个构造方法，如何匹配具体某一个？
9. bean标签下的constructor-arg标签和property标签的区别？
10. ref属性和value属性的区别？value属性可以传递null么？
11. BeanFactory和FactoryBean的区别
12. @Resource注解和@Autowired注解有什么区别
13. @Component注解的作用
14. SpringMVC前台提交一个表单到数据库，中途都会经过哪些主要的类
15. SpringMVC的拦截器使用的什么设计模式
16. SpringMVC如何返回一个页面？
17. MyBatis取值#和$的区别

    #: 是占位符
    $: 是字符串拼接(会出现sql注入问题)
18. MyBatis有哪几种传参方式
19. MyBatis如何实现的分页查询
20. MyBatis如何实现的数据库字段和类属性的绑定


### 数据库
1. MySQL数据库如何实现的事务？
2. 常用的SQL优化
3. 哪些操作会破坏索引
4. 在使用utf8编码下，varchar(4)可以存储几个汉字？占用多少字节？
5. 在一个有id，name，score三个字段的表tb中，有数据(1,’b’,5),(2,’a’,3),(3,’d’,4),(4,’c’,1),(5,’e’,2)，执行sql：delete from tb order by score，问，删除的顺序是什么？
6. 写一个sql查询第二高分数？
7. MySQL的锁有哪几种
8. 乐观锁和悲观锁的区别
9. 主键和唯一键的区别
10. MySQL的事务隔离级别有哪些？每个级别下有可能导致什么问题？


### Linux
1. 如何挂起一个任务
2. unix socket和TCP哪个通信效率高？


### HTML
1.可以实现行内合并单元格的是？
2.谈谈前端使用bootstrap的优缺点
3.Velocity的使用场景是什么
4.一个表单提交响应速度很慢，如何防止重复提交？
5.JS闭包
6.JS变量的作用域？JS方法内的变量能否在外部访问？
7.Jquery的选择器有哪些？


### Redis
1. Redis支持哪些数据结构
2. Redis持久化的机制
3. Redis是单例还是多例的
4. Redis你们是如何使用的


### Mongodb
1.Mongodb非常吃内存，在生产环境是如何处理的？
2.Mongodb可否同时更改一条记录内一个集合中符合条件的多个值？


### 性能调优
1.JVM性能调优
2.Tomcat性能调优
3.MySQL性能调优
4.Mongodb性能调优


### 其他零零碎碎的
1.git和svn的区别
2.git从本地修改到推送到远程所需要经历的状态和命令
3.maven的打包命令
4.谈谈Jenkins在项目中的意义
5.使用过哪些消息队列























    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    