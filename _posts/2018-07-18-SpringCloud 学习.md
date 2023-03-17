---
layout: post
title: "SpringCloud 学习"
categories: [Framework]
description:
keywords:
---

* content
{:toc}
## 微服务

一系列微小的服务组成

跑在自己的进程中

每个服务为独立的业务

独立部署

分布式的管理

### 不适合微服务的场景

**系统中包含很多强事务场景的不适合做微服务**

**业务相对稳定, 迭代周期长(稳定项目)**

**访问压力不大, 可用性要求不高(后台 OA)**

**...**

### 如何拆分功能

单一职责, 松耦合(不同服务尽量不影响), 高内聚(所有行为放到一个服务)

关注点分离

### 服务和数据的关系

先考虑业务功能, 在考虑数据

无状态服务

## 注册中心Spring Cloud Eureka

Eureka Server 提供服务注册服务, 各个节点启动后, 会在 Eureka Server 中进行注册, 这样 EurekaServer 中的**服务注册表**中将会存储所有可用服务节点的信息, 服务节点的信息可以在界面中直观的看到. 

Eureka Client 是一个 java 客户端, 用于简化与 Eureka Server 的交互, 客户端同时也就是一个内置的、使用轮询(round-robin)负载算法的负载均衡器. 



服务启动后向Eureka注册, ***\*Eureka Server会将注册信息向其他Eureka Server进行同步, 当服务消费者要调用服务提供者, 则向服务注册中心获取服务提供者地址, 然后会将服务提供者地址缓存在本地, 下次再调用时, 则直接从本地缓存中取, 完成一次调用\****

当服务注册中心Eureka Server检测到服务提供者因为宕机、网络原因不可用时, 则在服务注册中心将服务置为`DOWN`状态, 并把当前服务提供者状态向订阅者发布, 订阅过的服务消费者更新本地缓存. 

**服务提供者(节点)在启动后, 周期性(默认30秒)向Eureka Server发送心跳, 以证明当前服务是可用状态. Eureka Server在一定的时间(默认90秒)未收到客户端的心跳, 则认为服务宕机, 注销该实例**



@EnableEurekaServer, @EnableEurekaClient, @EnableDiscoveryClient

基于 Netflix Eureka 做了二次封装

两部分组成

* Eureka Server 注册中心
* Eureka Client 服务注册

**Eureka 各个节点是平等的, 几个节点挂掉不会影响正常的工作, 剩余的节点依然可以提供注册和查询服务. Eureka 的客户端再向某个 Eureka 注册时如果发现连接失败, 则会自动切换至其他节点, 只要有一台 Eureka 还在, 就可以保证服务可用, 只不过查询到的信息可能不是最新的. Eureka 还用一种自我保护机制, 如果在 15 分钟内超过85%的节点没有正常心跳, 那么 Eureka 就认为客户端与注册中心出现网络故障, 会出现以下几种情况:** 

1. Eureka 不在从注册列表中移除因为长时间没收到心跳而应该过期的服务

2. Eureka 仍然能够接受新服务注册和查询请求, 但是不会同步到其他节点上

3. 当网络稳定时, 当前实例新的注册信息会同步到其他节点中



### Spring Cloud Eureka Server

启动类加入 `@EnableEurekaServer` 注解

```java
@SpringBootApplication
@EnableEurekaServer
public class SpringcloudSellEurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellEurekaApplication.class, args);
    }
}
```

SpringCloudEurekaServer 既是服务端又是客户端, 启动时会默认查找 defaultZone 注册, 也会注册自己, 采用心跳的方式每隔一段时间注册一次, 可以改变配置不让自己注册

```yaml
server:
  port: 9900
eureka:
  client:
    service-url:
      defaultZone: http://localhost:9900/eureka/ # 寻找注册中心地址
    register-with-eureka: false # 不向注册中心注册自己
    fetch-registry: false # 不向注册中心拉取配置
spring:
  application:
    name: springcloud-sell-eureka
```

自我保护模式: Eureka 采用心跳机制, Server 端会一直检查 Client 端是否在线, 当 Client 上线率过低时会报出警告, 但是 Server 会认为是否是网络问题导致的, 此时并不会立刻剔除客户端注册信息, 开发时建议关闭, 可以实时更新服务注册信息**(生产不要关)**

```yaml
eureka:
  server:
    enable-self-preservation: false # 是否开启自我保护机制
```

查看注册信息

```http
http://127.0.0.1:9901/eureka/apps
```

### Spring Cloud Eureka Client

启动类加入 `@EnableDiscoveryClient` 注解

```java
@SpringBootApplication
@EnableDiscoveryClient
public class SpringcloudSellClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellClientApplication.class, args);
    }
}
```

### 高可用

拷贝 EurekaServer 项目, 修改端口号, 修改服务注册地址为其他 EurekaServer 的地址

```yaml
server:
  port: 9901
eureka:
  client:
    service-url:
      defaultZone: http://eureka9902:9902/eureka/,http://eureka9903:9903/eureka/ # 寻找注册中心地址
```

EurekaClient 的地址填写集群的每一个地址

```yaml
server:
  port: 9910
eureka:
  client:
    service-url:
      # 寻找注册中心地址
      defaultZone: http://eureka9901:9901/eureka/,http://eureka9902:9902/eureka/,http://eureka9903:9903/eureka/
```

### 服务发现的两种方式

#### 客户端发现

客户端获取所有服务端地址, 需要自己实现负载均衡逻辑去调用, SpringCloud 就采用这种方式

Eureka: http://eureka9901:9901/eureka/apps 可以查看注册信息

#### 服务端发现

需要代理服务的介入, 对客户端是完全透明的

Nginx

Zookeeper

Kubernetes

Eureka



### Eureka 心跳健康检查机制

运行阶段执行健康检查的目的是为了从 Eureka 服务器注册表中识别并删除不可访问的微服务, Eureka 服务器并不是向客户端发送心跳请求, 而是反过来, Eureka 客户端将心跳发送到 Eureka 服务器, 让服务器了解其状态. 

这些心跳机制就需要在微服务嵌入一个客户端, 用来发送心跳, 但是客户端本身必须确定其健康状态, 而且 Eureka 服务器必须为客户端公开一些 REST 操作以让其发布心跳. 

Eureka 服务器向客户端公开下面资源以让其发送心跳: 

PUT /eureka/apps/{app id}/{instance id}?status={status}

{instance id}采用 hostname:app id:port, 其中 app id 代表标识唯一的 Eureka 客户端实例, Eureka 服务器会识别一些状态数值: UP; DOWN; STARTING; OUT_OF_SERVICE; UNKNOWN.

客户端发送心跳时的URL如下: 

PUT /eureka/apps/ORDER-SERVICE/localhost:order-service:8886?status=UP

Eureka 服务器收到心跳请求后, 会续订该实例的租约. 如果是第一个心跳, 则Eureka服务器以404响应, 之后客户端必须首先发送注册请求. 

此外,  Eureka 服务器公开以下操作以允许健康状态的修改和删除: 

PUT /eureka/apps/{app id}/{instance id}/status?value={status}

DELETE /eureka/apps/{app id}/{instance id}/status

修改操作(即 PUT 上面的操作)是用于手动获取健康的实例 OUT_OF_SERVICE 时操作, 或者使用 Asgard 等管理工具 (暂时禁止某些实例的流量)时操作. 

这种修改操作对于“红/黑”部署非常有用, 在这种情况下, 你可以在一段时间内运行较旧版本的微服务(如果新版本不稳定, 则可以轻松回滚到旧版本). 完成新版本的部署并且新版本开始为请求提供服务后, 可以让旧版本OUT_OF_SERVICE(但不会让他们停止)暂停提供请求服务. 即

PUT /eureka/apps/ORDER-SERVICE/localhost:order-service:8886/statusvalue=OUT_OF_SERVICE

上面修改的状态也可以被丢弃, 我们可以指示让 Eureka 服务器开始遵守实例本身发布的状态, 如下所示: 

DELETE /eureka/apps/ORDER-SERVICE/localhost:order-service:8886/status

