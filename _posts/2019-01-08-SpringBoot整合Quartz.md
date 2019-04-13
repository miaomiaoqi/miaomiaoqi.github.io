---
layout: post
title:  "SpringBoot整合Quartz"
date:   2018-01-08 14:06:15
categories: Framework
tags: SpringBoot
author: miaoqi
---

* content
{:toc}
# 基于SpringBoot & Quartz完成定时任务分布式单节点持久化

定时任务在企业项目比较常用到, 几乎所有的项目都会牵扯该功能模块, 定时任务一般会处理指定时间点执行某一些业务逻辑、间隔时间执行某一些业务逻辑等. 

在SpringBoot中也提供了@Scheduled创建定时任务, 但是该方法是在内存中维护的定时任务, 由于某种特殊的原因定时任务可能丢失, 那对于我们来说可能是致命的问题

本次就研究一下基于`SpringBoot`架构整合定时任务框架`quartz`来完成**分布式单节点定时任务持久化**, 将任务持久化到数据库, 更好的预防任务丢失. 

## 构建项目

我们使用`idea`开发工具创建一个`SpringBoot`项目, pom.xml依赖配置如下所示：

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
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
    </dependency>
</dependencies>
```

我们采用的是`quartz`官方最新版本`2.3.0`, 新版本的任务调度框架做出了很多封装, 使用也变得简易明了. 
创建初始化完成, 下面我们来创建定时任务相关的`Configuration`配置. 

## 创建Quartz配置类

`quartz`与`Spring`相关框架的整合方式有很多种, 我们采用`jobDetail`使用`Spring Ioc`托管方式来完成整合, 我们可以在定时任务实例中使用`Spring`注入注解完成业务逻辑处理, 配置类如下:

- **AutowiringSpringBeanJobFactory**

    ```
    package com.miaoqi.springboot.springbootquartz.configuration;
    
    import org.quartz.spi.TriggerFiredBundle;
    import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    import org.springframework.scheduling.quartz.SpringBeanJobFactory;
    import org.springframework.stereotype.Component;
    
    /**
     * 继承org.springframework.scheduling.quartz.SpringBeanJobFactory
     * 实现任务实例化方式
     */
    @Component
    public class AutowiringSpringBeanJobFactory extends SpringBeanJobFactory implements
            ApplicationContextAware {
    
        private transient AutowireCapableBeanFactory beanFactory;
    
        @Override
        public void setApplicationContext(final ApplicationContext context) {
            beanFactory = context.getAutowireCapableBeanFactory();
        }
    
        /**
         * 将job实例交给spring ioc托管
         * 我们在job实例实现类内可以直接使用spring注入的调用被spring ioc管理的实例
         *
         * @author miaoqi
         * @date 2019/1/15
         * @param bundle
         * @return
         */
        @Override
        protected Object createJobInstance(final TriggerFiredBundle bundle) throws Exception {
            final Object job = super.createJobInstance(bundle);
            // 将job实例交付给spring ioc
            beanFactory.autowireBean(job);
            return job;
        }
    }
    ```

    可以看到上面配置类中, AutowiringSpringBeanJobFactory我们继承了SpringBeanJobFactory类, 并且通过实现ApplicationContextAware接口获取ApplicationContext设置方法, 通过外部实例化时设置ApplicationContext实例对象, 在createJobInstance方法内, 我们采用AutowireCapableBeanFactory来托管SpringBeanJobFactory类中createJobInstance方法返回的定时任务实例, **这样我们就可以在定时任务类内使用Spring Ioc相关的注解进行注入业务逻辑实例了. **

- **QuartzPropertiesConfig**

    ```
    package com.miaoqi.springboot.springbootquartz.configuration;
    
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Configuration;
    
    import java.util.Properties;
    
    /**
     * quartz配置类
     *
     * @author miaoqi
     * @date 2019/1/14
     */
    @Configuration
    @ConfigurationProperties(prefix = "org")
    public class QuartzPropertiesConfig {
    
        private Properties quartz = new Properties();
    
        public Properties getQuartz() {
            return quartz;
        }
    
        public void setQuartz(Properties quartz) {
            this.quartz = quartz;
        }
    }
    ```

    该配置类是读取SpringBoot的配置信息的配置对象, 不熟悉的人可以了解一下SprongBoot的自动配置

- **QuartzThreadPoolConfiguration**

    ```
    package com.miaoqi.springboot.springbootquartz.configuration;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
    
    /**
     * Quartz执行任务的线程池配置
     *
     * @author miaoqi
     * @date 2019/1/15
     */
    @Configuration
    public class QuartzThreadPoolConfiguration {
    
        @Bean
        ThreadPoolTaskExecutor quartzThreadPoolExecutor() {
            ThreadPoolTaskExecutor poolTaskExecutor = new ThreadPoolTaskExecutor();
            // 线程池所使用的缓冲队列
            poolTaskExecutor.setQueueCapacity(0);
            // 线程池维护线程的最少数量
            poolTaskExecutor.setCorePoolSize(5);
            // 线程池维护线程的最大数量
            poolTaskExecutor.setMaxPoolSize(1000);
            // 线程池维护线程所允许的空闲时间
            poolTaskExecutor.setKeepAliveSeconds(30000);
            return poolTaskExecutor;
        }
    }
    ```

    这个类配置的是与Quartz线程池相关的信息

- **SchedulerFactoryConfiguration**

    ```
    package com.miaoqi.springboot.springbootquartz.configuration;
    
    import org.quartz.spi.JobFactory;
    import org.springframework.beans.factory.annotation.Autowire;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.scheduling.annotation.EnableScheduling;
    import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
    import org.springframework.scheduling.quartz.SchedulerFactoryBean;
    
    import java.util.Properties;
    
    @Configuration
    @EnableScheduling
    public class SchedulerFactoryConfiguration {
    
        @Autowired
        private ThreadPoolTaskExecutor threadPoolTaskExecutor;
        @Autowired
        private QuartzPropertiesConfig quartzPropertiesConfig;
    
        /**
         * 配置任务调度器
         * 使用项目数据源作为quartz数据源
         * @param jobFactory 自定义配置任务工厂
         * @return
         * @throws Exception
         */
        @Bean(destroyMethod = "destroy", autowire = Autowire.NO)
        public SchedulerFactoryBean schedulerFactoryBean(JobFactory jobFactory) throws Exception {
            SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
            // 将spring管理job自定义工厂交由调度器维护
            schedulerFactoryBean.setJobFactory(jobFactory);
            // 设置覆盖已存在的任务
            schedulerFactoryBean.setOverwriteExistingJobs(true);
            // 项目启动完成后, 等待2秒后开始执行调度器初始化
            schedulerFactoryBean.setStartupDelay(2);
            // 设置调度器自动运行
            schedulerFactoryBean.setAutoStartup(true);
            // 设置数据源(使用系统的主数据源, 覆盖propertis文件的dataSource配置)
            // schedulerFactoryBean.setDataSource(dataSource);
            // 设置上下文spring bean name
            schedulerFactoryBean.setApplicationContextSchedulerContextKey("applicationContext");
            // 设置配置文件位置
            // schedulerFactoryBean.setConfigLocation(new ClassPathResource("/quartz.properties"));
            // 将配置文件的内容以properties对象的方式配置
            Properties raw = quartzPropertiesConfig.getQuartz();
            Properties quartzProperties = new Properties();
            ConfigurationProperties annotation = QuartzPropertiesConfig.class.getDeclaredAnnotation(
                    ConfigurationProperties.class);
            for (Object key : raw.keySet()) {
                Object value = raw.get(key);
                quartzProperties.setProperty(annotation.prefix() + ".quartz." + key, value.toString());
            }
            schedulerFactoryBean.setQuartzProperties(quartzProperties);
            // 设置线程池
            schedulerFactoryBean.setTaskExecutor(threadPoolTaskExecutor);
            // 设置调度器名称, 手动设置 > bean实例名称 > 配置文件
            schedulerFactoryBean.setSchedulerName("mySchedulerFactoryBean");
            return schedulerFactoryBean;
        }
    }
    ```

    创建调度器的工厂bean对象, Quartz可以使用独立的数据源, 也可以与业务类使用相同的数据源, 如果使用与业务相同的数据源只需要在创建SchedulerFactoryBean的时候手动注入DataSource即可, 如果使用独立的数据源需要在配置文件中进行配置, 配置文件有两种方式, 一种是采用quartz.properties配置文件, 这种是官网原生的配置方式, 另外一种是将配置写入SpringBoot的配置文件中, 在自动配置到一个Properties对象中注入给SchedulerFactoryBean, **两种方法本质上是一样的, 我采用的是第二种**, 另外一个重要的属性是schedulerName, 这个属性默认会使用配置文件中的值, 如果配置文件中没有配置则会取bean的名字, 如果在代码中手动设置, 则手动的优先级最高, **并且这个属性的值在不同的项目中不能重复, 否则有可能造成错误消费, 导致定时任务找不到执行的类而影响业务逻辑, 这一点一定要注意**

    

    下面我们来看下`application.yml`文件内的配置, 如下所示：

    ```
    # 业务数据源
    spring:
      datasource:
        username: root
        password: miaoqi
        url: jdbc:mysql://127.0.0.1:3306/quartz_business_db
        driver‐class‐name: com.mysql.jdbc.Driver
        
    org:
      quartz:
        scheduler:
          # 调度器实例名称
          instanceName: quartzScheduler
          # 调度器实例编号自动生成
          instanceId: AUTO
        jobStore:
          # 持久化方式配置(存储方式使用JobStoreTX, 也就是数据库)
          class: org.quartz.impl.jdbcjobstore.JobStoreTX
          # 持久化方式配置数据驱动, MySQL数据库
          driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
          # quartz相关数据表前缀名
          tablePrefix: QRTZ_
          # quartz相关的数据库
          dataSource: Qrtz
          # 开启分布式部署
          isClustered: true
          # 配置是否使用
          useProperties: false
          # 分布式节点有效性检查时间间隔, 单位：毫秒
          clusterCheckinInterval: 20000
        threadPool:
          # 线程池实现类
          class: org.quartz.simpl.SimpleThreadPool
          # 执行最大并发线程数量
          threadCount: 10
          # 线程优先级
          threadPriority: 5
          # 配置为守护线程, 设置后任务将不会执行
          # makeThreadsDaemons: true
          # 配置是否启动自动加载数据库内的定时任务, 默认true
          threadsInheritContextClassLoaderOfInitializingThread: true
        
        #============================================================================
        # Configure Datasources配置数据源(可被覆盖, 如果在schedulerFactoryBean指定数据源)
        #============================================================================
        # 单独配置quartz的数据源, 与业务数据库隔离开来
        dataSource:
          # 这个名字与org.quartz.scheduler.jobStore.dataSource的值一直
          Qrtz:
            driver: com.mysql.jdbc.Driver
            URL: jdbc:mysql://127.0.0.1:3306/quartz_job_db?useUnicode=true&characterEncoding=utf8
            user: root
            password: miaoqi
            validationQuery: select 1
    ```

    由于我们后续需要做分布式多节点自动交付高可用, 配置文件加入了分布式相关的配置. 
    	

    在上面配置中org.quartz.jobStore.class与org.quartz.jobStore.driverDelegateClass是定时任务持久化的关键配置, 配置了数据库持久化定时任务以及采用MySQL数据库进行连接, 当然这里我们也可以配置其他的数据库, 如下所示:

    ​	PostgreSQL ： org.quartz.impl.jdbcjobstore.PostgreSQLDelegate

    ​	Sybase : org.quartz.impl.jdbcjobstore.SybaseDelegate

    ​	MSSQL : org.quartz.impl.jdbcjobstore.MSSQLDelegate

    ​	HSQLDB : org.quartz.impl.jdbcjobstore.HSQLDBDelegate

    ​	Oracle : org.quartz.impl.jdbcjobstore.oracle.OracleDelegate

    org.quartz.jobStore.tablePrefix属性配置了定时任务数据表的前缀, 在quartz官方提供的创建表SQL脚本默认就是qrtz_, 在对应的XxxDelegate驱动类内也是使用的默认值, 所以这里我们如果修改表名前缀, 配置可以去掉. 

    org.quartz.jobStore.isClustered属性配置了开启定时任务分布式功能, 再开启分布式时对应属性org.quartz.scheduler.instanceId 改成Auto配置即可, 实例唯一标识会自动生成, 这个标识具体生成的内容, 我们一会

​	在运行的控制台就可以看到了, 定时任务分布式准备好后会输出相关的分布式节点配置信息. 



## 创建任务

### 定义固定执行时间定时任务

我们先来创建一个任务实例, 并且继承`org.springframework.scheduling.quartz.QuartzJobBean`抽象类, 重写父抽象类内的`executeInternal`方法来实现任务的主体逻辑. 如下所示：

```
package com.miaoqi.springboot.springbootquartzfirst.job;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;

