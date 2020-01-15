---
layout: post
title: "SpringCloud 学习"
categories: [Framework]
description:
keywords:
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

**Eureka各个节点是平等的, 几个节点挂掉不会影响正常的工作, 剩余的节点依然可以提供注册和查询服务. Eureka的客户端再向某个Eureka注册时如果发现连接失败, 则会自动切换至其他节点, 只要有一台Eureka还在, 就可以保证服务可用, 只不过查询到的信息可能不是最新的. Eureka还用一种自我保护机制, 如果在15分钟内超过85%的节点没有正常心跳, 那么Eureka就认为客户端与注册中心出现网络故障, 会出现以下几种情况:** 

1. Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务

2. Eureka仍然能够接受新服务注册和查询请求, 但是不会同步到其他节点上

3. 当网络稳定时, 当前实例新的注册信息会同步到其他节点中

## Spring Cloud Eureka Server

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

SpringCloudEurekaServer既是服务端又是客户端, 启动时会默认查找 defaultZone 注册, 也会注册自己, 采用心跳的方式每隔一段时间注册一次, 可以改变配置不让自己注册

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

## Spring Cloud Eureka Client

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

## 高可用

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

### Ribbon组件

Netflix Ribbon 是客户端负载均衡器, 是 LoadBalance 实现负载均衡的组件, 可以实现**服务发现, 服务选择规则, 服务监听**, **RestTemplate, Feign, Zuul 均使用该组件**

* ServerList: 获取所有可用列表
* IRule: 根据规则获取一个地址
* ServerListFilter: 过滤掉一部分服务地址

## Feign(面向接口)

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



## ConfigClient

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



# 自动刷新配置 Spring Cloud Bus

![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_2.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_2.png)

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

    ![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_3.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_3.png)

    

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

    ![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_4.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_4.png)

* 修改 git 仓库中的配置文件内容, 向 ConfigServer 发送一条 POST 请求

    curl -X POST "http://localhost:9936/actuator/bus-refresh", 发送该请求后可以看到 queue 中多了一条消息

* 查看 order 服务中获取到的配置文件内容已经发生了变化

* **目前为止我们实现了手动的自动刷新, 接下来要配置 git 服务器的 webhooks 实现更改配置后自动 push**

    ![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_5.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_5.png)

    **我使用的是 git 所以需要配置外网域名进行 push, 生产环境我们可以搭建 gitlab 在内网中使用更安全**

# 消息队列 AMQP

amqp 定义了一系列消息接口, 典型的实现是 rabbitmq, springcloud 默认使用的实现就是 rabbitmq

* 加入 amqp 依赖

    ```xml
    <dependencies>
    		<dependency>
        		<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>
    ```

* 修改 git 中的配置文件

    ```yaml
    spring:
      rabbitmq:
        addresses: 127.0.0.1:6672
        username: guest
        password: guest
        virtual-host: /springcloud-sell
        connection-timeout: 15000
    ```



# 路由网关Zuul

**Zuul 的核心是一系列的过滤器**, 过滤器配合路由就是 Zuul 的本质了, Zuul 是 Netflix 公司的产品, Zuul1.x 的内部是 servlet 阻塞模型, Zuul2.x 采用的是 netty 的非阻塞模型, 但是 Zuul2.x 没有整合进入 SpringCloud, SpringCloud 推出了自己的网关 SpringCloudGateway 需要结合 Webflux 使用

- 稳定性, 高可用
- 性能, 并发
- 安全性
- 扩展性

![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_6.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_6.png)

* 加入 Zuul 依赖

    ```xml
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    ```

* 启动类加入 @EnableZuulProxy 注解

    ```java
    @SpringBootApplication
    @EnableZuulProxy
    public class SpringcloudSellGatewayApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(SpringcloudSellGatewayApplication.class, args);
        }
    
    }
    ```

* 修改 git 中的配置文件

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

* 修改项目中 bootstrap.yml 文件

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

* 启动项目, 通过 zuul 服务访问 product 服务, 这里是默认路由

    http://localhost:9946/springcloud-sell-product/product/list

    springcloud-sell-product 就是 product 服务在 eureka 中的服务 id, gateway-zuul 会默认配置

## 自定义路由

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

## 传递 Cookie

路由规则中默认设置了敏感头, Zuul 会过滤掉这些值, 我们需要手动去除敏感头, 见上述配置

默认的敏感头

```java
private Set<String> sensitiveHeaders = new LinkedHashSet(Arrays.asList("Cookie", "Set-Cookie", "Authorization"));
```