当您发现微服务的新版本不稳定并且您希望获得旧版本(即已经被打上 OUT_OF_SERVICE 标记的版本)以开始提供请求时, 上述办法非常有用. 



### SpringCloudEureka 常用配置

#### EurekaServer 配置

```yaml
eureka:
  server:
    # 启用主动失效, 并且每次主动失效检测间隔为 3s
    # 默认 60s
    eviction-interval-timer-in-ms: 3000
    # 关闭自我保护
    enable-self-preservation: false
    # eureka server 缓存 readWriteCacheMap 失效时间, 这个只有在这个时间过去后缓存才会失效, 失效前不会更新, 过期后从 registry 重			  新读取注册服务信息, registry 是一个 ConcurrentHashMap. 
    # 由于启用了 evict 其实就用不太上改这个配置了
    # 默认 180s
    responseCacheAutoExpirationInSeconds: 180
    # eureka server 刷新 readCacheMap 的时间, 注意, client 读取的是 readCacheMap, 这个时间决定了多久会把 readWriteCacheMap 		  的缓存更新到 readCacheMap 上
    # 默认30s
    responseCacheUpdateIntervalMs: 3000

  instance:
    # 配置指示 eureka 服务器在接收到最后一个心跳之后等待的时间, 然后才能从列表中删除此实例(Spring Cloud默认该配置是 90s)
    # 注意, EurekaServer 一定要设置 eureka.server.eviction-interval-timer-in-ms 否则这个配置无效, 这个配置一般为服务刷新时间配置的三倍
    # 默认 90s
    lease-expiration-duration-in-seconds: 15
    # 该配置指示eureka客户端需要向eureka服务器发送心跳的频率  (Spring Cloud默认该配置是 30s)
    # 默认 30s
    lease-renewal-interval-in-seconds: 5
    preferIpAddress: true
    
  client:
    serviceUrl:
      defaultZone: http://10.120.242.153:8211/eureka/,http://10.120.242.154:8211/eureka/
```

#### EurekaClient 配置

```yaml
eureka:
  instance:
    # 该配置指示eureka客户端需要向eureka服务器发送心跳的频率(Spring Cloud 默认该配置是 30s)
    # 默认 30s
    lease-renewal-interval-in-seconds: 5
    # 配置指示 eureka 服务器在接收到最后一个心跳之后等待的时间, 然后才能从列表中删除此实例(Spring Cloud 默认该配置是 90s)
    # 注意, EurekaServer 一定要设置 eureka.server.eviction-interval-timer-in-ms 否则这个配置无效, 这个配置一般为心跳时间配置的三倍
    # 默认 90s
    lease-expiration-duration-in-seconds: 15
    preferIpAddress: true
  
  client:
    # eureka client 间隔多久去拉取服务注册信息, 对于 api-gateway 如果要迅速获得服务注册状态, 可以缩小该值, 比如 5 秒, 默认 30s
    registry-fetch-interval-seconds: 5
    fetch-registry: true
    serviceUrl:
      defaultZone: http://10.120.242.153:8211/eureka/,http://10.120.242.154:8211/eureka/
# eureka 客户端 ribbon 刷新时间
# 默认 30s
ribbon:
  ServerListRefreshInterval: 1000
```



### SpringCloud 注册中心 Eureka 集群是怎么保持数据一致的

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_8.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_8.png" style="zoom: 50%;" />

服务注册中心不可能是单点的, 一定会有一个集群, 那么集群中的服务注册信息如何在集群中保持一致的呢? 

首先要明确的是 Eureka 是**弱数据一致性**的. 

下面从 2 个方面来说明: 

1.  什么是弱数据一致性
2.  Eureka 是如何同步数据的

#### 弱数据一致性

我们知道 ZooKeeper 也可以实现数据中心, ZooKeeper 就是强一致性的. 

分布式系统中有一个重要理论: CAP. 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_9.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_9.png" style="zoom: 50%;" />

该理论提到了分布式系统中的3个特性: 

-   Consistency 数据一致性

    分布式系统中, 数据会存在多个副本中, 有一些问题会导致写入数据时, 一部分副本成功、一部分副本失败, 造成数据不一致. 

​	满足一致性就要求对数据的更新操作成功后, 多副本的数据必须保持一致. 

-   Availability 可用性

    在任何时候客户端对集群进行读写操作时, 请求能够正常响应. 

-   Partition Tolerance 分区容忍性

    发生通信故障时, 集群被分割为多个无法通信的分区时, 集群仍然可用. 

>   CAP 理论指出: 这3个特性不可能同时满足, 最多满足2个. 

**P** 是客观存在的, *不可绕过*, 那么就是选择 **C** 还是选择 **A**. 

ZooKeeper 选择了 **C**, 就是尽可能的保证数据一致性, 某些情况下可以牺牲可用性. 

Eureka 则选择了 **A**, 所以 Eureka 具有高可用性, 在任何时候, 服务消费者都能正常获取服务列表, 但不保证数据的强一致性, 消费者可能会拿到过期的服务列表. 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_10.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_10.png" style="zoom: 50%;" />

>   Eureka 的设计理念: 保留可用及过期的数据总比丢掉可用的数据好. 

#### Eureka 的数据同步方式

##### 复制方式

分布式系统的数据在多个副本之间的复制方式, 主要有: 

-   主从复制

    就是 **Master-Slave** 模式, 有一个主副本, 其他为从副本, 所有写操作都提交到主副本, 再由主副本更新到其他从副本. 

    写压力都集中在主副本上, 是系统的瓶颈, 从副本可以分担读请求. 

-   对等复制

    就是 **Peer to Peer** 模式, 副本间不分主从, 任何副本都可以接收写操作, 然后每个副本间互相进行数据更新. 

    对等复制模式, 任何副本都可以接收写请求, 不存在写压力瓶颈, 但各个副本间数据同步时可能产生数据冲突. 

    Eureka 采用的就是 **Peer to Peer** 模式. 

##### 同步过程

Eureka Server 本身依赖了 Eureka Client, 也就是每个 Eureka Server 是作为其他 Eureka Server 的 Client. 

Eureka Server 启动后, 会通过 Eureka Client 请求其他 Eureka Server 节点中的一个节点, 获取注册的服务信息, 然后复制到其他 peer 节点. 

Eureka Server 每当自己的信息变更后, 例如 Client 向自己发起*注册、续约、注销*请求,  就会把自己的最新信息通知给其他 Eureka Server, 保持数据同步. 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_11.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_11.png" style="zoom: 50%;" />

如果自己的信息变更是另一个 Eureka Server 同步过来的, 这是再同步回去的话就出现**数据同步死循环**了. 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_12.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_12.png" style="zoom: 50%;" />

Eureka Server 在执行复制操作的时候, 使用 `HEADER_REPLICATION` 这个 http header 来区分普通应用实例的正常请求, 说明这是一个复制请求, 这样其他 peer 节点收到请求时, 就不会再对其进行复制操作, 从而避免死循环. 

还有一个问题, 就是**数据冲突**, 比如 server A 向 server B 发起同步请求, 如果 A 的数据比 B 的还旧, B 不可能接受 A 的数据, 那么 B 是如何知道 A 的数据是旧的呢? 这时 A 又应该怎么办呢? 

数据的新旧一般是通过*版本号*来定义的, Eureka 是通过 `lastDirtyTimestamp` 这个类似版本号的属性来实现的. 

>   `lastDirtyTimestamp` 是注册中心里面服务实例的一个属性, 表示此服务实例最近一次变更时间. 

比如 Eureka Server A 向 Eureka Server B 复制数据, 数据冲突有2种情况: 

1.  A 的数据比 B 的新, B 返回 404, A 重新把这个应用实例注册到 B. 

2.  A 的数据比 B 的旧, B 返回 409, 要求 A 同步 B 的数据. 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_13.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_13.png" style="zoom: 50%;" />

还有一个重要的机制:**hearbeat 心跳**, 即续约操作, 来进行数据的最终修复, 因为节点间的复制可能会出错, 通过心跳就可以发现错误, 进行弥补. 

例如发现某个应用实例数据与某个 server 不一致, 则 server 返回 404, 实例重新注册即可. 

#### 小结

