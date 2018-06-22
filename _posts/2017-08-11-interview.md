---
layout: post
title:  "面试题"
date:   2017-08-10 15:12:38
categories: Work
tags: Interview
author: miaoqi
---

* content
{:toc}
            

## 非技术
1. 为什么离职


## 业务
1. 在项目开发中遇到的难点有哪些？都是怎么解决的？
2. 前后端分离的优缺点


## 数据结构基础知识
1. 算法的复杂度
2. 双层for循环累加一个二维数组的算法复杂度
3. 一个递归算阶乘的算法复杂度
4. 写一个程序，判断字符串是否对称，并给出复杂度

1. Hash算法

    MD5, SHA, SHA-256, SHA-512


## JavaSE

### 包装类型

1. Integer类型中的缓存大小可以改吗? 可以改, 需要加虚拟机参数-Djava.lang.Integer.IntegerCache.high=250, Integer在加载的时候会首先获取这个参数的值

### 多线程

1. 多线程的实现方式

    * 继承Thread类, 重写run方法

    * 方式二: 实现Runnable接口, 作为参数构造Thread类

    * 方式三: 实现Callable接口, 相较于实现 Runnable 接口的方式, 方法可以有返回值, 并且可以抛出异常

    * 方式四: 线程池

1. 解决线程安全问题:

    * 方式一: 同步代码块

    * 方式二: 同步方法

    * 方式三: 1.5以后出现的同步锁(Lock)

1. 线程状态转换



### NIO

1. NIO与IO

    * 区别

    |IO  |NIO   |
    |----|------|
    |面向流|面向缓冲区|
    |阻塞IO|非阻塞IO|
    |无|选择器|

    * 通道(Channel)负责传输, 缓冲区(Buffer)负责存储数据, 双向传递

    * 可以操作直接缓冲区即内存, 内存消耗大, 不受控制

    * 通道是一个单独的处理器, 附属于CPU

### 集合

* List、Set、Map接口都有哪些实现？

    Collection
        |-- List: 多了对角标的操作
            |-- ArrayList: 底层是数组, 线程不安全, 查询快(contains利用了equals方法)
            |-- LinkedList: 底层是链表, 线程不安全, 增删快, 多了对头部和尾部的操作方法
            |-- Vector: 底层是数组, 线程安全, 查询快
        |-- Set:
            |-- HashSet: 底层是哈希表, 无序
                |-- LinkedHashSet: 底层是哈希表, 同时维护了一个链表, 存入和取出顺序一致
            |-- TreeSet: 底层是二叉树, 会进行排序
        Map:
        |-- HashMap: 允许空的键值对, 线程不安全(利用了hasCode和equals方法)(在1.7中是数组 + 链表, 在1.8中是数组 + 链表 + 红黑树)
            |-- LinkedHashMap
        |-- HashTable: 不允许空的键值对, 线程安全
        |-- TreeMap: 可以进行排序(实现Comparable方法, 或者传入Comparator实现类)
            
* HashMap中负载因子的作用

    * 当数组Map中的数组元素数量达到负载因子时, 会进行扩容

* StringBuffer 和 StringBuilder的区别?

    * StringBuffer: 线程安全, 效率低    
    
    * StringBuilder: 线程不安全, 效率高
        
* 接口和抽象类的异同

    * 接口: 所有方法都是抽象的(jdk1.8以后接口中可以有实现的方法)
    
    * 抽象类: 可以有带实现的方法
        
* JDK1.8新特性
* 设计模式
* 单例模式的使用场景

* 泛型的使用场景及优势

    * 泛型多用于集合中, 可以在程序编译时进行校验.

* 对象中equals方法的作用


### JUC

1. 锁类型

    可重入锁：在执行对象中所有同步方法不用再次获得锁

    可中断锁：在等待获取锁过程中可中断
    
    公平锁： 按等待获取锁的线程的等待时间进行获取，等待时间长的具有优先获取锁权利
    
    读写锁：对资源读取和写入的时候拆分为2部分处理，读的时候可以多线程一起读，写的时候必须同步地写

1. synchronized和Lock

|类别|synchronized|Lock|
|---------|-------------------|---------|
|存在层次|Java的关键字，在jvm层面上|是一个类|
|锁的释放|1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁|在finally中必须释放锁，不然容易造成线程死锁|
|锁的获取|假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待|分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待|
|锁状态|无法判断|可以判断|
|锁类型|可重入 不可中断 非公平|可重入 可判断 可公平（两者皆可）
|性能|少量同步|大量同步|