import java.util.Date;

/**
 * 固定时间任务
 *
 * @author miaoqi
 * @date 2019/1/14
 */
public class StartAtJob extends QuartzJobBean {
    /**
     * logback
     */
    static Logger logger = LoggerFactory.getLogger(StartAtJob.class);

    /**
     * 定时任务逻辑实现方法
     * 每当触发器触发时会执行该方法逻辑
     * @param jobExecutionContext 任务执行上下文
     * @throws JobExecutionException
     */
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("startAt job, execute at：{}", new Date());
    }
}
```

### 定义cron表达式定时任务

同样需要继承`org.springframework.scheduling.quartz.QuartzJobBean`抽象类实现抽象类内的`executeInternal`方法, 如下所示：

```
package com.miaoqi.springboot.springbootquartzfirst.job;

import com.miaoqi.springboot.springbootquartzfirst.service.ProductService;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * cron表达式定时任务
 *
 * @author miaoqi
 * @date 2019/1/14
 */
@Component
public class CronJob extends QuartzJobBean {

    @Autowired
    private ProductService productService;

    /**
     * logback
     */
    static Logger logger = LoggerFactory.getLogger(CronJob.class);

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("cron job, execute at：{}", new Date());
        this.productService.business();
    }
}
```

**在CronJob中我们注入了一个ProductService业务类, 模拟定时任务调用业务方法**



## 将任务与触发器关联到调度器

### 设置固定执行时间定时任务到调度器

```
package com.miaoqi.springboot.springbootquartzfirst.scheduler;