Eureka 是弱数据一致性, 选择了 CAP 中的 AP. 

Eureka 采用 Peer to Peer 模式进行数据复制. 

Eureka 通过 lastDirtyTimestamp 来解决复制冲突. 

Eureka 通过心跳机制实现数据修复. 



## 应用间通信 RestTemplate 和 Feign

### RestTemplate(面向服务)

基于 Netflix Ribbon 实现的一套 http 客户端负载均衡工具, Ribbon + RestTemplate, 结合 eureka 使用, 会从 eureka 中查找可用的机器进行访问

```java
@RestController
@Slf4j
public class ClientController {

    // @Autowired
    // private LoadBalancerClient loadBalancerClient;

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/getProductMsg")
    public String getProductMsg() {
        // 1. 第一种方式(直接使用 RestTemplate, url 写死)
        // RestTemplate restTemplate = new RestTemplate();
        // String response = restTemplate.getForObject("http://localhost:9916/msg", String.class);


        // 2. 第二种方式(利用 LoadBalancerClient 通过应用名获取 url, 然后再使用 RestTemplate)
        // ServiceInstance serviceInstance = loadBalancerClient.choose("SPRINGCLOUD-SELL-PRODUCT");
        // String url = String.format("http://%s:%s/msg", serviceInstance.getHost(), serviceInstance.getPort());
        // RestTemplate restTemplate = new RestTemplate();
        // String response = restTemplate.getForObject(url, String.class);

        // 3. 第三种方式(利用 @LoadBalanced 注解, 可在 RestTemplate 里使用应用名字)
        String response = this.restTemplate.getForObject("http://SPRINGCLOUD-SELL-PRODUCT/msg", String.class);
        log.error("response = [{}]", response);
        return response;
    }

}
```

```java
@Component
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

#### Ribbon 组件

Netflix Ribbon 是客户端负载均衡器, 是 LoadBalance 实现负载均衡的组件, 可以实现**服务发现, 服务选择规则, 服务监听**, **RestTemplate, Feign, Zuul 均使用该组件**

* ServerList: 获取所有可用列表
* IRule: 根据规则获取一个地址
* ServerListFilter: 过滤掉一部分服务地址

#### Ribbon 轮询规则

RoundRobinRule: 轮询

RandomRule: 随机

RetryRule: 先按照 `RoundRobinRule` 的策略获取服务, 如果获取服务失败则在指定时间内会进行重试, 获取可用的服务

WeightedResponseTimeRule: 对 `RoundRobinRule` 的扩展, 响应速度越快的实例选择权重越大, 越容易被选择

BestAvailableRule: 会过滤掉由于多次访问故障而处于断路器跳闸状态的服务, 然后选择一个并发量最小的服务

AvailabilityFilteringRule: 先过滤掉故障实例, 再选择并发较小的实例

ZoneAvoidanceRule: 默认规则, 复合判断 server 所在区域的性能和 server 的可用性选择服务器



### Feign(面向接口)

声明式 REST 客户端(伪RPC), 采用了基于接口的注解 @FeignClient, 内部也使用了 Ribbon 做负载均衡

加入 Feign 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

启动类加入 @EnableFeignClients 注解

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SpringcloudSellOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellOrderApplication.class, args);
    }

}
```

编写客户端接口, 使用 @FeignClient 标明服务端名称, 接口方法是服务接口地址

```java
@FeignClient(name = "SPRINGCLOUD-SELL-PRODUCT")
public interface ProductClient {

    @GetMapping("/msg")
    String productMsg();

}
```

## 分布式统一配置中心 Config

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_1.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_1.png" style="zoom: 50%;" />

* 配置的内容安全与权限

- 集中管理配置文件方便维护

- 不同环境不同配置, 动态化配置更新, 分环境部署比如dev/test/prod

- 运行期间动态调整配置, 不需要再每个服务部署的机器上编写配置文件, 服务会向配置中心统一拉取配置自己的信息

- 当配置发生变动时, 服务不需要重启即可感知到配置的变化并应用新的配置

- 将配置中心以REST接口的形式暴露, 后缀可以是 **yml, properties, json**

    /{name}-{profiles}.yml

    **/{label}/{name}-{profiles}.yml** 常用

    name: 即文件名

    profiles: 环境

    label: git 中的分支branch, 不写的话默认是 master 分支

### ConfigServer

* 加入 ConfigServer 依赖, config 本身也是一个微服务, 需要注册到 eureka 中

    ```xml
    <dependencies>
    	<dependency>
    		<groupId>org.springframework.cloud</groupId>
    		<artifactId>spring-cloud-config-server</artifactId>
    	</dependency>
    	<dependency>
    		<groupId>org.springframework.cloud</groupId>
    		<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    	</dependency>
    </dependencies>
    ```

    

* 开启服务端配置中心

    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    @EnableConfigServer
    public class SpringcloudSellConfigApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(SpringcloudSellConfigApplication.class, args);
        }
    
    }
    ```

    

* 编写配置文件

    ```yaml
    server:
      port: 9936
    spring:
      application:
        name: springcloud-sell-config
      cloud:
        config:
          server:
            git:
              uri: https://github.com/miaomiaoqi/springcloud-sell-config # 拉取 git 仓库
              username: 363962900@qq.com
              password: # 自己填写
              # 项目启动时会拷贝一份配置到本地,因为权限问题可以指定拷贝的目录
              basedir: /Users/miaoqi/Documents/study/language/java/springcloud-sell/localconfig
    eureka:
      client:
        service-url:
          defaultZone: http://eureka9901:9901/eureka,http://eureka9902:9902/eureka,http://eureka9903:9903/eureka
    ```

* 访问地址举例:

    http://localhost:9936/release/springcloud-sell-order-test.yml



### ConfigClient

* 加入依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>
    ```

    

* **修改配置文件 bootstrap.yml, bootstrap.yml 加载优先级最高, 所以需要先加载 config server 的配置文件才能继续执行, 具体配置加载顺序参考 springboot**

    ```yaml
    spring:
      application:
        name: springcloud-sell-order
      cloud:
        config:
          profile: dev # 指定环境
          uri: http://localhost:9936 # config server 服务 uri, 集群指定多个
          label: master
    #      discovery:
    #        service-id: SPRINGCLOUD-SELL-CONFIG
    #        enabled: true
    ```

    **如果配置 spring.cloud.config.uri 就不需要指定 eureka 与 discovery 了, 该配置会直接从 config server 拉取配置. 如果配置 discovery 就需要配置 eureka, 要从注册中心找到 config server 服务拉取配置, uri 与 discovery 二者选一**

    **上述配置会从 config server 服务拉取 label 指定的分支(默认为 master 分支) 文件名为 springcloud-sell-order-dev.yml(spring.application.name-spring.cloud.config.profie.yml) 的配置文件, 在拉取该配置文件前, 还会拉取 application.yml(所有项目通用配置文件) 与 springcloud-sell-order.yml(同一项目不同 profile 的通用配置文件)配置文件, 三个配置文件的内容会合到一起, application.yml > springcloud-sell-order.yml > springcloud-sell-order-dev.yml**



## 自动刷新配置 Spring Cloud Bus

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_2.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_2.png" style="zoom: 50%;" />

SpringCloudBus 依赖 mq 发消息实现服务自动更新配置

* Config Server 服务加入 SpringCloudBus 依赖

    ```xml
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
    ```

* Config Server 服务加入 rabbitmq 配置

    ```yaml
    spring:
      rabbitmq:
        addresses: 127.0.0.1:6672
        username: guest
        password: guest
        virtual-host: /springcloud-sell
        connection-timeout: 15000
    management:
      endpoints:
        web:
          exposure:
            include: "*" # 暴露对外访问的接口, 使得我们可以访问 http://localhost:9936/actuator/bus-refresh
    ```

* 启动 ConfigServer 服务, 会在 rabbitmq 中自动创建一个 queue, 如下图

    ![https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_3.png](https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_3.png)

    

* Config Client 即 Order 服务加入 SpringCloudBus 依赖

    ```xml
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
    ```

