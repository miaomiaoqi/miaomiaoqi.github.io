---
layout: post
title:  "SpringBoot整合Quartz"
date:   2019-01-08 17:00:51
categories: Framework
tags: SpringBoot
author: miaoqi
---



[TOC]



# 基于SpringBoot & Quartz完成定时任务分布式单节点持久化

* 定时任务在企业项目比较常用到，几乎所有的项目都会牵扯该功能模块，定时任务一般会处理指定时间点执行某一些业务逻辑、间隔时间执行某一些业务逻辑等。
* 在SpringBoot中也提供了@Scheduled创建定时任务, 但是该方法是在内存中维护的定时任务, 由于某种特殊的原因定时任务可能丢失, 那对于我们来说可能是致命的问题
* 本次就研究一下基于`SpringBoot`架构整合定时任务框架`quartz`来完成**分布式单节点定时任务持久化**，将任务持久化到数据库，更好的预防任务丢失。

## 构建项目

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

## QuartzConfiguration

* `quartz`与`Spring`相关框架的整合方式有很多种，我们采用`jobDetail`使用`Spring Ioc`托管方式来完成整合，我们可以在定时任务实例中使用`Spring`注入注解完成业务逻辑处理，下面我先把全部的配置贴出来再逐步分析，配置类如下:

  ```
  package com.miaoqi.springboot.springbootquartzfirst.configuration;
  
  import org.quartz.spi.JobFactory;
  import org.quartz.spi.TriggerFiredBundle;
  import org.springframework.beans.factory.annotation.Autowire;
  import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
  import org.springframework.context.ApplicationContext;
  import org.springframework.context.ApplicationContextAware;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.core.io.ClassPathResource;
  import org.springframework.scheduling.annotation.EnableScheduling;
  import org.springframework.scheduling.quartz.SchedulerFactoryBean;
  import org.springframework.scheduling.quartz.SpringBeanJobFactory;
  
  import javax.sql.DataSource;
  
  @Configuration
  @EnableScheduling
  public class QuartzConfiguration {
      /**
       * 继承org.springframework.scheduling.quartz.SpringBeanJobFactory
       * 实现任务实例化方式
       */
      public static class AutowiringSpringBeanJobFactory extends SpringBeanJobFactory implements
              ApplicationContextAware {
  
          private transient AutowireCapableBeanFactory beanFactory;
  
          @Override
          public void setApplicationContext(final ApplicationContext context) {
              beanFactory = context.getAutowireCapableBeanFactory();
          }
  
          /**
           * 将job实例交给spring ioc托管
           * 我们在job实例实现类内可以直接使用spring注入的调用被spring ioc管理的实例
           * @param bundle
           * @return
           * @throws Exception
           */
          @Override
          protected Object createJobInstance(final TriggerFiredBundle bundle) throws Exception {
              final Object job = super.createJobInstance(bundle);
              /**
               * 将job实例交付给spring ioc
               */
              beanFactory.autowireBean(job);
              return job;
          }
      }
  
      /**
       * 配置任务工厂实例
       * @param applicationContext spring上下文实例
       * @return
       */
      @Bean
      public JobFactory jobFactory(ApplicationContext applicationContext) {
          /**
           * 采用自定义任务工厂 整合spring实例来完成构建任务
           * see {@link AutowiringSpringBeanJobFactory}
           */
          AutowiringSpringBeanJobFactory jobFactory = new AutowiringSpringBeanJobFactory();
          jobFactory.setApplicationContext(applicationContext);
          return jobFactory;
      }
  
      /**
       * 配置任务调度器
       * 使用项目数据源作为quartz数据源
       * @param jobFactory 自定义配置任务工厂
       * @param dataSource 数据源实例
       * @return
       * @throws Exception
       */
      @Bean(destroyMethod = "destroy", autowire = Autowire.NO)
      public SchedulerFactoryBean schedulerFactoryBean(JobFactory jobFactory, DataSource dataSource) throws Exception {
          SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
          //将spring管理job自定义工厂交由调度器维护
          schedulerFactoryBean.setJobFactory(jobFactory);
          //设置覆盖已存在的任务
          schedulerFactoryBean.setOverwriteExistingJobs(true);
          //项目启动完成后，等待2秒后开始执行调度器初始化
          schedulerFactoryBean.setStartupDelay(2);
          //设置调度器自动运行
          schedulerFactoryBean.setAutoStartup(true);
          //设置数据源，使用与项目统一数据源
          schedulerFactoryBean.setDataSource(dataSource);
          //设置上下文spring bean name
          schedulerFactoryBean.setApplicationContextSchedulerContextKey("applicationContext");
          //设置配置文件位置
          schedulerFactoryBean.setConfigLocation(new ClassPathResource("/quartz.properties"));
          return schedulerFactoryBean;
      }
  }
  ```

  * ### AutowiringSpringBeanJobFactory

  	可以看到上面配置类中，AutowiringSpringBeanJobFactory我们继承了SpringBeanJobFactory类，并且通过实现ApplicationContextAware接口获取ApplicationContext设置方法，通过外部实例化时设置ApplicationContext实例对象，在createJobInstance方法内，我们采用AutowireCapableBeanFactory来托管SpringBeanJobFactory类中createJobInstance方法返回的定时任务实例，这样我们就可以在定时任务类内使用Spring Ioc相关的注解进行注入业务逻辑实例了。

  * ### JobFactory

  	任务工厂是在本章配置调度器时所需要的实例，我们通过jobFactory方法注入ApplicationContext实例，来创建一个AutowiringSpringBeanJobFactory对象，并且将对象实例托管到Spring Ioc容器内。

  * ### SchedulerFactoryBean

  	我们本章采用的是项目内部数据源的方式来设置调度器的jobSotre，官方quartz有两种持久化的配置方案。

  	第一种：采用quartz.properties配置文件配置独立的定时任务数据源，可以与使用项目的数据库完全独立。

  	第二种：采用与创建项目统一个数据源，定时任务持久化相关的表与业务逻辑在同一个数据库内。

  	可以根据实际的项目需求采取不同的方案，我们本章主要是通过第二种方案来进行讲解，在上面配置类中可以看到方法schedulerFactoryBean内自动注入了JobFactory实例，也就是我们自定义的AutowiringSpringBeanJobFactory任务工厂实例，另外一个参数就是DataSource，在我们引入spring-starter-data-jpa依赖后会根据application.yml文件内的数据源相关配置自动实例化DataSource实例，这里直接注入是没有问题的。

  	我们通过调用SchedulerFactoryBean对象的setConfigLocation方法来设置quartz定时任务框架的基本配置，配置文件所在位置：resources/quartz.properties => classpath:/quartz.properties下。

  	下面我们来看下`quartz.properties`文件内的配置，如下所示：

  	```
  	#调度器实例名称
  	org.quartz.scheduler.instanceName=quartzScheduler
  	#调度器实例编号自动生成
  	org.quartz.scheduler.instanceId=AUTO
  	#持久化方式配置
  	org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
  	#持久化方式配置数据驱动，MySQL数据库
  	org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
  	#quartz相关数据表前缀名
  	org.quartz.jobStore.tablePrefix=QRTZ_
  	#开启分布式部署
  	org.quartz.jobStore.isClustered=true
  	#配置是否使用
  	org.quartz.jobStore.useProperties=false
  	#分布式节点有效性检查时间间隔，单位：毫秒
  	org.quartz.jobStore.clusterCheckinInterval=20000
  	#线程池实现类
  	org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
  	#执行最大并发线程数量
  	org.quartz.threadPool.threadCount=10
  	#线程优先级
  	org.quartz.threadPool.threadPriority=5
  	#配置为守护线程，设置后任务将不会执行
  	#org.quartz.threadPool.makeThreadsDaemons=true
  	#配置是否启动自动加载数据库内的定时任务，默认true
  	org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true
  	```

  	由于我们下一章需要做分布式多节点自动交付高可用，本章的配置文件加入了分布式相关的配置。
  	在上面配置中org.quartz.jobStore.class与org.quartz.jobStore.driverDelegateClass是定时任务持久化的关键配置，配置了数据库持久化定时任务以及采用MySQL数据库进行连接，当然这里我们也可以配置其他的数据库，如下所示：
  	PostgreSQL ： org.quartz.impl.jdbcjobstore.PostgreSQLDelegate
  	Sybase : org.quartz.impl.jdbcjobstore.SybaseDelegate
  	MSSQL : org.quartz.impl.jdbcjobstore.MSSQLDelegate
  	HSQLDB : org.quartz.impl.jdbcjobstore.HSQLDBDelegate
  	Oracle : org.quartz.impl.jdbcjobstore.oracle.OracleDelegate

  	org.quartz.jobStore.tablePrefix属性配置了定时任务数据表的前缀，在quartz官方提供的创建表SQL脚本默认就是qrtz_，在对应的XxxDelegate驱动类内也是使用的默认值，所以这里我们如果修改表名前缀，配置可以去掉。

  	org.quartz.jobStore.isClustered属性配置了开启定时任务分布式功能，再开启分布式时对应属性org.quartz.scheduler.instanceId 改成Auto配置即可，实例唯一标识会自动生成，这个标识具体生成的内容，我们一会

  	在运行的控制台就可以看到了，定时任务分布式准备好后会输出相关的分布式节点配置信息。

## 准备测试

# 基于SpringBoot & Quartz完成定时任务分布式单节点持久化