1. 谈谈volatile关键字

    > 内存可见性（Memory Visibility）是指当某个线程正在使用对象状态而另一个线程在同时修改该状态，需要确保当一个线程修改了对象状态后，其他线程能够看到发生的状态变化。volatile就是解决内存可见性问题的, 但是不保证变量的原子性, 不具备互斥性, 相较于synchronized是一种较为轻量级的同步策略

1. ConcurrentHashMap是如何实现的线程安全？

    > 内部采用锁分段机制代替了HashTable的独占锁, 进而性能提高. 默认有16个段(Segment), 每个段内维护了一个数组, 数组维护链表, 1.8以后改为了CAS机制

1. newCachedThreadPool线程池何时回收闲置的线程？若长时间没有任务，所有的线程会被回收吗？
1. 线程的状态切换
1. wait和sleep的区别？对待锁的区别？

    * 锁的区别:    
    
        wait会释放锁, 当前线程进入休眠状态, 等待其他线程唤醒     
    
        sleep不会释放锁, 当前线程进入休眠状态, 休眠时间结束后自己醒来, 可以使用interrupt()强行打断
        
    * 作用范围:    
    
        wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用。

1. CAS算法:

    * 内存值(V), 旧的预期值(A), 如果V == A代表没人修改过, 可以将更新值(B)赋值给(V)

    * ABA问题: 

        比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。

1. 常见的锁类:

    * ReentrantLock: 重入锁

    * ReadWriteLock: 读写锁, 写写/读写 需要“互斥”

    * CountDownLatch: 闭锁, 内部维护了一个计数器, 只有计数器为0的时候才会往下执行, 作为参数传到线程中取, 每个线程运行完了减少计数器

    * CopyOnWriteArrayList/CopyOnWriteArraySet: 写入并复制, 每次写入时, 都会复制一个新的列表

## JavaEE

* Servlet的生命周期

    第一次访问Servlet时创建实例, 调用init方法, 调用service方法, 之后的每次访问调用service方法, 服务器关闭时调用destory方法, 是单例的.
    
* Servlet是否线程安全的？其类内有两个类变量，name和age，会不会有线程问题？如何解决？

    线程不是安全的, 会有多线程并发问题, 实现SingleThreadModel
        
* Filter的生命周期

    web应用启动时创建filter对象调用init方法, 之后会调用doFilter方法, 服务器停止时调用destory方法, 是单例的.
        
* JSP的九大内置对象

    page: 当前servlet对象
    
    request: 请求对象
    
    response: 响应对象
    
    session: 会话对象
    
    pageContext: 获取其他域对象
    
    application: 应用程序(ServletContext)
    
    out: 输出对象
    
    config: 配置(ServletConfig)
   
    exception: 页面中开启异常才有该对象

* Session的生命周期

    request.getSession()时第一次创建Session对象，开始计时如在20分钟内没有访问session，那么session生命周期被销毁, 如果在20分钟内（如在第19分钟时）访问过session，那么，将重新计算session的生命周期
        
    > 1. 关闭tomcat

    > 2. reload web应用
        
    > 3. session时间到
        
    > 4. 手动调用invalidate方法

* Session如何实现的保持会话

    > Session是基于Cookie的, 在Cookie中保存JSESSIONID, 每次访问根据该值从服务器端获取同一个Session对象

* Session如何在浏览器端禁用cookie的情况下保持回话

    > 将用到Session的url全部进行url重写, 会在url后追加一个JSESSIONID参数

* synchronized关键字可以使用的场景：类？方法？变量？代码块？

    > 方法, 代码块

* 重定向与请求转发的区别

    > 重定向是2次请求, 请求参数会刷新
    
    > 转发是1次请求, 参数会保留


## 框架知识

### Solr

1. Document -> Field域 -> Term -> 建立索引 -> IKAnalyzer -> 维护了一个词库

2. 倒排索引:

### Hibernate

1. 三种状态

    > 瞬态: 没有OID, 没有被session管理
    
    > 持久态: 有OID, 被session管理
    
    > 游离态: 有OID, 没有被session管理


1. SSM框架中，各个框架的作用是什么？

### Spring

1. 在一个Service方法中调用另外一个方法, 会有事物吗?

    * 没有事物, 因为在controller中注入的是代理对象, 而在目标对象内部调用的方法是原始的方法, 没有加aop, 所以没有事物, 

    * 解决办法: 开启暴露AOP代理到ThreadLocal支持, 修改业务代码this.b();-----------修改为--------->((AService) AopContext.currentProxy()).b();