* Config Client 即 Order 服务在 git 配置文件中加入 rabbitmq 配置

    ```yaml
    spring:
      rabbitmq:
        addresses: 127.0.0.1:6672
        username: guest
        password: guest
        virtual-host: /springcloud-sell
        connection-timeout: 15000
    ```

* Config Client 的 Controller 类中加入 @RefreshScope 注解

    ```java
    @RestController
    @RequestMapping("/env")
    @RefreshScope # 哪里需要自动刷新哪里就加该注解
    public class EnvController {
    
        @Value("${env}")
        private String env;
    
        @GetMapping("/print")
        public String print() {
            return env;
        }
    }
    ```

* 启动 Config Client 服务, 会在 rabbitmq 中自动创建一个 queue, 如下图

    ![https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_4.png](https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_4.png)

* 修改 git 仓库中的配置文件内容, 向 ConfigServer 发送一条 POST 请求

    curl -X POST "http://localhost:9936/actuator/bus-refresh", 发送该请求后可以看到 queue 中多了一条消息

* 查看 order 服务中获取到的配置文件内容已经发生了变化

* **目前为止我们实现了手动的自动刷新, 接下来要配置 git 服务器的 webhooks 实现更改配置后自动 push**

    <img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_5.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_5.png" style="zoom: 50%;" />

    **我使用的是 git 所以需要配置外网域名进行 push, 生产环境我们可以搭建 gitlab 在内网中使用更安全**

## 消息队列 AMQP

amqp 定义了一系列消息接口, 典型的实现是 rabbitmq, springcloud 默认使用的实现就是 rabbitmq

加入 amqp 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
</dependencies>
```

修改 git 中的配置文件

```yaml
spring:
  rabbitmq:
    addresses: 127.0.0.1:6672
    username: guest
    password: guest
    virtual-host: /springcloud-sell
    connection-timeout: 15000
```



## 路由网关 Zuul

**Zuul 的核心是一系列的过滤器**, 过滤器配合路由就是 Zuul 的本质了, Zuul 是 Netflix 公司的产品, Zuul1.x 的内部是 servlet 阻塞模型, Zuul2.x 采用的是 netty 的非阻塞模型, 但是 Zuul2.x 没有整合进入 SpringCloud, SpringCloud 推出了自己的网关 SpringCloudGateway 需要结合 Webflux 使用

- 稳定性, 高可用
- 性能, 并发
- 安全性
- 扩展性

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_6.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_6.png" style="zoom:67%;" />

加入 Zuul 依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

启动类加入 @EnableZuulProxy 注解

```java
@SpringBootApplication
@EnableZuulProxy
public class SpringcloudSellGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellGatewayApplication.class, args);
    }

}
```

修改 git 中的配置文件

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka9901:9901/eureka/,http://eureka9902:9902/eureka/,http://eureka9903:9903/eureka/
    register-with-eureka: true
    fetch-registry: true
server:
  port: 9946
spring:
  application:
    name: springcloud-sell-gateway
env: dev33
```

修改项目中 bootstrap.yml 文件

```yaml
spring:
  application:
    name: springcloud-sell-gateway
  cloud:
    config:
      profile: dev # 指定环境
      uri: http://localhost:9936 # config server地址
      label: master # git 分支
```

启动项目, 通过 zuul 服务访问 product 服务, 这里是默认路由

http://localhost:9946/springcloud-sell-product/product/list

springcloud-sell-product 就是 product 服务在 eureka 中的服务 id, gateway-zuul 会默认配置

### 自定义路由

修改 git 中配置文件

```yaml
zuul:
  routes:
    myPorudct: # 定义一个路由规则, 规则名字可以任意起
      path: /myProduct/** # 路由匹配的路径
      serviceId: SPRINGCLOUD-SELL-PRODUCT # 转发到的服务
      stripPrefix: true # 是否去除前缀
      sensitiveHeaders:
    # SPRINGCLOUD-SELL-PRODUCT: /myProduct/** 如果只有 serviceId 和 path 可以简写成这样, 默认去掉前缀
  ignored-patterns: # 忽略的路由规则, 即禁止访问, 是个 Set 集合
    - /**/product/listForOrder
  sensitive-headers: # 忽略全部服务敏感头
management:
  endpoints:
    web:
      exposure:
        include: "*" # 暴露对外访问的接口, 以便使用 http://localhost:9946/actuator/routes
```

访问 http://localhost:9946/myProduct/product/list 依旧有效, 证明我们自定义的规则生效了, 会将 myProdut 转发到 product 服务中, 并且默认去除掉 myProduct 前缀

使用 http://localhost:9946/actuator/routes 查看全部的路由规则

### 传递 Cookie

路由规则中默认设置了敏感头, Zuul 会过滤掉这些值, 我们需要手动去除敏感头, 见上述配置

默认的敏感头

```java
private Set<String> sensitiveHeaders = new LinkedHashSet(Arrays.asList("Cookie", "Set-Cookie", "Authorization"));
```

### 动态更新配置

* 将配置文件放到 git 中

* Zuul 服务集成 springcloud-bus

* 配置刷新域, 我们在配置文件中配置的 zuul 开头的配置, 都是配置的 ZuulProperties 中的属性

    第一种方式, 自己管理 ZuulProperties, 配置 @RefreshScope 注解

    ```java
    @Component
    public class ZuulConfig {
    
        @ConfigurationProperties(prefix = "zuul")
        @RefreshScope
        public ZuulProperties zuulProperties() {
            return new ZuulProperties();
        }
    
    }
    ```

### 过滤器

前置(Pre): 限流, 鉴权, 参数校验调整

路由(Route)

后置(Post): 统计, 日志, 跨域

错误(Error)

### 鉴权

利用前置过滤器, 需要前端携带参数 token 才能通过, 生产中校验 cookie 中的 jwt

```java
@Component
public class TokenFilter extends ZuulFilter {

    @Override
    public String filterType() {
        // FilterConstants
        // public static final String ERROR_TYPE = "error";
        // public static final String POST_TYPE = "post";
        // public static final String PRE_TYPE = "pre";
        // public static final String ROUTE_TYPE = "route";
        // 前置过滤器
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
    	// 判断是否需要拦截, 假如是登录就不需要拦截
    	// RequestContext requestContext = RequestContext.getCurrentContext();
        // HttpServletRequest request = requestContext.getRequest();
        // String uri = request.getRequestURI();
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        // 这里从 url 参数中, 也可以从 cookie, header 中获取
        String token = request.getParameter("token");
        if (StringUtils.isEmpty(token)) {
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```

### 限流

利用前置过滤器, **在请求被转发之前调用, 我们放在鉴权过滤器前**

```java
package com.miaoqi.springcloudsell.gateway.filter;

import com.google.common.util.concurrent.RateLimiter;
import com.miaoqi.springcloudsell.gateway.exception.RateLimitException;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;

/**
 * 限流拦截器
 *
 * @author miaoqi
 * @date 2019-06-12
 */
public class RateLimitFilter extends ZuulFilter {

    /**
     * google guava 中封装好的令牌限流桶, 每秒放入 100 个
     */
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(100);

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    /**
     * 优先级要最高
     */
    @Override
    public int filterOrder() {
        return FilterConstants.SERVLET_DETECTION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        // 尝试获取一个令牌
        if (!RATE_LIMITER.tryAcquire()) {
            // 如果令牌桶中的令牌不够, 就直接返回错误状态, 或者抛异常
            // requestContext.setSendZuulResponse(false);
            // requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
            throw new RateLimitException();
        }
        return null;
    }
}

```



### 全局加响应头, 利用后置过滤器

```java
@Component
public class AddResponseHeaderFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SEND_RESPONSE_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletResponse response = currentContext.getResponse();
        response.setHeader("X-Foo", UUID.randomUUID().toString());
        return null;
    }
}
```

### 跨域

在被调用的类或方法上增加 @CrossOrigin 注解

