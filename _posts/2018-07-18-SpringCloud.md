---
layout: post
title:  "SpringCloud学习"
date:   2018-07-18 14:36:38
categories: Framework
tags: Spring
author: miaoqi
---

* content
{:toc}
# 微服务

一系列微小的服务组成

跑在自己的进程中

每个服务为独立的业务

独立部署

分布式的管理

## 不适合微服务的场景

**系统中包含很多强事务场景的不适合做微服务**

**业务相对稳定, 迭代周期长(稳定项目)**

**访问压力不大, 可用性要求不高(后台 OA)**

**...**

## 如何拆分功能

单一职责, 松耦合(不同服务尽量不影响), 高内聚(所有行为放到一个服务)

关注点分离

## 服务和数据的关系

先考虑业务功能, 在考虑数据

无状态服务

# 注册中心Spring Cloud Eureka

@EnableEurekaServer, @EnableEurekaClient, @EnableDiscoveryClient

基于Netflix Eureka做了二次封装

两部分组成

* Eureka Server 注册中心
* Eureka Client 服务注册

Eureka遵守AP

* Eureka各个节点是平等的, 几个节点挂掉不会影响正常的工作, 剩余的节点依然可以提供注册和查询服务. Eureka的客户端再向某个Eureka注册时如果发现连接失败, 则会自动切换至其他节点, 只要有一台Eureka还在, 就可以保证服务可用, 只不过查询到的信息可能不是最新的. Eureka还用一种自我保护机制, 如果在15分钟内超过85%的节点没有正常心跳, 那么Eureka就认为客户端与注册中心出现网络故障, 会出现以下几种情况: 

    1. Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务
2. Eureka仍然能够接受新服务注册和查询请求, 但是不会同步到其他节点上
    3. 当网络稳定时, 当前实例新的注册信息会同步到其他节点中

## Spring Cloud Eureka Server

启动类加入 `@EnableEurekaServer` 注解

```
@SpringBootApplication
@EnableEurekaServer
public class SpringcloudSellEurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellEurekaApplication.class, args);
    }
}
```

SpringCloudEurekaServer既是服务端又是客户端, 启动时会默认查找 defaultZone 注册, 也会注册自己, 采用心跳的方式每隔一段时间注册一次, 可以改变配置不让自己注册

```
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

```
eureka:
  server:
    enable-self-preservation: false # 是否开启自我保护机制
```

查看注册信息

```
http://127.0.0.1:9901/eureka/apps
```



## Spring Cloud Eureka Client

启动类加入 `@EnableDiscoveryClient` 注解

```
@SpringBootApplication
@EnableDiscoveryClient
public class SpringcloudSellClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellClientApplication.class, args);
    }
}
```

## 高可用

拷贝 EurekaServer 项目, 修改端口号, 修改服务注册地址为其他 EurekaServer 的地址

```
server:
  port: 9901
eureka:
  client:
    service-url:
      defaultZone: http://eureka9902:9902/eureka/,http://eureka9903:9903/eureka/ # 寻找注册中心地址
```

EurekaClient 的地址填写集群的每一个地址

```
server:
  port: 9910
eureka:
  client:
    service-url:
      # 寻找注册中心地址
      defaultZone: http://eureka9901:9901/eureka/,http://eureka9902:9902/eureka/,http://eureka9903:9903/eureka/
```

## 服务发现的两种方式

### 客户端发现

客户端获取所有服务端地址, 需要自己实现负载均衡逻辑去调用, SpringCloud 就采用这种方式

Eureka: http://eureka9901:9901/eureka/apps 可以查看注册信息

### 服务端发现

需要代理服务的介入, 对客户端是完全透明的

Nginx

Zookeeper

Kubernetes

# 应用间通信 RestTemplate 和 Feign

## RestTemplate(面向服务)

基于 Netflix Ribbon 实现的一套 http 客户端负载均衡工具, Ribbon + RestTemplate, 结合 eureka 使用, 会从 eureka 中查找可用的机器进行访问

```
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

```
@Component
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

### Ribbon组件

Netflix Ribbon 是客户端负载均衡器, 是 LoadBalance 实现负载均衡的组件, 可以实现**服务发现, 服务选择规则, 服务监听**, **RestTemplate, Feign, Zuul 均使用该组件**

* ServerList: 获取所有可用列表
* IRule: 根据规则获取一个地址
* ServerListFilter: 过滤掉一部分服务地址

## Feign(面向接口)

声明式 REST 客户端(伪RPC), 采用了基于接口的注解 @FeignClient, 内部也使用了 Ribbon 做负载均衡

加入 Feign 依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

启动类加入 @EnableFeignClients 注解

```
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

```
@FeignClient(name = "SPRINGCLOUD-SELL-PRODUCT")
public interface ProductClient {

    @GetMapping("/msg")
    String productMsg();

}
```

# 分布式统一配置中心 Config

![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_1.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_1.png)

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

## ConfigServer

加入 ConfigServer 依赖, config 本身也是一个微服务, 需要注册到 eureka 中

```
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

开启服务端配置中心

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableConfigServer
public class SpringcloudSellConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudSellConfigApplication.class, args);
    }

}
```

编写配置文件

```
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

访问地址举例:

http://localhost:9936/release/springcloud-sell-order-test.yml



## ConfigClient

加入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

**修改配置文件 bootstrap.yml, bootstrap.yml 加载优先级最高, 所以需要先加载 config server 的配置文件**

```
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

* **如果配置 spring.cloud.config.uri 就不需要指定 eureka 与 discovery 了, 该配置会直接从 config server 拉取配置. 如果配置 discovery 就需要配置 eureka, 要从注册中心找到 config server 服务拉取配置, uri 与 discovery 二者选一**
* **上述配置会从 config server 服务拉取 label 指定的分支(默认为 master 分支) 文件名为 springcloud-sell-order-dev.yml(spring.application.name-spring.cloud.config.profie.yml) 的配置文件, 在拉取该配置文件前, 还会拉取 application.yml(所有项目通用配置文件) 与 springcloud-sell-order.yml(同一项目不同 profile 的通用配置文件)配置文件, 三个配置文件的内容会合到一起, application.yml > springcloud-sell-order.yml > springcloud-sell-order-dev.yml**



# 自动刷新配置 Spring Cloud Bus

![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_2.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_2.png)

# Hystrix断路器

* 服务熔断(服务端)

    * Hystrix是一个用于处理分布式系统的延迟和容错的开源库, Hystrix能够保证在一个依赖出问题的情况下, 不会导致整个服务失败, 避免级联故障, 以提高分布式系统的弹性
    
    * 断路器本身是一种开关装置, 当某个服务单元发生故障之后, 通过断路器的故障监控(类似熔断保险丝), 向调用方返回一个符合预期的, 可处理的备选响应(FallBack), 而不是长时间等待或者抛出调用无法处理的异常

* 服务降级(客户端)

    * 整体资源不够了, 先关闭一些服务, 待资源充足了, 再将服务打开

# Zuul路由网关

* 代理

* 路由

* 过滤

  