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


## SpringCloud

* 是多种技术的集合, 提倡将单一的应用拆分成一组小的服务, 每个服务运行在其独立的进程中

* Dubbo和SpringCloud的区别

    ||Dubbo|SpringCloud|
    |-----|-----|-----|
    |通信方式|RPC|HTTP|

### Eureka和Zookeeper

* Eureka遵守AP

    * Eureka各个节点是平等的, 几个节点挂掉不会影响正常的工作, 剩余的节点依然可以提供注册和查询服务. Eureka的客户端再向某个Eureka注册时如果发现连接失败, 则会自动切换至其他节点, 只要有一台Eureka还在, 就可以保证服务可用, 只不过查询到的信息可能不是最新的. Eureka还用一种自我保护机制, 如果在15分钟内超过85%的节点没有正常心跳, 那么Eureka就认为客户端与注册中心出现网络故障, 会出现以下几种情况: 

        1. Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务

        2. Eureka仍然能够接受新服务注册和查询请求, 但是不会同步到其他节点上

        3. 当网络稳定时, 当前实例新的注册信息会同步到其他节点中

* Zookeeper遵守CP

    * Zookeeper当master节点因为网络故障与其他节点失去联系时, 剩余节点会重新进行leader选举, 选举leader的时间为30~120s, 且选举期间整个Zookeeper集群是不可用的, 这就导致服务瘫痪了

### Ribbon负载均衡(面向服务)

* 基于Netflix Ribbon实现的一套客户端  负载均衡的工具, Ribbon + RestTemplate, 结合eureka使用, 会从eureka中查找可用的机器进行访问

### Feign负载均衡(面向接口)

* 通过接口 + 注解获取服务地址

* 只需要创建一个接口, 在上边添加注解即可

### Hystrix断路器

* 服务熔断(服务端)

    * Hystrix是一个用于处理分布式系统的延迟和容错的开源库, Hystrix能够保证在一个依赖出问题的情况下, 不会导致整个服务失败, 避免级联故障, 以提高分布式系统的弹性
    
    * 断路器本身是一种开关装置, 当某个服务单元发生故障之后, 通过断路器的故障监控(类似熔断保险丝), 向调用方返回一个符合预期的, 可处理的备选响应(FallBack), 而不是长时间等待或者抛出调用无法处理的异常

* 服务降级(客户端)

    * 整体资源不够了, 先关闭一些服务, 待资源充足了, 再将服务打开

### Zuul路由网关

* 代理

* 路由

* 过滤

### Config分布式配置中心

* 集中管理配置文件

* 不同环境不同配置, 动态化配置更新, 分环境部署比如dev/test/prod

* 运行期间动态调整配置, 不需要再每个服务部署的机器上编写配置文件, 服务会向配置中心统一拉取配置自己的信息

* 当配置发生变动时, 服务不需要重启即可感知到配置的变化并应用新的配置

* 将配置中心以REST接口的形式暴露

    /{application}/{profile}/{label}
    
    /{application}-{profile}.yml
    
    **/{label}/{application}-{profile}.yml** 常用
    
    /{application}-{profile}.properties
    
    /{label}/{application}-{profile}.properties

* 如果不指定label即分支, 默认是master分支
  
    