在 Zuul 里增加 CorsFilter 过滤器

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();

        config.setAllowCredentials(true); // 是否允许 cookie 跨域
        config.setAllowedOrigins(Arrays.asList("*")); // 允许的域名
        config.setAllowedHeaders(Arrays.asList("*")); // 允许的头
        config.setAllowedMethods(Arrays.asList("*")); // 允许的头
        config.setMaxAge(300L);

        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }

}
```

## Spring Cloud Hystrix

Hystrix, 中文含义是豪猪, 因其背上长满棘刺, 从而拥有了自我保护的能力. 本文所说的Hystrix是Netflix开源的一款容错框架, 同样具有自我保护能力. 为了实现容错和自我保护, 下面我们看看Hystrix如何设计和实现的. 

Hystrix设计目标: 

-   对来自依赖的延迟和故障进行防护和控制——这些依赖通常都是通过网络访问的
-   阻止故障的连锁反应
-   快速失败并迅速恢复
-   回退并优雅降级
-   提供近实时的监控与告警

Hystrix遵循的设计原则: 

-   防止任何单独的依赖耗尽资源(线程)
-   过载立即切断并快速失败, 防止排队
-   尽可能提供回退以保护用户免受故障
-   使用隔离技术(例如隔板, 泳道和断路器模式)来限制任何一个依赖的影响
-   通过近实时的指标, 监控和告警, 确保故障被及时发现
-   通过动态修改配置属性, 确保故障及时恢复
-   防止整个依赖客户端执行失败, 而不仅仅是网络通信

Hystrix如何实现这些设计目标? 

-   包裹请求: 使用命令模式将所有对外部服务(或依赖关系)的调用包装在HystrixCommand或HystrixObservableCommand对象中, 并将该对象放在单独的线程中执行；
-   资源隔离: Hystrix为每个依赖都维护了⼀个⼩型的线程池(舱壁模式)(或者信号量). 如果该线程池已满,  发往该依赖的请求就被⽴即拒绝, ⽽不是排队等待, 从⽽加速失败判定
-   跳闸机制: 服务错误百分比超过了阈值, 熔断器开关自动打开, 一段时间内停止对该服务的所有请求. 
-   回退机制: 当请求失败、超时、被拒绝, 或当断路器打开时, 执⾏降级逻辑. 降级逻辑由开发⼈员⾃⾏提供, 例如返回⼀个缺省值
-   监控: Hystrix可以近乎实时地监控运⾏指标和配置的变化, 例如成功、失败、超时、以及被拒绝的请求等



### Hystrix 处理流程

#### Hystrix 整个工作流

构造一个 HystrixCommand 或 HystrixObservableCommand 对象, 用于封装请求, 并在构造方法配置请求被执行需要的参数；

1.  执行命令, Hystrix提供了4种执行命令的方法, 后面详述；
2.  判断是否使用缓存响应请求, 若启用了缓存, 且缓存可用, 直接使用缓存响应请求. Hystrix支持请求缓存, 但需要用户自定义启动；
3.  判断熔断器是否打开, 如果打开, 跳到第8步；
4.  判断线程池/队列/信号量是否已满, 已满则跳到第8步；
5.  执行HystrixObservableCommand.construct()或HystrixCommand.run(), 如果执行失败或者超时, 跳到第8步；否则, 跳到第9步；
6.  统计熔断器监控指标；
7.  走Fallback备用逻辑
8.  返回请求响应

第 5 步线程池/队列/信号量已满时, 还会执行第 7 步逻辑, 更新熔断器统计信息, 而第 6 步无论成功与否, 都会更新熔断器统计信息. 



#### 执行命令的几种方法

Hystrix提供了4种执行命令的方法, execute()和queue()适用于 HystrixCommand 对象, 而observer()和toObservable()适用于 HystrixObservableCommand对象. 

1.  execute()

    以同步阻塞方法执行run(), 只支持接收一个值对象.  Hystrix会从线程池中取一个线程来执行run(), 并等待返回值. 

2.  queue()

    以异步非阻塞方法执行run(), 只支持接收一个值对象. 调用queue()就直接返回一个Future对象. 可通过Future.get()拿到run()的返回结果, 但 Future.get() 是阻塞执行的. 若执行成功,  Future.get() 返回单个返回值. 当执行失败时, 如果没有重写fallback,  Future.get() 抛出异常. 

3.  observe()

    事件注册前执行run()/construct(), 支持接收多个值对象, 取决于发射源. 调用observe()会返回一个hot Observable, 也就是说, 调用 observe()自动触发执行run()/construct(), 无论是否存在订阅者. 

    如果继承的是HystrixCommand, hystrix会从线程池中取一个线程以非阻塞方式执行run()；如果继承的是HystrixObservableCommand, 将以调用线程阻塞执行construct(). 

    **observe()使用方法:**

    1.  调用 observe()会返回一个Observable对象
    2.  调用这个 Observable对象的subscribe()方法完成事件注册, 从而获取结果

4.  toObservable()

    事件注册后执行run()/construct(), 支持接收多个值对象, 取决于发射源. 调用 toObservable() 会返回一个cold Observable, 也就是说, 调用 toObservable() 不会立即触发执行run()/construct(), 必须有订阅者订阅 Observable 时才会执行. 

    如果继承的是 HystrixComman, hystrix会从线程池中取一个线程以非阻塞方式执行run(), 调用线程不必等待run()；如果继承的是 HystrixObservableCommand , 将以调用线程堵塞执行construct(), 调用线程需等待construct()执行完才能继续往下走. 

    **toObservable()使用方法:**

    1.  调用observe()会返回一个Observable对象
    2.  调用这个 Observable对象的subscribe()方法完成事件注册, 从而获取结果

    需注意的是,  HystrixCommand也支持 toObservable()和observe(),  但是即使将 HystrixCommand 转换成Observable, 它也只能发射一个值对象. 只有 HystrixObservableCommand才支持发射多个值对象. 

#### 几种方法的关系

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_14.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_14.png" style="zoom: 50%;" />

-   execute()实际是调用了queue().get()
-   queue()实际调用了toObservable().toBlocking().toFuture()
-   observe()实际调用toObservable()获得一个cold Observable, 再创建一个ReplaySubject对象订阅Observable, 将源Observable转化为hot Observable. 因此调用observe()会自动触发执行run()/construct(). 

Hystrix 总是以Observable的形式作为相应返回, 不同执行命令的方法只是进行了相应的转换. 



### 依赖隔离

资源隔离主要指对线程的隔离.  Hystrix提供了两种线程隔离的方式: 线程池和信号量. 

#### 线程池隔离

所谓线程池隔离, 就是每个过来的请求, 系统会单独将这个请求的执行放在一个线程或线程池里, 这个根据自己的需求进行配置, 以后, 相同的请求不管有多少打进来, 都会在指定数量的线程池内进行, 因此不会出现诸如线程资源耗尽的情况, 这种隔离方式在某些场景下是非常有用的, 下面来看具体的代码

Hystrix还通过命令模式对发送请求的对象和执行请求的对象进行解耦, 将不同类型的业务请求封装为对应的命令请求. 如订单服务查询商品, 查询商品请求->商品command；商品服务查询库存, 查询库存请求->库存command. 并且为每个类型的command配置一个线程池, 当第一次创建command时, 根据配置创建一个线程池, 并放入ConcurrentHashMap, 如商品command: 

```java
final static ConcurrentHashMap<String, HystrixThreadPool> threadPools = new ConcurrentHashMap<String, HystrixThreadPool>();...if (!threadPools.containsKey(key)) {    threadPools.put(key, new HystrixThreadPoolDefault(threadPoolKey, propertiesBuilder));}
```

后续查询商品的请求创建command时, 将会重用已创建的线程池. 线程池隔离之后的服务依赖关系: 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_15.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_15.png" style="zoom: 50%;" />

通过发送请求线程与执行请求的线程分离, 可有效防止发生级联故障. 当线程池或请求队列饱和时, Hystrix将拒绝服务, 使得请求线程可以快速失败, 从而避免依赖问题扩散. 

**线程池隔离优点:**

-   保护应用程序以免受来自依赖故障的影响, 指定依赖线程池饱和不会影响应用程序的其余部分. 
-   当引入新客户端lib时, 即使发生问题, 也是在lib中, 并不会影响其他内容. 
-   当依赖从故障恢复正常时, 应用程序会立即恢复正常的性能. 
-   当应用程序一些配置参数错误时, 线程池的运行状况会很快检测到这一点(通过增加错误、延迟、超时、拒绝等), 同时可以通过动态属性进行实时纠正错误的参数配置. 
-   如果服务的性能有变化, 需要实时调整, 比如增加或减少超时时间, 更改重试次数, 可以通过线程池指标状态属性修改, 而且不会影响到其它调用请求. 
-   除了隔离优势外,  Hystrix 拥有专门的线程可提供内置的并发功能, 使得可以在同步调用之上构建异步门面(外观模式), 为异步编程提供了支持( Hystrix 引入了R小Java异步框架). 

**注意:**尽管线程池提供了线程隔离, 我们的客户端底层代码也必须要有超时设置或响应线程中断, 不能无限制的阻塞以致线程池一直饱和. 

**缺点:**

*   线程池的主要缺点是增加了计算开销. 每个命令的执行都在单独的线程完成, 增加了排队、调度和上下文切换的开销. 因此, 要使用 Hystrix , 就必须接受它带来的开销, 以换取它所提供的的好处. 

*   通常情况下, 线程池引入的开销足够小, 不会有重大的成本和性能影响. 但对于一些访问延迟极低的服务, 如只依赖内存缓存, 线程池引入的开销就比较明显了, 这时候使用线程池隔离技术就不合适了, 我们需要考虑更轻量级的方式, 如信号量隔离. 



#### 信号量隔离

上面提到了线程池隔离的缺点, 当依赖延迟极低的服务时, 线程池隔离技术引入的开销超过了它所带来的好处. 这时候可以使用信号量隔离技术来代替, 通过设置信号量来限制对任何给定依赖的并发调用量. 下图说明了线程池隔离和信号量隔离的主要区别: 

信号量的资源隔离只是起到一个开关的作用, 比如, 服务 A 的信号量大小为 10, 那么就是说它同时只允许有 10 个 tomcat 线程来访问服务 A, 其它的请求都会被拒绝, 从而达到资源隔离和限流保护的作用.

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_16.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_16.png" style="zoom: 50%;" />

使用线程池时, 发送请求的线程和执行依赖服务的线程不是同一个, 而使用信号量时, 发送请求的线程和执行依赖服务的线程时同一个,  都是发起请求的线程. 

由于Hystrix默认使用线程池做线程隔离，使用信号量隔离需要显示地将属性execution.isolation.strategy设置为ExecutionIsolationStrategy.SEMAPHORE，同时配置信号量个数，默认为10。客户端需向依赖服务发起请求时，首先要获取一个信号量才能真正发起调用，由于信号量的数量有限，当并发请求量超过信号量个数时，后续的请求都会直接拒绝，进入fallback流程。

信号量隔离主要是通过控制并发请求量，防止请求线程大面积阻塞，从而达到限流和防止雪崩的目的。

#### 线程隔离总结

线程池和信号量都可以做线程隔离, 但各有各的优缺点和支持的场景, 对比如下: 

线程切换支持异步支持超时支持熔断限流开销信号量否否否是是小线程池是是是是是大

线程池和信号量都支持熔断和限流. 相比线程池, 信号量不需要线程切换, 因此避免了不必要的开销. 但是信号量不支持异步, 也不支持超时, 也就是说当所请求的服务不可用时, 信号量会控制超过限制的请求立即返回, 但是已经持有信号量的线程只能等待服务响应或从超时中返回, 即可能出现长时间等待. 线程池模式下, 当超过指定时间未响应的服务,  Hystrix会通过响应中断的方式通知线程立即结束并返回. 

线程池和信号量都可以做线程隔离, 但各有各的优缺点和支持的场景, 对比如下: 

|        | 线程切换 | 支持异步 | 支持超时 | 支持熔断 | 限流 | 开销 |
| ------ | -------- | -------- | -------- | -------- | ---- | ---- |
| 信号量 | 否       | 否       | 否       | 是       | 是   | 小   |
| 线程池 | 是       | 是       | 是       | 是       | 是   | 大   |

线程池和信号量都支持熔断和限流。相比线程池, 信号量不需要线程切换, 因此避免了不必要的开销。但是信号量不支持异步, 也不支持超时, 也就是说当所请求的服务不可用时, 信号量会控制超过限制的请求立即返回, 但是已经持有信号量的线程只能等待服务响应或从超时中返回, 即可能出现长时间等待。线程池模式下, 当超过指定时间未响应的服务, Hystrix会通过响应中断的方式通知线程立即结束并返回。



### 服务降级

**优先核心服务可用, 非核心服务不可用或若可用**, 通过 HystrixCommand 注解指定, fallbackMethod(回退函数)中具体实现降级逻辑

降级, 通常指事务高峰期, 为了保证核心服务正常运行, 需要停掉一些不太重要的业务, 或者某些服务不可用时, 执行备用逻辑从故障服务中快速失败或快速返回, 以保障主体业务不受影响.  Hystrix提供的降级主要是为了容错, 保证当前服务不受依赖服务故障的影响, 从而提高服务的健壮性. 要支持回退或降级处理, 可以重写 HystrixCommand的getFallBack方法或HystrixObservableCommand的resumeWithFallback方法. 



#### Hystrix 在以下几种情况下会走降级逻辑

执行construct()或run()抛出异常

熔断器打开导致命令短路

命令的线程池和队列或信号量的容量超额, 命令被拒绝

命令执行超时

#### 降级回退方式

1.  Fail Fast 快速失败

    快速失败是最普通的命令执行方法, 命令没有重写降级逻辑.  如果命令执行发生任何类型的故障, 它将直接抛出异常. 

2.  Fail Fast 无声失败

    指在降级方法中通过返回 null, 空 Map, 空 List 或其他类似的响应来完成. 

    ```java
    @Override
    protected Integer getFallback() {
       return null;
    }
     
    @Override
    protected List<Integer> getFallback() {
       return Collections.emptyList();
    }
     
    @Override
    protected Observable<Integer> resumeWithFallback() {
       return Observable.empty();
    }
    ```

3.  FallBack: Static

    指在降级方法中返回静态默认值. 这不会导致服务以“无声失败”的方式被删除, 而是导致默认行为发生. 如: 应用根据命令执行返回 true / false 执行相应逻辑, 但命令执行失败, 则默认为 true. 

    ```java
    @Override
    protected Boolean getFallback() {
        return true;
    }
    @Override
    protected Observable<Boolean> resumeWithFallback() {
        return Observable.just( true );
    }
    ```

4.  FallBack: Stubbed

    当命令返回一个包含多个字段的复合对象时, 适合以 Stubbed 的方式回退. 

    ```java
    @Override
    protected MissionInfo getFallback() {
       return new MissionInfo("missionName","error");
    }
    ```

5.  FallBack: Cache via Network

    有时, 如果调用依赖服务失败, 可以从缓存服务(如redis)中查询旧数据版本. 由于又会发起远程调用, 所以建议重新封装一个 Command, 使用不同的ThreadPoolKey, 与主线程池进行隔离. 

    ```java
    @Override
    protected Integer getFallback() {
       return new RedisServiceCommand(redisService).execute();
    }
    ```

6.  Primary+Secondary with FallBack

    有时系统具有两种行为- 主要和次要, 或主要和故障转移. 主要和次要逻辑涉及到不同的网络调用和业务逻辑, 所以需要将主次逻辑封装在不同的Command 中, 使用线程池进行隔离. 为了实现主从逻辑切换, 可以将主次 command 封装在外观 HystrixCommand 的 run 方法中, 并结合配置中心设置的开关切换主从逻辑. 由于主次逻辑都是经过线程池隔离的 HystrixCommand, 因此外观 HystrixCommand 可以使用信号量隔离, 而没有必要使用线程池隔离引入不必要的开销. 原理图如下: 

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_18.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_18.png" style="zoom: 50%;" />

主次模型的使用场景还是很多的. 如当系统升级新功能时, 如果新版本的功能出现问题, 通过开关控制降级调用旧版本的功能. 

```java
public class CommandFacadeWithPrimarySecondary extends HystrixCommand<String> {
 
