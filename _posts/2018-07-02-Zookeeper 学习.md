---
layout: post
title: "Zookeeper 学习"
categories: [BigData]
description:
keywords:
---

* content
{:toc}

## 简介

Zookeeper 是一个**分布式协调服务**; 就是为用户的分布式应用程序提供协调服务

Zookeeper 是为别的分布式程序服务的

Zookeeper 本身就是一个分布式程序(只要有半数以上节点存活, zk就能正常服务)

Zookeeper 所提供的服务涵盖: 主从协调、服务器节点动态上下线、统一配置管理、分布式共享锁、统一名称服务……

虽然说可以提供各种服务, 但是 zookeeper 在底层其实只提供了两个功能

1. 管理(存储, 读取)用户程序提交的数据
2. 为用户程序提供数据节点监听服务

Zookeeper从设计模式角度来理解: 是一个基于观察者模式设计的分布式服务管理框架, 它负责存储和管理大家都关心的数据, 然后接受观察者的注册, 一旦这些数据的状态发生变化, Zookeeper 就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应



## 特点

1. Zookeeper: 一个领导者（Leader）, 多个跟随者（Follower）组成的集群. 

2. 集群中只要有半数以上节点存活, Zookeeper集群就能正常服务. 所以Zookeeper适合安装奇数台服务器. 

3. 全局数据一致: 每个Server保存一份相同的数据副本, Client无论连接到哪个Server, 数据都是一致的. 

4. 更新请求顺序执行, 来自同一个Client的更新请求按其发送顺序依次执行. 

5. 数据更新原子性, 一次数据更新要么成功, 要么失败. 

6. 实时性, 在一定时间范围内, Client能读到最新数据







## Zookeeper 的数据模型

ZooKeeper 数据模型的结构与Unix 文件系统很类似, 整体上可以看作是一棵树, 每个节点称做一个ZNode. 每一个ZNode 默认能够存储1MB 的数据, 每个ZNode 都可以通过其路径唯一标识. 

<img src="http://www.milky.show/images/bigdata/zookeeper/6.png" alt="http://www.milky.show/images/bigdata/zookeeper/6.png" style="zoom:50%;" />

1. 层次化的目录结构, 命名符合常规文件系统规范, 它很像数据结构当中的树, 也很像文件系统的目录
1. 每个节点在 Zookeeper 中叫做 **Znode**, 并且其有一个唯一的路径标识
1. 节点 Znode 可以包含数据和子节点
1. 客户端应用可以在节点上设置监视器

<img src="http://www.milky.show/images/bigdata/zookeeper/1.png" alt="http://www.milky.show/images/bigdata/zookeeper/1.png" style="zoom:50%;" />

### Znode

Znode 包含了数据, 子节点引用, 访问权限等等

<img src="http://www.milky.show/images/bigdata/zookeeper/2.png" alt="http://www.milky.show/images/bigdata/zookeeper/2.png" style="zoom:50%;" />

**data:** Znode存储的数据信息. 

**ACL:** 记录Znode的访问权限, 即哪些人或哪些IP可以访问本节点. 

**stat:** 包含Znode的各种元数据, 比如事务ID、版本号、时间戳、大小等等. 

**child:** 当前节点的子节点引用, 类似于二叉树的左孩子右孩子. 



这里需要注意一点, Zookeeper是为读多写少的场景所设计. Znode并不是用来存储大规模业务数据, 而是用于存储少量的状态和配置信息, **每个节点的数据最大不能超过1MB**. 



### ZNode 节点信息

**查看当前znode中所包含的内容**

```bash
[zk: hadoop102 :2181(CONNECTED) 0] ls /
[zookeeper]
```

查看当前节点详细数据 