## 动态更新配置

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

## 过滤器

- 前置(Pre): 限流, 鉴权, 参数校验调整
- 路由(Route)
- 后置(Post): 统计, 日志, 跨域

- 错误(Error)

## 鉴权

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

## 限流

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



## 全局加响应头, 利用后置过滤器

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

## 跨域

* 在被调用的类或方法上增加 @CrossOrigin 注解
* 在 Zuul 里增加 CorsFilter 过滤器

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

# Spring Cloud Hystrix

基于 Netflix 的 Hystrix 开发的防雪崩利器, 为服务提供一系列容错保护功能

## 服务降级

**优先核心服务可用, 非核心服务不可用或若可用**, 通过 HystrixCommand 注解指定, fallbackMethod(回退函数)中具体实现降级逻辑

* 加入 hystrix 依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
        <version>LATEST</version>
    </dependency>
    ```

* 加入注解

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

* 请求方法加入 @HysttixCommand 注解

    ```java
    @RestController
    // @DefaultProperties(defaultFallback = "defaultFallback")
    public class HystrixController {
    
    		// @HystrixCommand 配合 @DefaultProperties 会触发默认降级方法
        @HystrixCommand(fallbackMethod = "fallback", commandProperties = 
    				@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", 
    						value = "3000")) // 会指定特殊的降级方法, 优先级高于默认降级, 默认超时 1s, 这个配置会改为 3s
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

* 关闭 product 服务, 访问 http://localhost:9926/getProductInfoList, 页面返回太拥挤了, 请稍后再试~

* 也可以采用配置文件的方式进行配置, 但是方法上一定要配置 @HystrixCommand 注解

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

### Feign-Hystrix

feign 整合 hystrix 进行降级, feign 已经自动依赖了 hystrix 包

* 配置 feign 的配置文件

    ```yaml
    feign:
      hystrix:
        enabled: true
    ```

* 修改 feign 接口

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

### 可视化 hystrix 工具 Hystrix-Dashboard

* 加入依赖

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

* 启动类加入注解

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

* 配置文件配置去除访问前缀

    ```yaml
    management:
      endpoints:
        web:
          exposure:
            include: "*"
          base-path: /
    ```

* 访问图形化界面

    http://localhost:9926/hystrix 填入 http://localhost:9926/hystrix.stream



## 服务熔断

当某个服务发生降级数量达到一定的百分比, 那么正常的逻辑也会直接触发降级, 将整个服务熔断, 一定时间后再恢复访问, 在 SpringCloud 中的熔断就是配置 4 个属性

![http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_7.png](http://www.miaomiaoqi.cn/images/springcloud/springcloud_sell_7.png)

**Closed:** 默认熔断器是关闭的, 当失败次数达到一定阈值, 会变为打开状态

**Open:** 此时所有的请求都会触发降级, 一定时间后, 会变为半打开状态

**Half Open:** 此时会释放一定的请求, 当请求成功达到一定比例, 会恢复为 Closed 状态

```java
// 熔断
@HystrixCommand(commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 设置熔断
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 默认值20.意思是至少有20个请求才进行 errorThresholdPercentage 错误百分比计算。
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), // 半开试探休眠时间，默认值5000ms。当熔断器开启一段时间之后比如5000ms，会尝试放过去一部分流量进行试探，确定依赖服务是否恢复。
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60") // 设定错误百分比，默认值50%，例如一段时间（10s）内有100个请求，其中有55个超时或者异常返回了，那么这段时间内的错误百分比是55%，大于了默认值50%，这种情况下触发熔断器-打开。 
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

## 依赖隔离

线程池隔离, Hystrix 自动实现了依赖隔离

## Zuul超时设置(降级)

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
  host: # 如果路由方式是 url 的方式，那么该超时配置生效
    connect-timeout-millis: 100 # HTTP连接超时
    socket-timeout-millis: 100  # socket超时

ribbon: # 如果路由方式是 serviceId 的方式，那么该全局超时配置生效
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
        // api服务id，如果需要所有调用都支持回退，则return "*"或return null
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
                result.put("msg", "系统错误，请求失败");
                return new ByteArrayInputStream(JsonUtil.toJson(result).getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                // 和body中的内容编码一致，否则容易乱码
                headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
                return headers;
            }

            /**
             * 网关向api服务请求是失败了，但是消费者客户端向网关发起的请求是OK的，
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



# 链路监控 Spring Cloud Sleuth