2. Spring默认是单例模式还是多例模式？各自的优缺点是什么？如何切换？

    * 默认是单例的, 通过prototype属性
    
3. Spring的AOP你是如何理解的？
4. Spring如何通过AOP实现的全局事务管理
5. Spring配置事务管理都用到了哪些标签
6. IOC你是如何理解的？

    将对象的创建交给Spring来管理

7. Spring的IOC是如何实现的？
8. Spring创建bean时，必须有构造方法？若有多个构造方法，如何匹配具体某一个？
9. bean标签下的constructor-arg标签和property标签的区别？
10. ref属性和value属性的区别？value属性可以传递null么？



11. BeanFactory和FactoryBean的区别
12. @Resource注解和@Autowired注解有什么区别

    @Autowired
    默认按照类型装配, 如果需要名称装配, 需要使用@Qualifier注解
    
    @Resource
    默认按照名称装配, 如果名称不匹配会按照类型装配, 如果指定了name属性, 那么只会按照名称去装配

13. @Component注解的作用
14. SpringMVC前台提交一个表单到数据库，中途都会经过哪些主要的类
15. SpringMVC的拦截器使用的什么设计模式
16. SpringMVC如何返回一个页面？
17. bean的作用域

    * singleton（单例）
    * prototype（原型）
    * request
    * session
    * global session

### SpringCloud

* 是多种技术的集合

#### Eureka和Zookeeper

* Eureka遵守AP

    * Eureka各个节点是平等的, 几个节点挂掉不会影响正常的工作, 剩余的节点依然可以提供注册和查询服务. Eureka的客户端再向某个Eureka注册时如果发现连接失败, 则会自动切换至其他节点, 只要有一台Eureka还在, 就可以保证服务可用, 只不过查询到的信息可能不是最新的. Eureka还用一种自我保护机制, 如果在15分钟内超过85%的节点没有正常心跳, 那么Eureka就认为客户端与注册中心出现网络故障, 会出现以下几种情况:

        1. Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务

        2. Eureka仍然能够接受新服务注册和查询请求, 但是不会同步到其他节点上

        3. 当网络稳定时, 当前实例新的注册信息会同步到其他节点中

* Zookeeper遵守CP

    * Zookeeper当master节点因为网络故障与其他节点失去联系时, 剩余节点会重新进行leader选举, 选举leader的时间为30~120s, 且选举期间整个Zookeeper集群是不可用的, 这就导致服务瘫痪了

#### Ribbon负载均衡(面向服务)

* 基于Netflix Ribbon实现的一套客户端  负载均衡的工具, Ribbon + RestTemplate

#### Feign负载均衡(面向接口)

* 通过接口 + 注解获取服务地址

* 只需要创建一个接口, 在上边添加注解即可

#### Hystrix断路器

* 服务熔断(服务端)

    * Hystrix是一个用于处理分布式系统的延迟和容错的开源库, Hystrix能够保证在一个依赖出问题的情况下, 不会导致整个服务失败, 避免级联故障, 以提高分布式系统的弹性
    
    * 断路器本身是一种开关装置, 当某个服务单元发生故障之后, 通过断路器的故障监控(类似熔断保险丝), 向调用方返回一个符合预期的, 可处理的备选响应(FallBack), 而不是长时间等待或者抛出调用无法处理的异常

* 服务降级(客户端)

    * 整体资源不够了, 先关闭一些服务, 待资源充足了, 再将服务打开

#### Zuul路由网关

* 代理

* 路由

* 过滤

### Config分布式配置中心

* 集中管理配置文件

* 不同环境不同配置, 动态化配置更新, 分环境部署比如dev/test/prod

* 运行期间动态调整配置, 不需要再每个服务部署的机器上编写配置文件, 服务会向配置中心统一拉取配置自己的信息

* 当配置发生变动时, 服务不需要重启即可感知到配置的变化并应用新的配置

* 将配置中心以REST接口的形式暴露



### MyBatis

17. MyBatis取值#和$的区别

    >  #: 是占位符

    >  $: 是字符串拼接(会出现sql注入问题)
        
18. MyBatis有哪几种传参方式
19. MyBatis如何实现的分页查询
20. MyBatis如何实现的数据库字段和类属性的绑定

    > 使用jdbcType标签