import com.miaoqi.springboot.springbootquartzfirst.job.StartAtJob;
import org.quartz.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.UUID;

/**
 * 实现ApplicationRunner可以在项目启动时就执行run方法进行任务的注册
 *
 * @author miaoqi
 * @date 2019/1/15
 */
@Component
public class StartAtScheduler implements ApplicationRunner {

    /**
     * 注入任务调度器
     */
    @Autowired
    private Scheduler scheduler;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 设置开始时间为1分钟后
        long startAtTime = System.currentTimeMillis() + 1000 * 60;
        // 任务名称
        String name = UUID.randomUUID().toString();
        // 任务所属分组
        String group = StartAtJob.class.getName();
        // 删除现有任务, 防止重复添加导致项目启动失败
        JobKey jobKey = JobKey.jobKey(name, group);
        scheduler.deleteJob(jobKey);
        // 创建任务
        JobDetail jobDetail = JobBuilder.newJob(StartAtJob.class).withIdentity(name, group).build();
        // 创建任务触发器
        Trigger trigger = TriggerBuilder.newTrigger().withIdentity(name, group).startAt(new Date(startAtTime)).build();
        // 将触发器与任务绑定到调度器内
        scheduler.scheduleJob(jobDetail, trigger);
    }
}
```

在上面方法中我们定义的`StartAtJob`实例只运行一次, 在项目启动完成后延迟1分钟进行调用任务主体逻辑. 

**其中任务的名称以及任务的分组是为了区分任务做的限制, 在同一个分组下如果加入同样名称的任务, 则会提示任务已经存在, 添加失败的提示. **

我们通过`JobDetail`来构建一个任务实例, 设置`CronJob`类作为任务运行目标对象, 当任务被触发时就会执行`CronJob`内的`executeInternal`方法. 

一个任务需要设置对应的触发器, 触发器也分为很多种, 该任务中我们并没有采用`cron`表达式来设置触发器, 而是调用`startAt`方法设置任务开始执行时间. 

**最后将任务以及任务的触发器共同交付给任务调度器, 这样就完成了一个任务的设置. **

### 设置cron表达式定时任务调度器

```
package com.miaoqi.springboot.springbootquartzfirst.scheduler;