    private final static DynamicBooleanProperty usePrimary = DynamicPropertyFactory.getInstance().getBooleanProperty("primarySecondary.usePrimary", true);
 
    private final int id;
 
    public CommandFacadeWithPrimarySecondary(int id) {
        super(Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("PrimarySecondaryCommand"))
                .andCommandPropertiesDefaults(
                        // 由于主次command已经使用线程池隔离，Facade Command使用信号量隔离即可
                        HystrixCommandProperties.Setter()
                                .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }
 
    @Override
    protected String run() {
        if (usePrimary.get()) {
            return new PrimaryCommand(id).execute();
        } else {
            return new SecondaryCommand(id).execute();
        }
    }
 
    @Override
    protected String getFallback() {
        return "static-fallback-" + id;
    }
 
    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }
 
    private static class PrimaryCommand extends HystrixCommand<String> {
 
        private final int id;
 
        private PrimaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("PrimaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("PrimaryCommand"))
                    .andCommandPropertiesDefaults(                          HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(600)));
            this.id = id;
        }
 
        @Override
        protected String run() {
            return "responseFromPrimary-" + id;
        }
 
    }
 
    private static class SecondaryCommand extends HystrixCommand<String> {
 
        private final int id;
 
        private SecondaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("SecondaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("SecondaryCommand"))
                    .andCommandPropertiesDefaults(  HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(100)));
            this.id = id;
        }
 
        @Override
        protected String run() {
            return "responseFromSecondary-" + id;
        }
 
    }
 
    public static class UnitTest {
 
        @Test
        public void testPrimary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", true);
                assertEquals("responseFromPrimary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }
 
        @Test
        public void testSecondary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", false);
                assertEquals("responseFromSecondary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }
    }
}
```

通常情况下, 建议重写 getFallBack 或 resumeWithFallback 提供自己的备用逻辑, 但不建议在回退逻辑中执行任何可能失败的操作. 



### 服务熔断

现实生活中, 可能大家都有注意到家庭电路中通常会安装一个保险盒, 当负载过载时, 保险盒中的保险丝会自动熔断, 以保护电路及家里的各种电器, 这就是熔断器的一个常见例子. Hystrix中的熔断器(Circuit Breaker)也是起类似作用, Hystrix在运行过程中会向每个commandKey对应的熔断器报告成功、失败、超时和拒绝的状态, 熔断器维护并统计这些数据, 并根据这些统计信息来决策熔断开关是否打开. 如果打开, 熔断后续请求, 快速返回. 隔一段时间(默认是5s)之后熔断器尝试半开, 放入一部分流量请求进来, 相当于对依赖服务进行一次健康检查, 如果请求成功, 熔断器关闭. 

熔断器配置, Circuit Breaker主要包括如下6个参数: 

1.  circuitBreaker.enabled

    是否启用熔断器, 默认是TRUE. 

2.  circuitBreaker.forceOpen

    熔断器强制打开, 始终保持打开状态, 不关注熔断开关的实际状态. 默认值FLASE. 

3.  circuitBreaker.forceClosed

    熔断器强制关闭, 始终保持关闭状态, 不关注熔断开关的实际状态. 默认值FLASE. 

4.  circuitBreaker.errorThresholdPercentage

    错误率, 默认值50%, 例如一段时间(10s)内有100个请求, 其中有54个超时或者异常, 那么这段时间内的错误率是54%, 大于了默认值50%, 这种情况下会触发熔断器打开. 

5.  circuitBreaker.requestVolumeThreshold

    默认值20. 含义是一段时间内至少有20个请求才进行errorThresholdPercentage计算. 比如一段时间了有19个请求, 且这些请求全部失败了, 错误率是100%, 但熔断器不会打开, 总请求数不满足20. 

6.  circuitBreaker.sleepWindowInMilliseconds

    半开状态试探睡眠时间, 默认值5000ms. 如: 当熔断器开启5000ms之后, 会尝试放过去一部分流量进行试探, 确定依赖服务是否恢复. 



#### 熔断器工作原理

下图展示了 HystrixCircuitBreaker 的工作原理

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_17.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_17.png" style="zoom: 50%;" />

**熔断器工作的详细过程如下:**

**第一步, 调用 allowRequest() 判断是否允许将请求提交到线程池**

1.  允许熔断器强制打开,  circuitBreaker.forceOpen为true, 不允许放行, 返回. 
2.  如果熔断器强制关闭,  circuitBreaker.forceOpen为true, 允许放行.  此外不必关注熔断器实际状态, 也就是说熔断器仍然会维护统计数据和开关状态, 只是不生效而已. 

**第二步, 调用 isOpen() 判断熔断器开关是否打开**

1.  如果熔断器开关打开, 进入第三步, 否则继续；
2.  如果一个周期内总的请求数小于circuitBreaker.requestVolumeThreshold的值, 允许请求放行, 否则继续；
3.  如果一个周期内错误率小于circuitBreaker.errorThresholdPercentage的值, 允许请求放行. 否则, 打开熔断器开关, 进入第三步. 

**第三步,  调用 allowSingleTest() 判断是否允许单个请求通行, 检查依赖服务是否恢复**

如果熔断器打开, 且距离熔断器打开的时间或上一次试探请求放行的时间超过
 circuitBreaker.sleepWindowInMilliseconds的值时, 熔断器器进入半开状态, 允许放行一个试探请求；否则, 不允许放行. 

此外, 为了提供决策依据, 每个熔断默认维护了 10 个 bucket, 每秒一个bucket, 当心的 bucket 被创建时, 最旧的bucket会被抛弃. 其中每个bucket维护了请求、失败、超时、拒绝的计数器, Hystrix 负责收集并统计这些计数器. 





当某个服务发生降级数量达到一定的百分比, 那么正常的逻辑也会直接触发降级, 将整个服务熔断, 一定时间后再恢复访问, 在 SpringCloud 中的熔断就是配置 4 个属性

<img src="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_7.png" alt="https://miaomiaoqi.github.io/images/springcloud/springcloud_sell_7.png" style="zoom: 50%;" />

**Closed:** 默认熔断器是关闭的, 当失败次数达到一定阈值, 会变为打开状态

**Open:** 此时所有的请求都会触发降级, 一定时间后, 会变为半打开状态

**Half Open:** 此时会释放一定的请求, 当请求成功达到一定比例, 会恢复为 Closed 状态

```java
// 熔断
@HystrixCommand(commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 设置熔断
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 默认值20.意思是至少有20个请求才进行 errorThresholdPercentage 错误百分比计算. 
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), // 半开试探休眠时间, 默认值5000ms. 当熔断器开启一段时间之后比如5000ms, 会尝试放过去一部分流量进行试探, 确定依赖服务是否恢复. 
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60") // 设定错误百分比, 默认值50%, 例如一段时间(10s)内有100个请求, 其中有55个超时或者异常返回了, 那么这段时间内的错误百分比是55%, 大于了默认值50%, 这种情况下触发熔断器-打开.  
})
@GetMapping("/getProductInfoList")
public String getProductInfoList(@RequestParam("number") Integer number) {
    if (number % 2 == 0) {
        return "success";
    }
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate.postForObject("http://localhost:9916/product/listForOrder",
            Arrays.asList("157875196366160022"),
            String.class);
    // throw new RuntimeException("发送异常了"); 抛异常就会触发降级
}
```

http://localhost:9926/getProductInfoList?number=1 会触发降级

http://localhost:9926/getProductInfoList?number=2 会正常访问

**当降级请求达到 60%的比例后, 正常访问的接口, 也会直接降级整个服务熔断 , 10s 后恢复一定量访问**



### 使用配置

加入 hystrix 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>LATEST</version>
</dependency>
```