## 数据库
    
    
3. 哪些操作会破坏索引
4. 在使用utf8编码下，varchar(4)可以存储几个汉字？占用多少字节？
5. 在一个有id，name，score三个字段的表tb中，有数据(1,’b’,5),(2,’a’,3),(3,’d’,4),(4,’c’,1),(5,’e’,2)，执行sql：delete from tb order by score，问，删除的顺序是什么？
6. 写一个sql查询第二高分数？
7. MySQL的锁有哪几种
8. 乐观锁和悲观锁的区别
9. 主键和唯一键的区别


## FastDFS

1. 开源分布式文件系统

    > 冗余备份(高可用)
    
    > 负载均衡(集群, 高性能)
    
    > 横向扩展

2. 三个角色(模仿dubbo)

    > Client(Consumer) 上传下载数据的服务器，也就是我们自己的项目所部署在的服务器。

    > Storage(Provider) 存储服务器，主要提供容量和备份服务；以 group 为单位，每个 group 内可以有多台 storage server，数据互为备份。
    
    > Tracker(Registry) 跟踪服务器，主要做调度工作，起到均衡的作用；负责管理所有的 storage server和 group，每个 storage 在启动后会连接 Tracker，告知自己所属 group 等信息，并保持周期性心跳。
    
## dubbo(SOA解决方案)

1. 三个角色

    > Consumer
    
    > Provider
    
    > Registry
    
2. 流程

    > 消费者向注册中心要服务, 如果要不到就重复尝试, 提供者向注册中心注册服务, 消费者要到服务后向提供者发起http请求

3. 注册中心挂掉

    * 如果注册中心宕机了, 还是可以请求Provider, 因为Consumer缓存了请求地址


## Nginx

1. 反向代理服务器, 图片服务器, 静态资源代理

2. 程序员只关心server{}


## Mongodb

1. Mongodb非常吃内存，在生产环境是如何处理的？
2. Mongodb可否同时更改一条记录内一个集合中符合条件的多个值？


## 性能调优

1. JVM性能调优
2. Tomcat性能调优
3. MySQL性能调优
4. Mongodb性能调优

## 架构

1. SEO(搜索引擎优化)

    > 静态页面(html)比动态页面(jsp, asp, php)分数要高

2. RPC和消息队列:

    * 系统结构

        > RPC系统结构: Consumer <=> Provider

        > MessageQueue系统结构: Sender <=> Queue <=> Receiver

    * 功能特点

        * 消息的特点:

            > Message Queue把请求的压力保存一下，逐渐释放出来，让处理者按照自己的节奏来处理。
    
            > Message Queue引入一下新的结点，让系统的可靠性会受Message Queue结点的影响。
        
            > Message Queue是异步单向的消息。发送消息设计成是不需要等待消息处理的完成。

        * RPC的特点:

            > 同步调用，对于要等待返回结果/处理结果的场景，RPC是可以非常自然直觉的使用方式。

            > 由于等待结果，Consumer（Client）会有线程消耗。

        * 适用场合说明

            > 希望同步得到结果的场合，RPC合适。

            > 希望使用简单，则RPC；RPC操作基于接口，使用简单，使用方式模拟本地调用。异步的方式编程比较复杂。

            > 不希望发送端（RPC Consumer、Message Sender）受限于处理端（RPC Provider、Message Receiver）的速度时，使用Message Queue。

3. 蓝绿部署(Blue Green Deployment)和滚动部署(Rolling update)

    * 蓝绿部署:

        > 维护两个集群, 上线时将A集群移除, 更新A集群, 将A集群加回负载均衡列表中, 移除B集群, 更新B集群, 再将B集群加回负载均衡列表中, 成本高
    
    * 滚动部署:

        > 滚动部署只需要一个集群, 集群下的不同节点可以独立进行版本升级, 比如16节点的集群中, 每次选择4个节点升级, 不好定位问题

4. CAP定理

    * C(Consistency): 强一致性

    * A(Availability): 高可用性

    * P(Partition tolerance): 分区容错性
    
    * 在一个分布式系统中, 在出现节点之间无法通信, 你只能选择可用性或者一致性, 没法同时选择他们, 即只能满足CP或者AP不能满足CA, 如果变成了CA就成了传统架构


## 消息队列