import com.miaoqi.springboot.springbootquartzfirst.job.CronJob;
import org.quartz.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

import java.util.UUID;

/**
 * 实现ApplicationRunner可以在项目启动时就执行run方法进行任务的注册
 *
 * @author miaoqi
 * @date 2019/1/15
 */
@Component
public class CronScheduler implements ApplicationRunner {

    /**
     * 注入任务调度器
     */
    @Autowired
    private Scheduler scheduler;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 任务名称
        String name = UUID.randomUUID().toString();
        // 任务所属分组
        String group = CronJob.class.getName();
		// 删除现有任务, 防止重复添加导致项目启动失败
        JobKey jobKey = JobKey.jobKey(name, group);
        scheduler.deleteJob(jobKey);
        // 构建corn表达式触发器
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/15 * * * * ?");
        // 创建任务
        JobDetail jobDetail = JobBuilder.newJob(CronJob.class).withIdentity(name, group).build();
        // 创建任务触发器
        Trigger trigger = TriggerBuilder.newTrigger().withIdentity(name, group).withSchedule(scheduleBuilder).build();
        // 将触发器与任务绑定到调度器内
        scheduler.scheduleJob(jobDetail, trigger);
    }
}
```

该任务的触发器我们采用了`cron`表达式来设置, 每隔15秒执行一次任务主体逻辑. 

**任务触发器在创建时`cron`表达式可以搭配`startAt`方法来同时使用. **

下面我们就来测试下任务是否可以顺序的被持久化到数据库, 并且是否可以在重启服务后执行重启前添加的任务. 

## 测试

下面我们来启动项目, 启动成功后, 我们来查看控制台输出的分布式节点的信息, 如下所示：

```
2019-01-15 14:23:09.382  INFO 67908 --- [anualScheduler]] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now, after delay of 2 seconds
2019-01-15 14:23:09.392  INFO 67908 --- [anualScheduler]] org.quartz.core.QuartzScheduler          : Scheduler manualScheduler_$_localhost1547533386943 started.
```

定时任务是在项目启动后2秒进行执行初始化, 完成了`instance`的创建, 创建的节点唯一标识为_localhost1547533386943. 

接下来我们查看任务是否写入成功呢? 我们来查看`qrtz_job_details`表内任务列表, 如下所示

```
manualScheduler	CronScheduler	TestQuartz		com.miaoqi.springboot.springbootquartzfirst.job.CronJob	0	0	0	0	
manualScheduler	StartAtScheduler	TestQuartz		com.miaoqi.springboot.springbootquartzfirst.job.StartAtJob	0	0	0	0	
```

任务已经被成功的持久化到数据库内, 等待1分钟后查看控制台输出内容如下所示：

```
2019-01-15 14:23:45.013  INFO 67908 --- [eduler_Worker-3] c.m.s.springbootquartzfirst.job.CronJob  : cron job, execute at：Tue Jan 15 14:23:45 CST 2019
业务类执行了
2019-01-15 14:24:00.014  INFO 67908 --- [eduler_Worker-4] c.m.s.springbootquartzfirst.job.CronJob  : cron job, execute at：Tue Jan 15 14:24:00 CST 2019
业务类执行了
2019-01-15 14:24:07.543  INFO 67908 --- [eduler_Worker-5] c.m.s.s.job.StartAtJob                   : startAt job, execute at：Tue Jan 15 14:24:07 CST 2019
2019-01-15 14:24:15.013  INFO 67908 --- [eduler_Worker-6] c.m.s.springbootquartzfirst.job.CronJob  : cron job, execute at：Tue Jan 15 14:24:15 CST 2019
业务类执行了
```

根据输出的内容来判定完全吻合我们的配置参数, 库存检查为15秒执行一次, 而添加成功后的提醒则是1分钟后执行一次. 执行完成后就会被直接销毁, 我们再来查看数据库表`qrtz_job_details`, 这时就可以看到还剩下`1个任务`. 

```
manualScheduler	CronScheduler	TestQuartz		com.miaoqi.springboot.springbootquartzfirst.job.CronJob	0	0	0	0	
```

至此我们的分布式单节点SpringBoot整合Quartz告一段落, 接下来是多节点的整合

# 基于SpringBoot & Quartz完成定时任务分布式多节点负载持久化

我们基于单节点项目复制一份代码, 因为实际项目中同一个Scheduler是根据job的全类名去执行任务的, 而多节点一般情况下是集群模式

## 配置分布式

- **org.quartz.scheduler.instanceId**

    定时任务的实例编号, 如果手动指定需要保证每个节点的唯一性, 因为`quartz`不允许出现两个相同`instanceId`的节点, 我们这里指定为`Auto`就可以了, 我们把生成编号的任务交给`quartz`. 

- **org.quartz.jobStore.isClustered**

    这个属性才是真正的开启了定时任务的分布式配置, 当我们配置为`true`时`quartz`框架就会调用`ClusterManager`来初始化分布式节点. 

- **org.quartz.jobStore.clusterCheckinInterval**

    配置了分布式节点的检查时间间隔, 单位：毫秒. 

- **threadsInheritContextClassLoaderOfInitializingThread**

    配置进行是否自动加载任务, 默认`true`自动加载数据库内的任务到节点. 

- 修改第二个项目的端口号不要与第一个项目冲突

## 修改job的代码区分两个项目

### 修改first项目代码

```
package com.miaoqi.springboot.springbootquartz.job;