加入注解

```java
@EnableFeignClients
// @SpringBootApplication
// @EnableDiscoveryClient 开启 eureka
// @EnableCircuitBreaker 开启 hystrix
@SpringCloudApplication // 包含上边 3 个注解
public class SpringcloudSellOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellOrderApplication.class, args);
    }

}
```

请求方法加入 @HysttixCommand 注解

```java
@RestController
// @DefaultProperties(defaultFallback = "defaultFallback")
public class HystrixController {

    // @HystrixCommand 配合 @DefaultProperties 会触发默认降级方法
    @HystrixCommand(fallbackMethod = "fallback", 
		commandProperties = @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000"))     // 会指定特殊的降级方法, 优先级高于默认降级, 默认超时 1s, 这个配置会改为 3s
    @GetMapping("/getProductInfoList")
    public String getProductInfoList() {
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.postForObject("http://localhost:9916/product/listForOrder",
                Arrays.asList("157875196366160022"),
                String.class);
        // throw new RuntimeException("发送异常了"); 抛异常就会触发降级
    }

    private String fallback() {
        return "太拥挤了, 请稍后再试~";
    }

    private String defaultFallback() {
        return "默认提示: 太拥挤了, 请稍后再试~";
    }

}
```

关闭 product 服务, 访问 http://localhost:9926/getProductInfoList, 页面返回太拥挤了, 请稍后再试~

