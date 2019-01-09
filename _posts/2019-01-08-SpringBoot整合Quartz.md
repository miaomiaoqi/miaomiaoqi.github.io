---
layout: post
title:  "SpringBoot整合Quartz"
date:   2019-01-08 17:00:51
categories: Framework
tags: SpringBoot
author: miaoqi
---



## 基于SpringBoot & Quartz完成定时任务分布式单节点持久化

* 定时任务在企业项目比较常用到，几乎所有的项目都会牵扯该功能模块，定时任务一般会处理指定时间点执行某一些业务逻辑、间隔时间执行某一些业务逻辑等。

* 在SpringBoot中也提供了@Scheduled创建定时任务, 但是该方法是在内存中维护的定时任务, 由于某种特殊的原因定时任务可能丢失, 那对于我们来说可能是致命的问题
* 本次就研究一下基于`SpringBoot`架构整合定时任务框架`quartz`来完成**分布式单节点定时任务持久化**，将任务持久化到数据库，更好的预防任务丢失。

### 构建项目

* 我们使用`idea`开发工具创建一个`SpringBoot`项目，pom.xml依赖配置如下所示：

	```
	<properties>
	        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	        <java.version>1.8</java.version>
	        <druid.version>1.1.5</druid.version>
	        <quartz.version>2.3.0</quartz.version>
	    </properties>
	
	    <dependencies>
	        <!--spring data jpa相关-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-data-jpa</artifactId>
	        </dependency>
	        <!--web相关依赖-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <!--数据库相关依赖-->
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	            <scope>runtime</scope>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>druid</artifactId>
	            <version>${druid.version}</version>
	        </dependency>
	        <!--quartz相关依赖-->
	        <dependency>
	            <groupId>org.quartz-scheduler</groupId>
	            <artifactId>quartz</artifactId>
	            <version>${quartz.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.quartz-scheduler</groupId>
	            <artifactId>quartz-jobs</artifactId>
	            <version>${quartz.version}</version>
	        </dependency>
	        <!--定时任务需要依赖context模块-->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-context-support</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.projectlombok</groupId>
	            <artifactId>lombok</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-test</artifactId>
	            <scope>test</scope>
	        </dependency>
	    </dependencies>
	```

* 我们采用的是`quartz`官方最新版本`2.3.0`，新版本的任务调度框架做出了很多封装，使用也变得简易明了。
	创建初始化完成，下面我们来创建定时任务相关的`Configuration`配置。

### QuartzConfiguration

* `quartz`与`Spring`相关框架的整合方式有很多种，我们采用`jobDetail`使用`Spring Ioc`托管方式来完成整合，我们可以在定时任务实例中使用`Spring`注入注解完成业务逻辑处理，下面我先把全部的配置贴出来再逐步分析，配置类如下:

	```
	
	```


## 基于SpringBoot & Quartz完成定时任务分布式单节点持久化