import com.miaoqi.springboot.springbootquartz.service.ProductService;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * cron表达式定时任务
 *
 * @author miaoqi
 * @date 2019/1/14
 */
@Component
public class CronJob extends QuartzJobBean {

    @Autowired
    private ProductService productService;

    /**
     * logback
     */
    static Logger logger = LoggerFactory.getLogger(CronJob.class);

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("first cron job, execute at：{}", new Date());
        this.productService.business();
    }
}
```

### 修改second项目代码

```
package com.miaoqi.springboot.springbootquartz.job;

import com.miaoqi.springboot.springbootquartz.service.ProductService;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * cron表达式定时任务
 *
 * @author miaoqi
 * @date 2019/1/14
 */
@Component
public class CronJob extends QuartzJobBean {

    @Autowired
    private ProductService productService;

    /**
     * logback
     */
    static Logger logger = LoggerFactory.getLogger(CronJob.class);

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        logger.info("second cron job, execute at：{}", new Date());
        this.productService.business();
    }
}
```



## 启动项目

### 启动项目一

启动项目一, 观察日志

```
2019-01-15 15:03:58.998  INFO 82022 --- [anualScheduler]] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now, after delay of 2 seconds
2019-01-15 15:03:59.006  INFO 82022 --- [anualScheduler]] org.quartz.impl.jdbcjobstore.JobStoreTX  : ClusterManager: detected 1 failed or restarted instances.
2019-01-15 15:03:59.007  INFO 82022 --- [anualScheduler]] org.quartz.impl.jdbcjobstore.JobStoreTX  : ClusterManager: Scanning for instance "localhost1547534376982"'s failed in-progress jobs.
2019-01-15 15:03:59.010  INFO 82022 --- [anualScheduler]] org.quartz.core.QuartzScheduler          : Scheduler manualScheduler_$_localhost1547535836521 started.
```

**从日志中我们可以看到, 项目一是从ClusterManager中获取到了自动分配的instanceId为_localhost1547535836521, 生成的规则是当前用户的名称+时间戳**. 然后`ClusterManager`分布式管理者自动介入进行扫描是否存在匹配的触发器任务, 如果存在则会自动执行任务逻辑

查看输出

```
2019-01-15 15:04:00.097  INFO 82022 --- [eduler_Worker-1] c.m.s.springbootquartz.job.CronJob       : first cron job, execute at：Tue Jan 15 15:04:00 CST 2019
业务类执行了
2019-01-15 15:04:15.013  INFO 82022 --- [eduler_Worker-2] c.m.s.springbootquartz.job.CronJob       : first cron job, execute at：Tue Jan 15 15:04:15 CST 2019
业务类执行了
```

通过日志的信息我们可以看到输出内容中带有"first", 确实是第一个项目中的输出

### 启动项目二

启动项目二, 观察日志

```
2019-01-15 15:08:11.979  INFO 83229 --- [anualScheduler]] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now, after delay of 2 seconds
2019-01-15 15:08:11.985  INFO 83229 --- [anualScheduler]] org.quartz.core.QuartzScheduler          : Scheduler manualScheduler_$_localhost1547536089485 started.
```

项目启动完成后, 定时节点并没有实例化`ClusterManager`来完成分布式节点的初始化, 因为`quartz`检测到有其他的节点正在处理任务, 这样也是保证了任务执行的唯一性. 

## 测试任务自动漂移

我们关闭first项目, 预计达到second项目会自动接管数据库中的任务, 完成任务执行的自动漂移, 关闭第一个项目后我们观察一下日志

```
2019-01-15 15:08:11.979  INFO 83229 --- [anualScheduler]] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now, after delay of 2 seconds
2019-01-15 15:08:11.985  INFO 83229 --- [anualScheduler]] org.quartz.core.QuartzScheduler          : Scheduler manualScheduler_$_localhost1547536089485 started.
2019-01-15 15:09:10.156  INFO 83229 --- [eduler_Worker-1] c.m.s.springbootquartz.job.StartAtJob    : startAt job, execute at：Tue Jan 15 15:09:10 CST 2019
2019-01-15 15:11:00.015  INFO 83229 --- [eduler_Worker-2] c.m.s.springbootquartz.job.CronJob       : second cron job, execute at：Tue Jan 15 15:11:00 CST 2019
业务类执行了
```

通过日志我们可以看到, 日志的信息变成了second, 说明我们的任务自动漂移成功了, 项目二完成了自动接管项目一的任务, 而这个过程肯定有一段时间间隔, 而这个间隔可以修改配置文件中的clusterCheckinInterval进行调节

**如果两个节点同时启动, 哪个节点先把节点信息注册到数据库就获得了优先执行权. **



# Quartz任务调度的基本实现原理

## Quartz核心元素

Quartz任务调度的核心元素为：Scheduler——任务调度器、Trigger——触发器、Job——任务. 其中trigger和job是任务调度的元数据, scheduler是实际执行调度的控制器. Quartz把触发job，叫做**fire**。**TRIGGER_STATE**是当前trigger的状态，**PREV_FIRE_TIME**是上一次触发时间，**NEXT_FIRE_TIME**是下一次触发时间，**misfire**是指这个job在某一时刻要触发，却因为某些原因没有触发的情况。

### Trigger

是用于定义调度时间的元素, 即按照什么时间规则去执行任务. Quartz中主要提供了**四种类型的trigger：SimpleTrigger, CronTirgger, DateIntervalTrigger, 和NthIncludedDayTrigger.** 这四种trigger可以满足企业应用中的绝大部分需求. 

- SimpleTrigger：简单触发器, 从某个时间开始, 每隔多少时间触发, 重复多少次. 

- CronTrigger：使用cron表达式定义触发的时间规则, 如"0 0 0,2,4 1/1 * ? *" 表示每天的0, 2, 4点触发. 

- DailyTimeIntervalTrigger：每天中的一个时间段, 每N个时间单元触发, 时间单元可以是毫秒, 秒, 分, 小时

- CalendarIntervalTrigger：每N个时间单元触发, 时间单元可以是毫秒, 秒, 分, 小时, 日, 月, 年. 

- trigger状态：`WAITING, ACQUIRED, EXECUTING, COMPLETE, BLOCKED, ERROR, PAUSED, PAUSED_BLOCKED, DELETED`

    ![![http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_4.png]()]()

    trigger的初始状态是**WAITING**，处于**WAITING**状态的trigger等待被触发。调度线程会不停地扫triggers表，根据NEXT_FIRE_TIME提前拉取即将触发的trigger，如果这个trigger被该调度线程拉取到，它的状态就会变为**ACQUIRED**。因为是提前拉取trigger，并未到达trigger真正的触发时刻，所以调度线程会等到真正触发的时刻，再将trigger状态由**ACQUIRED**改为**EXECUTING**。如果这个trigger不再执行，就将状态改为**COMPLETE**,否则为**WAITING**，开始新的周期。如果这个周期中的任何环节抛出异常，trigger的状态会变成**ERROR**。如果手动暂停这个trigger，状态会变成**PAUSED**。

- 未正常触发的任务：misfire job

    没有在正常触发时间点触发的任务. 主要由一下几种情况导致：

    - 触发时间在应用不可用的时间内, 比如重启
    - 上次的执行时间过长, 超过了下次触发的时间
    - 任务被暂停一段时间后, 重新被调度的时间在下次触发时间之后

    处理misfire job的策略, 需要在创建trigger的时候配置, 每种trigger对应的枚举值都不同, 具体在接口里面有定义. CronTrigger有2种处理misfire的策略：

    | 处理策略                          | 描述                       |
    | --------------------------------- | -------------------------- |
    | MISFIRE_INSTRUCTION_FIRE_ONCE_NOW | 立即触发一次               |
    | MISFIRE_INSTRUCTION_DO_NOTHING    | 忽略, 不处理, 等待下次触发 |

### Job

用于表示被调度的任务. 主要有两种类型的job：无状态的（stateless）和有状态的（stateful）. 对于同一个trigger来说, 有状态的job不能被并行执行, 只有上一次触发的任务被执行完之后, 才能触发下一次执行. Job主要有两种属性：volatility和durability, 其中volatility表示任务是否被持久化到数据库存储, 而durability表示在没有trigger关联的时候任务是否被保留. 两者都是在值为true的时候任务被持久化或保留. **一个job可以被多个trigger关联, 但是一个trigger只能关联一个job**. 

### Scheduler

由scheduler工厂创建：DirectSchedulerFactory或者StdSchedulerFactory. 第二种工厂StdSchedulerFactory使用较多, 因为DirectSchedulerFactory使用起来不够方便, 需要作许多详细的手工编码设置. Scheduler主要有三种：RemoteMBeanScheduler, RemoteScheduler和StdScheduler. 主要负责job和trigger的持久化管理, 包括新增、删除、修改、触发、暂停、恢复调度、停止调度等；

![http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_1.png](http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_1.png)



## Quartz线程视图

在Quartz中, 有两类线程, Scheduler调度线程和任务执行线程, 其中任务执行线程通常使用一个线程池维护一组线程. 

![http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_2.png](http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_2.png)

Scheduler调度线程主要有两个：执行常规调度的线程, 和执行misfiredtrigger的线程. **常规调度线程轮询存储的所有trigger, 如果有需要触发的trigger, 即到达了下一次触发的时间, 则从任务执行线程池获取一个空闲线程, 执行与该trigger关联的任务. Misfire线程是扫描所有的trigger, 查看是否有misfiredtrigger, 如果有的话根据misfire的策略分别处理**(**fire now** OR **wait for the next fire**). **处理misfire job的线程MisfireHandler：轮训所有misfire的trigger, 原理就是从数据库中查询所有下次触发时间小于当前时间的trigger, 按照每个trigger设定的misfire策略处理这些trigger. **



## QuartzJob数据存储

Quartz中的trigger和job需要存储下来才能被使用. Quartz中有两种存储方式：RAMJobStore,JobStoreSupport, 其中RAMJobStore是将trigger和job存储在内存中, 而JobStoreSupport是基于jdbc将trigger和job存储到数据库中. **RAMJobStore的存取速度非常快, 但是由于其在系统被停止后所有的数据都会丢失, 所以在集群应用中, 必须使用JobStoreSupport. **



# Quartz集群原理

## Quartz 集群架构

一个Quartz集群中的每个节点是一个独立的Quartz应用, 它又管理着其他的节点. 这就意味着你必须对每个节点分别启动或停止. Quartz集群中, 独立的Quartz节点并不与另一其的节点或是管理节点通信, 而是通过相同的数据库表来感知到另一Quartz应用的, 如图2.1所示. 

![http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_3.png](http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_3.png)

## Quartz 集群相关数据库表

因为Quartz集群依赖于数据库, 所以必须首先创建Quartz数据库表, Quartz发布包中包括了所有被支持的数据库平台的SQL脚本. 这些SQL脚本存放于<quartz_home>/docs/dbTables 目录下. 这里采用的Quartz 2.3.0版本, 总共11张表, 不同版本, 表个数可能不同. 数据库为mysql, 用tables_mysql_innodb.sql创建数据库表

- QRTZ_FIRED_TRIGGERS(触发器与任务关联表)

    存储与已触发的Trigger相关的状态信息, 以及相联Job的执行信息. 

- QRTZ_PAUSED_TRIGGER_GRPS

- QRTZ_SCHEDULER_STATE(调度器状态表)

    集群中节点实例信息, Quartz定时读取该表的信息判断集群中每个实例的当前状态. 

    - instance_name

        配置文件中org.quartz.scheduler.instanceId配置的名字, 如果设置为AUTO,quartz会根据物理机名和当前时间产生一个名字. 

    - last_checkin_time

        上次检入时间

    - checkin_interval

        检入间隔时间

- QRTZ_LOCKS(实现同步机制的锁表)

    QRTZ_LOCKS表就是Quartz集群实现同步机制的行锁表

- QRTZ_SIMPLE_TRIGGERS

- QRTZ_SIMPROP_TRIGGERS

- QRTZ_CRON_TRIGGERS

- QRTZ_BLOB_TRIGGERS

- QRTZ_TRIGGERS(触发器信息表)

    - trigger_name

        trigger的名字,该名字用户自己可以随意定制,无强行要求

    - trigger_group

        trigger所属组的名字,该名字用户自己随意定制,无强行要求

    - job_name

        qrtz_job_details表job_name的外键

    - job_group

        qrtz_job_details表job_group的外键

    - trigger_state

        当前trigger状态设置为ACQUIRED,如果设为WAITING,则job不会触发

    - trigger_cron

        触发器类型,使用cron表达式

- QRTZ_JOB_DETAILS(任务详细信息表)

    保存job详细信息,该表需要用户根据实际情况初始化

    - job_name

        集群中job的名字,该名字用户自己可以随意定制,无强行要求. 

    - job_group

        集群中job的所属组的名字,该名字用户自己随意定制,无强行要求. 

    - job_class_name

        集群中job实现类的完全包名,quartz就是根据这个路径到classpath找到该job类的. 

    - is_durable

        是否持久化,把该属性设置为1, quartz会把job持久化到数据库中

    - job_data

        一个blob字段, 存放持久化job对象. 

- QRTZ_CALENDARS



## 集群原理分析

trigger的状态储存在数据库，Quartz支持分布式，所以如果起了多个quartz服务，会有多个调度线程来抢夺触发同一个trigger。mysql在默认情况下执行select 语句，是不上锁的，那么如果同时有1个以上的调度线程抢到同一个trigger，是否会导致这个trigger重复调度呢？我们来看看，Quartz是如何解决这个问题的。



Quartz首先会通过LCOKS表获取行所, 执行如下SQL:

```
SELECT * FROM QRTZ_LOCKS WHERE CHED_NAME = 'quartzScheduler' AND LOCK_NAME = ? FOR UPDATE
```

这条SQL会给LOCKS表加上悲观锁, 其他线程就只能等待锁的释放

![http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_5.png](http://www.miaomiaoqi.cn/images/distributed/quartz/quartz_5.png)

### 拉取待触发trigger

调度线程会一次性拉取距离现在，一定时间窗口内的，一定数量内的，即将触发的trigger信息。那么，时间窗口和数量信息如何确定呢，我们先来看一下，以下几个参数：

- `idleWaitTime`： 默认30s，可通过配置属性`org.quartz.scheduler.idleWaitTime`设置。
- `availThreadCount`：获取可用（空闲）的工作线程数量，总会大于1，因为该方法会一直阻塞，直到有工作线程空闲下来。
- `maxBatchSize`：一次拉取trigger的最大数量，默认是1，可通过`org.quartz.scheduler.batchTriggerAcquisitionMaxCount`改写
- `batchTimeWindow`：时间窗口调节参数，默认是0，可通过`org.quartz.scheduler.batchTriggerAcquisitionFireAheadTimeWindow`改写
- `misfireThreshold`： 超过这个时间还未触发的trigger,被认为发生了misfire,默认60s，可通过`org.quartz.jobStore.misfireThreshold`设置。

调度线程一次会拉取**NEXT_FIRE_TIME**小于（`now + idleWaitTime +batchTimeWindow`）,大于（`now - misfireThreshold`）的，`min(availThreadCount,maxBatchSize)`个triggers，默认情况下，会拉取未来30s，过去60s之间还未fire的1个trigger。随后将这些triggers的状态由**WAITING**改为**ACQUIRED**，并插入fired_triggers表。

### 触发trigger

首先，我们会检查每个trigger的状态是不是**ACQUIRED**，如果是，则将状态改为**EXECUTING**，然后更新trigger的**NEXT_FIRE_TIME**，如果这个trigger的**NEXT_FIRE_TIME**为空，也就是未来不再触发，就将其状态改为**COMPLETE**。如果trigger不允许并发执行（即Job的实现类标注了`@DisallowConcurrentExecution`），则将状态变为**BLOCKED**，否则就将状态改为**WAITING**。

### 包装trigger，丢给工作线程池

遍历triggers，如果其中某个trigger在第二步出错，即返回值里面有exception或者为null，就会做一些triggers表，fired_triggers表的内容修正，跳过这个trigger，继续检查下一个。否则，则根据trigger信息实例化`JobRunShell`（实现了Thread接口），同时依据`JOB_CLASS_NAME`实例化`Job`，随后我们将`JobRunShell`实例丢入工作线。

在`JobRunShell`的`run()`方法，Quartz会在执行`job.execute()`的前后通知之前绑定的监听器，如果`job.execute()`执行的过程中有异常抛出，则执行结果`jobExEx`会保存异常信息，反之如果没有异常抛出，则`jobExEx`为null。然后根据`jobExEx`的不同，得到不同的执行指令`instCode`。

`JobRunShell`将trigger信息，job信息和执行指令传给`triggeredJobComplete()`方法来完成最后的数据表更新操作。例如如果job执行过程有异常抛出，就将这个trigger状态变为**ERROR**，如果是**BLOCKED**状态，就将其变为**WAITING**等等，最后从fired_triggers表中删除这个已经执行完成的trigger。注意，这些是在工作线程池异步完成。