也可以采用配置文件的方式进行配置, 但是方法上一定要配置 @HystrixCommand 注解

```yaml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000
    getProductInfoList: # 单独为某一方法设置超时时间
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000
```



#### Feign-Hystrix

feign 整合 hystrix 进行降级, feign 已经自动依赖了 hystrix 包

配置 feign 的配置文件

```yaml
feign:
  hystrix:
    enabled: true
```

修改 feign 接口

```java
@FeignClient(name = "SPRINGCLOUD-SELL-PRODUCT", fallback = ProductClient.ProductClientFallback.class)
public interface ProductClient {

    @PostMapping("/product/listForOrder")
    List<ProductInfoOutput> listForOrder(List<String> productIdList);

    @PostMapping("/product/decreaseStock")
    void decreaseStock(@RequestBody List<DecreaseStockInput> cartDTOList);

    @Component
    class ProductClientFallback implements ProductClient {

        @Override
        public List<ProductInfoOutput> listForOrder(List<String> productIdList) {
            return null;
        }

        @Override
        public void decreaseStock(List<DecreaseStockInput> cartDTOList) {

        }
    }
    
}
```



#### 可视化 hystrix 工具 Hystrix-Dashboard

加入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    <version>LATEST</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

启动类加入注解

```java
@EnableFeignClients
// @SpringBootApplication
// @EnableDiscoveryClient 开启 eureka
// @EnableCircuitBreaker 开启 hystrix
@EnableHystrixDashboard
@SpringCloudApplication
public class SpringcloudSellOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellOrderApplication.class, args);
    }

}
```

配置文件配置去除访问前缀

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
      base-path: /
```

访问图形化界面

http://localhost:9926/hystrix 填入 http://localhost:9926/hystrix.stream





### Zuul 超时设置(降级)

Zuul 使用 ribbon 负载均衡组件, 所以 zuul 的超时配置时配置 ribbon 的超时时间, 同时也可以指定 hystrix 超时配置, 两者可以同时存在, 哪个时间小就先触发哪个

```yaml
zuul:
  routes:
    springcloud-sell-product: # 定义一个路由规则, 规则名字可以任意起
      path: /myProduct/** # 路由匹配的路径
      serviceId: SPRINGCLOUD-SELL-PRODUCT # 转发到的服务
      stripPrefix: true # 是否去除前缀
      sensitiveHeaders:
    springcloud-sell-order:
      path: /api/order/**
      serviceid: SPRINGCLOUD-SELL-ORDER
      stripPrefix: true # 是否去除前缀
      sensitiveHeaders:
  host: # 如果路由方式是 url 的方式, 那么该超时配置生效
    connect-timeout-millis: 100 # HTTP连接超时
    socket-timeout-millis: 100  # socket超时

ribbon: # 如果路由方式是 serviceId 的方式, 那么该全局超时配置生效
  eureka:
    enabled: true
  ReadTimeout: 1000
  ConnectTimeout: 1000
  MaxAutoRetries: 0
  MaxAutoRetriesNextServer: 1
  OkToRetryOnAllOperations: false

SPRINGCLOUD-SELL-ORDER: # 指定服务超时设置, 优先级高于全局
  ribbon:
    eureka:
      enabled: true
    ReadTimeout: 3000
    ConnectTimeout: 3000
    MaxAutoRetries: 0
    MaxAutoRetriesNextServer: 1
    OkToRetryOnAllOperations: false

SPRINGCLOUD-SELL-PRODUCT:
  ribbon: 
    eureka:
      enabled: true
    ReadTimeout: 3000
    ConnectTimeout: 3000
    MaxAutoRetries: 0
    MaxAutoRetriesNextServer: 1
    OkToRetryOnAllOperations: false

# 因为懒加载, 所以设置默认超时加大一些, 防止第一次请求失败
# ribbon 超时与 hystrix 超时可以并存, 哪个超时时间小, 哪个生效
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 500
    SPRINGCLOUD-SELL-PRODUCT: # 指定特定服务或方法的超时时间
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000
```

超时 fallback 设置, 实现 FallbackProvider 接口

```java
@Component
public class GatewayFallback implements FallbackProvider {

	// 指定哪些服务支持 fallback
    @Override
    public String getRoute() {
        // api服务id, 如果需要所有调用都支持回退, 则return "*"或return null
        // return "api-user-server";
        return "*";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
						// 响应体
            @Override
            public InputStream getBody() throws IOException {
                Map<String, String> result = new HashMap<>();
                result.put("state", "9999");
                result.put("msg", "系统错误, 请求失败");
                return new ByteArrayInputStream(JsonUtil.toJson(result).getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                // 和body中的内容编码一致, 否则容易乱码
                headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
                return headers;
            }

            /**
             * 网关向api服务请求是失败了, 但是消费者客户端向网关发起的请求是OK的, 
             * 不应该把api的404,500等问题抛给客户端
             * 网关和api服务集群对于客户端来说是黑盒子
             */
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.getReasonPhrase();
            }

            @Override
            public void close() {
            }
        };
    }
}
```



## 链路监控 Spring Cloud Sleuth





## SpringCloud 多版本选择

英文命名方式也比较有意思, Spring Cloud 采用了英国伦敦地铁站的名称来命名, 并由地铁站名称字母A-Z依次类推的形式来发布迭代版本. 

由上可知, Spring Cloud 的第一个版本 "Angel" 就不觉得奇怪了, 接着 "Brixton" 就是第二个版本. 当一个项目到达发布临界点或者解决了一个严重的 BUG 后就会发布一个 "Service Release" 版本,  简称 SR(X) 版本, x 代表一个递增数字. 

**由此我们可以得出 "Finchley M9" 就是目前最新的开发版本, "Edgware SR3" 是最新稳定版本**

| Release Train | Boot Version |
| :------------ | :----------- |
| Hoxton        | 2.2.x        |
| Greenwich     | 2.1.x        |
| Finchley      | 2.0.x        |
| Edgware       | 1.5.x        |
| Dalston       | 1.5.x        |



## 服务下线

### 直接关闭

如果直接 KILL SpringCloud 的服务, 因为 Eureka 采用心跳的机制来上下线服务, 会导致服务消费者调用此已经 kill 的服务提供者然后出错

最粗暴. 写这个是因为, 直接关闭, 如果 Eureka 开了保护模式, 会导至服务已关闭, 但是未下线, 还是会重试调用. 如果不需强稳定性的话可以这么干. 

### 利用 Spring Boot Actuato 的管理端点

pom 中引用 Actuato

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

properties中添加如下内容

```properties
# 启用shutdown
endpoints.shutdown.enabled=true
# 禁用密码验证
endpoints.shutdown.sensitive=false
# 禁用actuator管理端鉴权
management.security.enabled=false
# 开启重启支持
endpoints.restart.enabled=true
#(只允许本机访问)
server.address=localhost
```

在服务器上用 curl 发送 post 请求到 pause

```bash
curl -X POST http://localhost:8080/pause
```

此时 eurake 上该服务被标记问下线, 但该服务其实还是可以正常访问的, 当 client 还未及时更新本地 Instances 缓存时, 依然不会中断服务. 当所有 client 都感知到该服务 DOWN 后就不会再往该服务发请求了. 

在服务器上利用 curl 发送 shutdown 命令

```bash
curl -X POST http://localhost:8080/shutdown

或者

curl -d "" http://localhost:8080/shutdown
```