```
[zk: hadoop102 :2181(CONNECTED) 5] ls s /
[zookeeper]cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x0
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

czxid: 创建节点的事务zxid 

每次修改ZooKeeper状态都会产生一个ZooKeeper事务ID. 事务ID是ZooKeeper中所有修改总的次序. 每次修改都有唯一的zxid, 如果zxid1小于zxid2, 那么zxid1在zxid2之前发生. 

ctime: znode被创建的毫秒数（从1970年开始）

mzxid: znode最后更新的事务zxid 

mtime: znode最后修改的毫秒数（从1970年开始）

pZxid: znode最后更新的子节点zxid 

cversion: znode 子节点变化号, znode 子节点修改次数

dataversion: znode 数据变化号

aclVersion: znode 访问控制列表的变化号

ephemeralOwner: 如果是临时节点, 这个是znode 拥有者的 session id. 如果不是临时节点则是0. 

dataLength: znode 的数据长度

numChildren: znode 子节点数量

### 节点类型（持久**/**短暂**/**有序号**/**无序号）

Znode分为四种类型: 

* 持久节点(PERSISTENT)默认

    默认的节点类型, 创建节点的客户端与zookeeper断开连接后, 该节点依旧存在 

* 持久节点顺序节点(PERSISTENT_SEQUENTIAL)

    所谓顺序节点, 就是在创建节点时, Zookeeper根据创建的时间顺序给该节点名称进行编号

* 临时节点(EPHEMERAL)

    和持久节点相反, 当创建节点的客户端与zookeeper断开连接后, 临时节点会被删除: 

* 临时顺序节点(EPHEMERAL_SEQUENTIAL)

    顾名思义, 临时顺序节点结合和临时节点和顺序节点的特点: 在创建节点时, Zookeeper根据创建的时间顺序给该节点名称进行编号, 当创建节点的客户端与zookeeper断开连接后, 临时节点会被删除

说明: 创建znode时设置顺序标识, znode名称后会附加一个值, 顺序号是一个单调递增的计数器, 由父节点维护注意: 在分布式系统中, 顺序号可以被用于为所有的事件进行全局排序, 这样客户端可以通过顺序号推断事件的顺序

## 应用场景

提供的服务包括: 统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等. 

### 统一命名服务

在分布式环境下, 经常需要对应用/服务进行统一命名, 便于识别. 例如: IP不容易记住, 而域名容易记住.  

### 统一配置管理

分布式环境下, 配置文件同步非常常见. 

1.  一般要求一个集群中, 所有节点的配置信息是一致的, 比如Kafka 集群
2.  对配置文件修改后, 希望能够快速同步到各个节点上. 

配置管理可交由ZooKeeper实现

1.  可将配置信息写入ZooKeeper上的一个Znode. 
2.  各个客户端服务器监听这个Znode. 
3.  一旦Znode中的数据被修改, ZooKeeper将通知各个客户端服务器

### 统一集群管理

分布式环境中, 实时掌握每个节点的状态是必要的. 

*   可根据节点实时状态做出一些调整

ZooKeeper可以实现实时监控节点状态变化

*   可将节点信息写入ZooKeeper上的一个ZNode. 
*   监听这个ZNode可获取它的实时状态变化.  

### 服务器动态上下线

客户端能实时洞察到服务器上下线的变化

### 软负载均衡

在Zookeeper中记录每台服务器的访问数, 让访问数最少的服务器去处理最新的客户端请求 





## 安装

### 单机版安装

1. 创建 data 目录存放 ZooKeeper 数据

1. conf 下编写 zoo.cfg 指定 data 目录

    ```bash
    # The number of milliseconds of each tick
    # 心跳周期毫秒
    tickTime=2000
    # The number of ticks that the initial
    # synchronization phase can take
    # 初始化心跳个数
    initLimit=10
    # The number of ticks that can pass between
    # sending a request and getting an acknowledgement
    # 发送请求到响应的心跳个数
    syncLimit=5
    # the directory where the snapshot is stored.
    # do not use /tmp for storage, /tmp here is just
    # example sakes.
    # 数据目录
    dataDir=/Users/miaoqi/Documents/zookeeper-3.4.6/data/
    dataLogDir=/Users/miaoqi/Documents/zookeeper-3.4.6/dataLog/
    # the port at which the clients will connect
    # 客户端端口
    clientPort=2181
    # the maximum number of client connections.
    # increase this if you need to handle more clients
    #maxClientCnxns=60
    #
    # Be sure to read the maintenance section of the
    # administrator guide before turning on autopurge.
    #
    # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
    #
    # The number of snapshots to retain in dataDir
    #autopurge.snapRetainCount=3
    # Purge task interval in hours
    # Set to "0" to disable auto purge feature
    #autopurge.purgeInterval=1
    ```

3.  启动 zk, 使用 status 查看状态

    `bin/zkServer.sh start`

    `bin/zkServer.sh status`

    `bin/zkServer.sh stop`

    `bin/zkCli.sh`

**配置参数解读**

**tickTime = 2000**: 通信心跳时间, **Zookeeper**服务器与客户端心跳时间, 单位毫秒 

<img src="http://www.milky.show/images/bigdata/zookeeper/7.png" alt="http://www.milky.show/images/bigdata/zookeeper/7.png" style="zoom:50%;" />

**initLimit = 10**: **LF**初始通信时限 

Leader和Follower初始连接时能容忍的最多心跳数（tickTime的数量） 

**syncLimit = 5**: **LF**同步通信时限 

Leader和Follower之间通信时间如果超过syncLimit * tickTime, Leader认为Follwer死掉, 从服务器列表中删除Follwer.  

**dataDir**: 保存Zookeeper中的数据 

注意: 默认的tmp目录, 容易被Linux系统定期删除, 所以一般不用默认的tmp目录.  

**clientPort = 2181**: 客户端连接端口, 通常不做修改.  



### 集群安装

1. 修改每一个 zookeeper 的配置文件, 增加如下配置

    ```bash
    # 2881 是leader和follower通信的端口 默认是2888
    # 3881 是投票的端口 默认是3888
    server.1=127.0.0.1:2881:3881
    server.2=127.0.0.1:2882:3882
    server.3=127.0.0.1:2883:3883
    ```

1. 到数据目录下创建文件 myid, 文件内容就是 myid 的值

1. 分别启动三台机器, 使用 status 查看状态

**配置参数解读**

`server.A=B:C:D`

**A** 是一个数字, 表示这个是第几号服务器；

集群模式下配置一个文件myid, 这个文件在dataDir目录下, 这个文件里面有一个数据就是A的值, Zookeeper启动时读取此文件, 拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server. 

**B** 是这个服务器的地址；

**C **是这个服务器Follower与集群中的Leader服务器交换信息的端口；

**D** 是万一集群中的Leader服务器挂了, 需要一个端口来重新进行选举, 选出一个新的Leader, 而这个端口就是用来执行选举时服务器相互通信的端口. 



## 选举机制

SID: 服务器ID. 用来唯一标识一台ZooKeeper集群中的机器, 每台机器不能重复, 和myid一致. 

ZXID: 事务ID. ZXID是一个事务ID, 用来标识一次服务器状态的变更. 在某一时刻,  集群中的每台机器的ZXID值不一定完全一致, 这和ZooKeeper服务器对于客户端“更新请求”的处理逻辑有关. 

Epoch: 每个Leader任期的代号. 没有Leader时同一轮投票过程中的逻辑时钟值是相同的. 每投完一次票这个数据就会增加 

### Zookeeper选举机制-第一次启动

<img src="http://www.milky.show/images/bigdata/zookeeper/8.png" alt="http://www.milky.show/images/bigdata/zookeeper/8.png" style="zoom:50%;" />

1.  服务器1启动, 发起一次选举. 服务器1投自己一票. 此时服务器1票数一票, 不够半数以上（3票）, 选举无法完成, 服务器1状态保持为LOOKING； 2.
2.  服务器2启动, 再发起一次选举. 服务器1和2分别投自己一票并交换选票信息: 此时服务器1发现服务器2的myid比自己目前投票推举的（服务器1） 大, 更改选票为推举服务器2. 此时服务器1票数0票, 服务器2票数2票, 没有半数以上结果, 选举无法完成, 服务器1, 2状态保持LOOKING 
3.  服务器3启动, 发起一次选举. 此时服务器1和2都会更改选票为服务器3. 此次投票结果: 服务器1为0票, 服务器2为0票, 服务器3为3票. 此时服务器3的票数已经超过半数, 服务器3当选Leader. 服务器1, 2更改状态为FOLLOWING, 服务器3更改状态为LEADING； LOOKING LOOKING 1 1 2 0 3 0 
4.  服务器4启动, 发起一次选举. 此时服务器1, 2, 3已经不是LOOKING状态, 不会更改选票信息. 交换选票信息结果: 服务器3为3票, 服务器4为1票. 此时服务器4服从多数, 更改选票信息为服务器3, 并更改状态为FOLLOWING； 
5.  服务器5启动, 同4一样当小弟. 

### Zookeeper选举机制-非第一次启动

<img src="http://www.milky.show/images/bigdata/zookeeper/9.png" alt="http://www.milky.show/images/bigdata/zookeeper/9.png" style="zoom:50%;" />

1.  当ZooKeeper集群中的一台服务器出现以下两种情况之一时, 就会开始进入Leader选举
    *   服务器初始化启动. 
    *   服务器运行期间无法和Leader保持连接. 

2.  而当一台机器进入Leader选举流程时, 当前集群也可能会处于以下两种状态:  

    *   集群中本来就已经存在一个Leader. 对于第一种已经存在Leader的情况, 机器试图去选举Leader时, 会被告知当前服务器的Leader信息, 对于该机器来说, 仅仅需要和Leader机器建立连接, 并进行状态同步即可. 

    * 集群中确实不存在Leader. 假设ZooKeeper由5台服务器组成, SID分别为1、2、3、4、5, ZXID分别为8、8、8、7、7, 并且此时SID为3的服务器是Leader. 某一时刻,  3和5服务器出现故障, 因此开始进行Leader选举.  

        SID为1、2、4的机器投票情况（EPOCH, ZXID, SID ）:  （1, 8, 1） （1, 8, 2） （1, 7, 4）选举Leader规则:  

        1.  EPOCH 大的直接胜出
        2.  EPOCH 相同, 事务 id 大的胜出
        3.  事务 id 相同, 服务器 id 大的胜出 



## 服务端命令

| 命令语法           | 功能描述       |
| ------------------ | -------------- |
| zkServer.sh start  | 启动服务端     |
| zkServer.sh status | 查看服务端状态 |
| zkServer.sh stop   | 停止服务端     |



## 客户端命令

| 命令基本语法                   | 功能描述                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| help                           | 显示所有操作命令                                             |
| ls path [watch]                | 使用ls 命令来查看当前znode的子节点[可监听]<br />-w 监听子节点变化<br />-s 附加次级信息 |
| create [-s] [-e] path data acl | 在某个节点下创建节点, 默认是持久节点<br />-s 是顺序节点, 会自动给节点加上序号<br />-e 是短暂节点, 客户端断开连接节点销毁 |
| get path [watch]               | 获得节点的值[可监听]<br />-w 监听节点内容变化<br />-s 附加次级信息 |
| set                            | 设置节点的具体值                                             |
| stat                           | 查看节点状态                                                 |
| delete                         | 删除节点                                                     |
| deleteall                      | 递归删除节点                                                 |
| zkCli.sh -server ip:port       | 客户端连接服务器端                                           |
| quit                           | 客户端断开连接                                               |
| getChildren                    | 获取节点下的所有子节点                                       |
| exists                         | 判断节点是否存在                                             |

这其中, exists, getData, getChildren属于读操作. Zookeeper客户端在请求读操作的时候, 可以选择是否设置**Watch**. 

我们可以理解成是注册在特定Znode上的触发器. 当这个Znode发生改变, 也就是调用了create, delete, setData方法的时候, 将会触发Znode上注册的对应事件, 请求Watch的客户端会接收到**异步通知**. 







## Zookeeper 监听器原理(Watch)

当客户端设置Watch时, 会开启两个线程, 一个线程负责监听器, 一个线程负责和Zookeeper服务端通信告诉服务端监听线程的端口, 当服务端变化时, 服务端主动调用客户端监听器端口, 监听器调用zkClient的watch方法实现监听器, 原理是各种rpc调用

客户端注册监听它关心的目录节点, 当目录节点发生变化（数据改变、节点删除、子目录节点增加删除）时, ZooKeeper 会通知客户端. 监听机制保证ZooKeeper 保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序. 

1. 客户端调用getData方法, watch参数是true. 服务端接到请求, 返回节点数据, 并且在对应的哈希表里插入被Watch的Znode路径, 以及Watcher列表. 

    <img src="http://www.milky.show/images/bigdata/zookeeper/3.png" alt="http://www.milky.show/images/bigdata/zookeeper/3.png" style="zoom: 50%;" />

2. 当被Watch的Znode已删除, 服务端会查找哈希表, 找到该Znode对应的所有Watcher, 异步通知客户端, 并且删除哈希表中对应的Key-Value. 

    <img src="http://www.milky.show/images/bigdata/zookeeper/4.png" alt="http://www.milky.show/images/bigdata/zookeeper/4.png" style="zoom:50%;" />

**监听原理详解**

首先要有一个 main() 线程

在 main 线程中创建 Zookeeper 客户端, 这时就会创建两个线程, 一个负责网络连接通信（connet）, 一个负责监听（listener）. 

通过connect线程将注册的监听事件发送给Zookeeper. 

在 Zookeeper 的注册监听器列表中将注册的监听事件添加到列表中. 

Zookeeper 监听到有数据或路径变化, 就会将这个消息发送给listener线程. 

listener 线程内部调用了 process() 方法. 

**常见的监听**

监听节点数据的变化

`get path [watch] `

监听子节点增减的变化

`ls path [watch] `



## Zookeeper 的一致性

Zookepper 身为分布式系统协调服务, 如果自身挂掉了怎么办呢? 为了防止单机挂掉的情况, Zookeeper 维护了一个集群

<img src="http://www.milky.show/images/bigdata/zookeeper/5.png" alt="http://www.milky.show/images/bigdata/zookeeper/5.png" style="zoom: 50%;" />

Zookeeper Service 集群是一主多从结构

1. 在更新数据时, 首先更新到主节点(这里的节点是指服务器, 不是Znode), 再同步到从节点

2. 在读取数据时, 直接读取任意从节点. 

为了保证主从节点的数据一致性, Zookeeper采用了**ZAB协议**, 这种协议非常类似于一致性算法**Paxos**和**Raft**. 

## Zookeeper 分布式锁

1. 获取锁

    首先, 在Zookeeper当中创建一个持久节点ParentLock. 当第一个客户端想要获得锁时, 需要在ParentLock这个节点下面创建一个临时顺序节点 Lock1. 

    之后, Client1查找ParentLock下面所有的临时顺序节点并排序, 判断自己所创建的节点Lock1是不是顺序最靠前的一个. 如果是第一个节点, 则成功获得锁. 

    这时候, 如果再有一个客户端 Client2 前来获取锁, 则在ParentLock下载再创建一个临时顺序节点Lock2. 

    Client2查找ParentLock下面所有的临时顺序节点并排序, 判断自己所创建的节点Lock2是不是顺序最靠前的一个, 结果发现节点Lock2并不是最小的. 

    于是, Client2向排序仅比它靠前的节点Lock1注册Watcher, 用于监听Lock1节点是否存在. 这意味着Client2抢锁失败, 进入了等待状态. 

    这时候, 如果又有一个客户端Client3前来获取锁, 则在ParentLock下载再创建一个临时顺序节点Lock3. 

    Client3查找ParentLock下面所有的临时顺序节点并排序, 判断自己所创建的节点Lock3是不是顺序最靠前的一个, 结果同样发现节点Lock3并不是最小的. 

    于是, Client3向排序仅比它靠前的节点Lock2注册Watcher, 用于监听Lock2节点是否存在. 这意味着Client3同样抢锁失败, 进入了等待状态. 

    这样一来, Client1得到了锁, Client2监听了Lock1, Client3监听了Lock2. 这恰恰形成了一个等待队列, 很像是Java当中ReentrantLock所依赖的AQS(AbstractQueuedSynchronizer). 

2. 释放锁

    释放锁分为两种情况: 

    1. 任务完成, 客户端显示释放
    
        当任务完成时, Client1会显示调用删除节点Lock1的指令. 

    2. 任务执行过程中, 客户端崩溃

        获得锁的Client1在任务执行过程中, 如果Duang的一声崩溃, 则会断开与Zookeeper服务端的链接. 根据临时节点的特性, 相关联的节点Lock1会随之自动删除. 

    由于Client2一直监听着Lock1的存在状态, 当Lock1节点被删除, Client2会立刻收到通知. 这时候Client2会再次查询ParentLock下面的所有节点, 确认自己创建的节点Lock2是不是目前最小的节点. 如果是最小, 则Client2顺理成章获得了锁. 

    同理, 如果Client2也因为任务完成或者节点崩溃而删除了节点Lock2, 那么Client3就会接到通知. 

3. Zookeeper与Redis分布式锁比较

    |分布式锁|优点|缺点|
    |-----|-----|-----|
    |Zookeeper|1. 有封装好的框架, 容易实现<br/>2.有等待锁的队列, 大大提升抢锁效率|添加和删除节点性能较低|
    |Redis|Set 和 Del 指令的性能较高|1. 实现复杂, 需要考虑超时, 原子性, 误删<br/>2. 没有等待锁的队列, 只能在客户端自旋来等锁, 效率低下|



## 算法

 Zookeeper 是如何保证数据一致性的？这也是困扰分布式系统框架的一个难题. 

### 拜占庭将军问题

拜占庭将军问题是一个协议问题, 拜占庭帝国军队的将军们必须全体一致的决定是否攻击某一支敌军. 问题是这些将军在地理上是分隔开来的, 并且将军中存在叛徒. 叛徒可以任意行动以达到以下目标：欺骗某些将军采取进攻行动；促成一个不是所有将军都同意的决定, 如当将军们不希望进攻时促成进攻行动；或者迷惑某些将军, 使他们无法做出决定. 如果叛徒达到了这些目的之一, 则任何攻击行动的结果都是注定要失败的, 只有完全达成一致的努力才能获得胜利. 

###  Paxos 算法

 Paxos算法：一种基于消息传递且具有高度容错特性的一致性算法. 

 Paxos算法解决的问题：就是如何快速正确的在一个分布式系统中对某个数据值达成一致, 并且保证不论发生任何异常,  都不会破坏整个系统的一致性.  

<img src="http://www.milky.show/images/bigdata/zookeeper/10.png" alt="http://www.milky.show/images/bigdata/zookeeper/10.png" style="zoom:50%;" />

**Paxos 算法描述**

在一个Paxos系统中, 首先将所有节点划分为Proposer（提议者）, Acceptor（接受者）, 和Learner（学习者）. （注意：每个节点都可以身兼数职）. 

<img src="http://www.milky.show/images/bigdata/zookeeper/11.png" alt="http://www.milky.show/images/bigdata/zookeeper/11.png" style="zoom:50%;" />

一个完整的Paxos算法流程分为三个阶段： 

* Prepare准备阶段
    * Proposer向多个Acceptor发出Propose请求Promise（承诺） 
    * Acceptor针对收到的Propose请求进行Promise（承诺） 
* Accept接受阶段
    *   Proposer收到多数Acceptor承诺的Promise后, 向Acceptor发出Propose请求
    *   Acceptor针对收到的Propose请求进行Accept处理
* Learn学习阶段：Proposer将形成的决议发送给所有Learners 



**Paxos 算法流程**

<img src="http://www.milky.show/images/bigdata/zookeeper/12.png" alt="http://www.milky.show/images/bigdata/zookeeper/12.png" style="zoom:50%;" />

Prepare: Proposer生成全局唯一且递增的Proposal ID, 向所有Acceptor发送Propose请求, 这里无需携带提案内容, 只携带Proposal ID即可. 

Promise: Acceptor收到Propose请求后, 做出“两个承诺, 一个应答”. 

*   不再接受Proposal ID小于等于（注意：这里是<= ）当前请求的Propose请求. 
*   不再接受Proposal ID小于（注意：这里是< ）当前请求的Accept请求. 
*   不违背以前做出的承诺下, 回复已经Accept过的提案中Proposal ID最大的那个提案的Value和Proposal ID, 没有则返回空值. 

Propose: Proposer收到多数Acceptor的Promise应答后, 从应答中选择Proposal ID最大的提案的Value, 作为本次要发起的提案. 如果所有应答的提案Value均为空值, 则可以自己随意决定提案Value. 然后携带当前Proposal ID, 向所有Acceptor发送Propose请求. 

Accept: Acceptor收到Propose请求后, 在不违背自己之前做出的承诺下, 接受并持久化当前Proposal ID和提案Value. 

Learn: Proposer收到多数Acceptor的Accept后, 决议形成, 将形成的决议发送给所有Learner. 

下面我们针对上述描述做三种情况的推演举例：为了简化流程, 我们这里不设置Learner. 

**情况1**

有A1, A2, A3, A4, A5 5位议员, 就税率问题进行决议. 

<img src="http://www.milky.show/images/bigdata/zookeeper/13.png" alt="http://www.milky.show/images/bigdata/zookeeper/13.png" style="zoom:50%;" />

*   A1发起1号Proposal的Propose, 等待Promise承诺

*   A2-A5回应Promise

*   A1在收到两份回复时就会发起税率10%的Proposal

*   A2-A5回应Accept

*   通过Proposal, 税率10%. 

**情况2**

现在我们假设在A1提出提案的同时, A5决定将税率定为20% 

<img src="http://www.milky.show/images/bigdata/zookeeper/14.png" alt="http://www.milky.show/images/bigdata/zookeeper/14.png" style="zoom:50%;" />

*   A1, A5同时发起Propose（序号分别为1, 2） 

*   A2承诺A1, A4承诺A5, A3行为成为关键

*   情况1：A3先收到A1消息, 承诺A1. 

*   A1发起Proposal（1, 10%）, A2, A3接受. 

*   之后A3又收到A5消息, 回复A1：（1, 10%）, 并承诺A5. 

*   A5发起Proposal（2, 20%）, A3, A4接受. 之后A1, A5同时广播决议. 



**情况3**

现在我们假设在A1提出提案的同时, A5决定将税率定为20% 

<img src="http://www.milky.show/images/bigdata/zookeeper/15.png" alt="http://www.milky.show/images/bigdata/zookeeper/15.png" style="zoom:50%;" />

*   A1, A5同时发起Propose（序号分别为1, 2） 

*   A2承诺A1, A4承诺A5, A3行为成为关键

*   情况2：A3先收到A1消息, 承诺A1. 之后立刻收到A5消息, 承诺A5. 

*   A1发起Proposal（1, 10%）, 无足够响应, A1重新Propose （序号3）, A3再次承诺A1. 

*   A5发起Proposal（2, 20%）, 无足够相应. A5重新Propose （序号4）, A3再次承诺A5. 

*   …… 

造成这种情况的原因是系统中有一个以上的Proposer, 多个Proposers 相互争夺Acceptor, 造成迟迟无法达成一致的情况. 针对这种情况, 一种改进的Paxos 算法被提出：从系统中选出一个节点作为Leader, 只有Leader 能够发起提案. 这样, 一次Paxos 流程中只有一个Proposer, 不会出现活锁的情况, 此时只会出现例子中第一种情况. 



### ZAB 协议

Zab 借鉴了Paxos 算法, 是特别为Zookeeper 设计的支持崩溃恢复的原子广播协议. 基于该协议, Zookeeper 设计为只有一台客户端（Leader）负责处理外部的写事务请求, 然后Leader 客户端将数据同步到其他Follower 节点. 即Zookeeper 只有一个Leader 可以发起提案. 

**Zab 协议内容**

Zab 协议包括两种基本的模式：消息广播、崩溃恢复. 

**消息广播**

<img src="http://www.milky.show/images/bigdata/zookeeper/16.png" alt="http://www.milky.show/images/bigdata/zookeeper/16.png" style="zoom:50%;" />

1.  客户端发起一个写操作请求. 

2.  Leader服务器将客户端的请求转化为事务Proposal提案, 同时为每个Proposal分配一个全局的ID, 即zxid. 

3.  Leader服务器为每个Follower服务器分配一个单独的队列, 然后将需要广播的Proposal依次放到队列中去, 并且根据FIFO策略进行消息发送. 

4.  Follower接收到Proposal后, 会首先将其以事务日志的方式写入本地磁盘中, 写入成功后向Leader反馈一个Ack响应消息. 

5.  Leader接收到超过半数以上Follower的Ack响应消息后, 即认为消息发送成功, 可以发送commit消息. 

6.  Leader向所有Follower广播commit消息, 同时自身也会完成事务提交. Follower接收到commit消息后, 会将上一条事务提交. 

7.  Zookeeper采用Zab协议的核心, 就是只要有一台服务器提交了Proposal, 就要确保所有的服务器最终都能正确提交Proposal. 

ZAB协议针对事务请求的处理过程类似于一个两阶段提交过程

*   广播事务阶段

*   广播提交操作

这两阶段提交模型如下, 有可能因为Leader宕机带来数据不一致, 比如

* Leader发起一个事务Proposal1后就宕机, Follower都没有Proposal1

* Leader收到半数ACK宕机, 没来得及向Follower发送Commit

怎么解决呢？**ZAB**引入了崩溃恢复模式. 

**崩溃恢复-异常假设**

一旦Leader服务器出现崩溃或者由于网络原因导致Leader服务器失去了与过半Follower的联系, 那么就会进入崩溃恢复模式. 

<img src="http://www.milky.show/images/bigdata/zookeeper/17.png" alt="http://www.milky.show/images/bigdata/zookeeper/17.png" style="zoom:50%;" />

**假设两种服务器异常情况**

*   假设一个事务在Leader提出之后, Leader挂了
*   一个事务在Leader上提交了, 并且过半的Follower都响应Ack了, 但是Leader在Commit消息发出之前挂了

**Zab协议崩溃恢复要求满足以下两个要求**

*   确保已经被Leader提交的提案Proposal, 必须最终被所有的Follower服务器提交. （已经产生的提案, **Follower**必须执行）
*   确保丢弃已经被Leader提出的, 但是没有被提交的Proposal. （丢弃胎死腹中的提案）

**崩溃恢复-Leader选举**

崩溃恢复主要包括两部分：**Leader**选举和数据恢复. 

<img src="http://www.milky.show/images/bigdata/zookeeper/18.png" alt="http://www.milky.show/images/bigdata/zookeeper/18.png" style="zoom:50%;" />

**Leader**选举：根据上述要求, Zab协议需要保证选举出来的Leader需要满足以下条件

*   新选举出来的Leader不能包含未提交的Proposal. 即新**Leader**必须都是已经提交了**Proposal**的**Follower**服务器节点. 
*   新选举的**Leader**节点中含有最大的**zxid**. 这样做的好处是可以避免Leader服务器检查Proposal的提交和丢弃工作. 

**崩溃恢复-数据恢复**

崩溃恢复主要包括两部分：**Leader**选举和数据恢复. 

<img src="http://www.milky.show/images/bigdata/zookeeper/19.png" alt="http://www.milky.show/images/bigdata/zookeeper/19.png" style="zoom:50%;" />

**Zab**如何数据同步

*   完成Leader选举后, 在正式开始工作之前（接收事务请求, 然后提出新的Proposal）, **Leader**服务器会首先确认事务日志中的所有的**Proposal**是否已经被集群中过半的服务器 **Commit**
*   Leader服务器需要确保所有的Follower服务器能够接收到每一条事务的Proposal, 并且能将所有已经提交的事务Proposal应用到内存数据中. 等到**Follower**将所有尚未同步的事务**Proposal**都从**Leader**服务器上同步过, 并且应用到内存数据中以后, Leader才会把该Follower加入到真正可用的Follower列表中

**崩溃恢复-异常提案处理**

**Zab**数据同步过程中, 如何处理需要丢弃的**Proposal**？

在Zab的事务编号zxid设计中, zxid是一个64位的数字. 其中低32位可以看成一个简单的单增计数器, 针对客户端每一个事务请求, Leader在产生新的Proposal事务时, 都会对该计数器加1. 而高32位则代表了Leader周期的epoch编号. 

epoch编号可以理解为当前集群所处的年代, 或者周期. 每次Leader变更之后都会在epoch的基础上加1, 这样旧的Leader崩溃恢复之后, 其他Follower也不会听它的了, 因为Follower只服从epoch最高的Leader命令. 

每当选举产生一个新的Leader, 就会从这个Leader服务器上取出本地事务日志充最大编号Proposal的zxid, 并从zxid中解析得到对应的epoch编号, 然后再对其加1, 之后该编号就作为新的epoch值, 并将低32位数字归零, 由0开始重新生成zxid. 

Zab协议通过epoch编号来区分Leader变化周期, 能够有效避免不同的Leader错误的使用了相同的zxid编号提出了不一样的Proposal的异常情况. 

基于以上策略, 当一个包含了上一个Leader周期中尚未提交过的事务Proposal的服务器启动时, 当这台机器加入集群中, 以Follower角色连上Leader服务器后, Leader服务器会根据自己服务器上最后提交的Proposal来和Follower服务器的Proposal进行比对, 比对的结果肯定是Leader要求Follower进行一个回退操作, 回退到一个确实已经被集群中过半机器Commit的最新Proposal. 



## 面试

### 选举机制

半数机制, 超过半数的投票通过, 即通过. 

1.  第一次启动选举规则: 

    投票过半数时, 服务器id大的胜出

2.  第二次启动选举规则: 

    1.  EPOCH大的直接胜出

    2.  EPOCH相同, 事务id大的胜出

    3.  事务id相同, 服务器id大的胜出

### 生产集群安装多少**zk**合适？

安装奇数台. 

生产经验: 

*   10台服务器: 3台zk；

*   20台服务器: 5台zk

*   100台服务器: 11台zk

*   200台服务器: 11台zk 

服务器台数多: 好处, 提高可靠性；坏处: 提高通信延时

### 常用命令

ls、get、create、delete 