1. ActiveMQ:

    * 后台地址: 

    * 服务端口: 61616

    * 两种模式

        1. 点多点模式(一对一)

        1. 发布订阅模式(一对多)

    * 消息消费不过来怎么办?

        1. 分析日志, 分析原因     
        
            从生产者考虑:       
                消息有没有乱发, 消息堆积       
            
            从消费者考虑:       
                消费者是否能正常消费消息, 增加消费者, 利用点对点+手动应答(能者多劳模式)

## 分布式:

* 分布式锁:

    对于单进程可以使用语言提供的锁, 对于分布式场景可以采用分布式锁

    1. Memcached分布式锁

        利用Memcached的add命令。此命令是原子性操作，只有在key不存在的情况下，才能add成功，也就意味着线程得到了锁。
    
    
    
    2. Redis分布式锁
    
        和Memcached的方式类似，利用Redis的setnx命令。此命令同样是原子性操作，只有在key不存在的情况下，才能set成功。（setnx命令并不完善，后续会介绍替代方案）
    
        1. 加锁: 

            最简单的方法是使用setnx命令。key是锁的唯一标识，按业务来决定命名。比如想要给一种商品的秒杀活动加锁，可以给key命名为 “lock_sale_商品ID” 。而value设置成什么呢？我们可以姑且设置成1。加锁的伪代码如下：    

            setnx（key，1）

            当一个线程执行setnx返回1，说明key原本不存在，该线程成功得到了锁；当一个线程执行setnx返回0，说明key已经存在，该线程抢锁失败。

        2. 解锁:

            有加锁就得有解锁。当得到锁的线程执行完任务，需要释放锁，以便其他线程可以进入。释放锁的最简单方式是执行del指令，伪代码如下：

            del（key）

            释放锁之后，其他线程就可以继续执行setnx命令来获得锁

        3. 锁超时:

            锁超时是什么意思呢？如果一个得到锁的线程在执行任务的过程中挂掉，来不及显式地释放锁，这块资源将会永远被锁住，别的线程再也别想进来。所以，setnx的key必须设置一个超时时间，以保证即使没有被显式释放，这把锁也要在一定时间后自动释放。setnx不支持超时参数，所以需要额外的指令，伪代码如下：

            expire（key， 30）

            综合起来，我们分布式锁实现的第一版伪代码如下：

                if（setnx（key，1） == 1）{
            
                    expire（key，30）
            
                    try {
            
                        do something ......
            
                    } finally {
            
                        del（key）
    
                    }
            
                }

        4. 第一版存在的问题:

            1.  setnx和expire的非原子性, 当A线程得到了锁, 还未来得及设置超时时间就挂掉了, 会导致锁没有超时时间, 在Redis2.6.12以上版本增加了set（key，1，30，NX）取代了setnx

            2. del 导致误删, 假如A线程得到了锁, 并且设置30秒超时, 如果某些原因导致了A超过了30秒, 自动释放锁, B线程获得了锁, 当A运行完成后, 删除了锁, 直接删除的是B线程的锁, 可以再del之前判断一下这个锁是不是自己的, 即将value设置成线程id, 每次删除前判断一下是不是自己的锁, 这样实际上有并发问题

            3. 还是刚才第二点所描述的场景，虽然我们避免了线程A误删掉key的情况，但是同一时间有A，B两个线程在访问代码块，仍然是不完美的。怎么办呢？我们可以让获得锁的线程开启一个守护线程，用来给快要过期的锁“续航”。这样就避免了A线程执行时间过长, 导致锁自动释放的问题, 假设锁是30秒, 让守护线程29秒时给锁续命20秒, 当A线程销毁时, 守护线程也就销毁了
    
    
    3. Zookeeper分布式锁
    
        利用Zookeeper的顺序临时节点，来实现分布式锁和等待队列。Zookeeper设计的初衷，就是为了实现分布式锁服务的。
    
    
    
    4. Chubby
    
        Google公司实现的粗粒度分布式锁服务，底层利用了Paxos一致性算法。

* 分布式事务:

    1. XA二段提交

    1. XA三段提交
    
    1. MQ事务

    1. TCC事务

## HTTP协议:



## JVM

> https://docs.oracle.com/javase/8/docs/

> JDK: Java Development Kit

> JRE: Java Runtime Enviroment

> JVM: Java Virtual Machine

* 内存溢出

    > 增加jvm启动参数, 生成内存快照, 使用MemoryAnalyzer分析插件分析内存快照, 定位发生问题的位置

    > jvm监控工具: jdk安装目录/bin/jconsole
    



    
    
    
    
    
    
    
    
    
    