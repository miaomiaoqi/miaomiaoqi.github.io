---
layout: post
title:  "SpringBoot学习"
date:   2018-07-17 14:34:32
categories: Framework
tags: SpringBoot
author: miaoqi
---

* content
{:toc}
## HelloWorld

创建一个maven工程;(jar) 

导入spring boot相关的依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring‐boot‐starter‐web</artifactId>
    </dependency>
</dependencies>
```
编写一个主程序;启动Spring Boot应用

```java
@SpringBootApplication
public class HelloWorldMainApplication{ 
	public static void main(String[] args) {
		// Spring应用启动起来
		SpringApplication.run(HelloWorldMainApplication.class,args);
	}
}
```

编写相关的Controller、Service

```java
@Controller
public class HelloController{

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
  
}
```
运行主程序测试

简化部署

```xml
<build>
    <!‐‐ 这个插件，可以将应用打包成一个可执行的jar包;‐‐> <build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring‐boot‐maven‐plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

将这个应用打成jar包，直接使用java -jar的命令进行执行;


## 探究HelloWorld

### POM文件

* 父项目

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring‐boot‐starter‐parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    
    他的父项目是
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring‐boot‐dependencies</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath>../../spring‐boot‐dependencies</relativePath>
    </parent>
    他来真正管理Spring Boot应用里面的所有依赖版本;
    ```

    以后我们导入依赖默认是不需要写版本;(没有在dependencies里面管理的依赖自然需要声明版本号)

* 启动器

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
</dependency>
    ```
    
    spring-boot-starter-web: 
    
    spring-boot-starter:spring-boot场景启动器;帮我们导入了web模块正常运行所依赖的组件;
    
    Spring Boot将所有的功能场景都抽取出来，做成一个个的starters(启动器)，只需要在项目里面引入这些starter 相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

### 主程序

@SpringBootApplication

@SpringBootApplication: Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot 就应该运行这个类的main方法来启动SpringBoot应用;

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

* @SpringBootConfiguration

    标注在某个类上，表示这是一个Spring Boot的配置类;

    * @Configuration:配置类上来标注这个注解;

        配置类 ----- 配置文件;配置类也是容器中的一个组件;@Component

* @EnableAutoConfiguration

    以前我们需要配置的东西，Spring Boot帮我们自动配置;
    
    @EnableAutoConfiguration告诉SpringBoot开启自动配置功能;这样自动配置才能生效;

    ```java
    @AutoConfigurationPackage
    @Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration{}
    ```

    * @AutoConfigurationPackage

        自动配置包

        **将主配置类(@SpringBootApplication标注的类)的所在包及下面所有子包里面的所有组件扫描到Spring容器;**

    * @Import(EnableAutoConfigurationImportSelector.class)

        Spring的底层注解@Import，给容器中导入一个组件;导入的组件由EnableAutoConfigurationImportSelector.class决定

        将所有需要导入的组件以全类名的方式返回;这些组件就会被添加到容器中;

        会给容器中导入非常多的自动配置类(xxxAutoConfiguration);就是给容器中导入这个场景需要的所有组件， 并配置好这些组件;
    
    **Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将以前我们需要自己配置的东西, 都由SpringBoot来配置, J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar;**


## 配置文件

修改SpringBoot自动配置的默认值

SpringBoot默认使用两种类型的文件作为配置文件:

* application.properties

* application.yaml

### YAML

YAML 是专门用来写配置文件的语言，非常简洁和强大，远比 JSON 格式方便。

* 语法格式

    * 大小写敏感

    * 使用缩进表示层级关系

    * 缩进时不允许使用Tab键，只允许使用空格

    * **缩进的空格数目不重要，只要相同层级的元素左侧对齐即可**

    * \# 表示注释，从这个字符一直到行尾，都会被解析器忽略

* 数据结构

    * 对象: 键值对的集合, 又称为映射(mapping)/哈希(hashes)/字典(dictionary)

        对象的一组键值对，使用冒号结构表示
    
        ```yaml
        animal: pets
        person:
          lastName: miaoqi
          age: 20
        # 也允许另一种写法，将所有键值对写成一个行内对象
        person: {lastName: Steve, age: 20}
        ```
        
    * 数组: 一组按次序排列的值，又称为序列(sequence)/列表(list), 一组连词线开头的行, 构成一个数组
    
        一组连词线开头的行，构成一个数组
    
        ```yaml
    pets:
          - Cat
          - Dog
          - Goldfish
        ```
        
        行内写法
        
        ```yaml
        pets: [Cat, Dog, Goldfish]
        ```
    
      
    
* 纯量(scalars)：单个的、不可再分的值
  
    * 布尔值: 布尔值用true和false表示
    
    * 数值: 分为整数和浮点数
    
    * Null: null 用 ~ 表示
    
        ```yaml
    parent: ~ 
        ```
    
    * 字符串默认不使用引号表示
    
      ```yaml
    str: 这是一行字符串
        ```
        
        如果字符串之中包含空格或特殊字符，需要放在引号之中
        
        ```yaml
        str: '内容： 字符串'
        ```
        
        单引号和双引号都可以使用，双引号不会对特殊字符转义
        
        ~~~yaml
        s1: '内容\n字符串'
        s2: "内容\n字符串"
        # 单引号之中如果还有单引号，必须连续使用两个单引号转义
        str: 'labor''s day'
        # 字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格
        str: 这是一段
        多行
        字符串
        # 多行字符串可以使用“|”保留换行符，也可以使用>折叠换行
        this: |
        Foo
        Bar
        that: >
        Foo
        Bar
        # + 表示保留文字块末尾的换行，- 表示删除字符串末尾的换行
        s1: |
        Foo
        
        s2: |+
        Foo
        
        s3: |-
        Foo
        
        {s1: 'Foo\n', s2: 'Foo\n\n\n', s3: 'Foo'}
        ```
        ~~~

### 编写配置文件

* 配置文件占位符

    ```properties
    person.lastName=哈哈哈哈哈${random.uuid}
    person.age=${random.int}
    person.birth=2018/08/20
    person.map.k1=v1
    person.map.k2=14
    person.list=a,b,c,d
    person.dog.name=${person.lastName:xxx}小狗
person.dog.age=15
    ```

    ${random.uuid}, 生成随机uuid
    
    ${person.lastName:xxx}, 取上文中配置的person.lastName的属性值, 如果person.lastName不存在取xxx作为值

### 配置文件加载位置

1. 项目路径下/config/

1. 项目路径下

1. 类路径下/config/

1. 类路径下/

**优先级从高到低, 高优先级的内容会覆盖低优先级的内容**

### 绑定配置文件中的值

* yaml写法

    ```yaml
    person:
      lastName: zhangas
      age: 18
      boss: false
      birth: 2017/12/12
      map: {k1: v1, k2: 12}
      list:
        - lisi
        - wangwu
        - zhaoliu
      dog:
        name: 小狗
        age: 2
    ```

* properties写法
  
    ```properties
    person.last-name=啊发发
    person.age=18
    person.birth=2018/08/20
    person.map.k1=v1
    person.map.k2=14
    person.list=a,b,c
person.dog.age=15
  ```
  
* @ConfigurationProperties: 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定, prefix = "person": 配置文件中哪个下面的所有属性进行一一映射
  
    只有这个组件是容器中的组件, 才能有容器提供的@ConfigurationProperties功能
    
    默认从全局配置文件中获取
    
    ```java
    @Component
    @ConfigurationProperties(prefix = "person") // 从资源文件中自动装配属性值, 优先查找全局配置文件, 使用了@PropertySource就会查找指定配置文件
    public class Person {
        // @Value("${person.last-name}")
        private String lastName;
        // @Value("#{11*2}")
        private Integer age;
        // @Value("true")
        private Boolean boss;
        private Date birth;
    
        private Map<String, Object> map;
        private List<Object> list;
        private Dog dog;
        
        // Get和Set方法
}
  ```
  
* 使用@Value注解手动注入值

    ```java
    public class Person {
        @Value("${person.last-name}")
        private String lastName;
        @Value("#{11*2}")
        private Integer age;
        @Value("true")
        private Boolean boss;
        private Date birth;
    
        private Map<String, Object> map;
        private List<Object> list;
        private Dog dog;
    }
    ```


* @ConfigurationProperties和@Value的区别

    ||@ConfigurationProperties|@Value|
    |-----|-----|-----|
    |功能|批量注入配置文件中的值|单独指定|
    |松散绑定|支持|不支持|
    |SpEL|不支持|支持|
    |JSR303数据校验|支持|不支持|
    |复杂类型封装|支持|不支持|

* @PropertySource

    SpringBoot默认加载全局配置文件中的内容, 使用该注解加载指定的配置文件

    ```java
    @PropertySource(value = {"classpath:person.properties"})
    ```

* 总结

    @ConfigurationProperties自动装配和@Value手动装配默认匹配默认配置文件即application文件, 使用@PropertySource可以额外指定要加载的配置文件, @ConfigurationProperties, @Value, @PropertySource可以配合使用, 加载顺序如下

    1. 自动装配application.properties

    1. 自动装配application.yaml

    1. 自动装配自定义配置文件

    1. 手动装配application.properties

    1. 手动装配application.yaml

    1. 手动装配自定义配置文件

    后边不会覆盖前边的内容


## Profile

Spring提供的对不同环境提供不同配置功能的支持, 可以通过激活配置快速切换环境, 默认使用application.properties文件中的配置

* 多Profile文件

    在主配置文件编写的时候, 文件名可以是 applicaiton-{profile}.properties/yml

    ![http://www.miaomiaoqi.cn/images/springboot/profile.png](http://www.miaomiaoqi.cn/images/springboot/profile.png)

* yml支持多文档块方式

    ```yaml
    server:
      port: 8083
    spring:
      profiles:
        active: dev
    ---
    server:
      port: 8084
    spring:
      profiles: dev
    ---
    server:
      port: 8085
    spring:
      profiles: prod
    ```

* 激活指定profile

    1. 在配置文件中指定激活哪个环境

        ```properties
    spring.profiles.active=dev
        ```

    1. 命令行方式激活

        ```bash
    java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
        ```

        会覆盖配置文件中的设置

    1. 虚拟机参数
    
        ```bash
    -Dspring.profiles.active=prod
        ```
        
        -D是默认语法


## 自动配置原理

* SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

    @EnableAutoConfiguration 利用EnableAutoConfigurationImportSelector给容器中导入一些组件
    
    可以查看selectImports()方法的内容;

    ```java
    // 获取候选的配置
    List configurations = getCandidateConfigurations(annotationMetadata, attributes);
    ```
    
    ```
    SpringFactoriesLoader.loadFactoryNames()
        
    扫描所有jar包类路径下 META‐INF/spring.factories
    把扫描到的这些文件的内容包装成properties对象
    从properties中获取到EnableAutoConfiguration.class类(全类名)对应的值，然后把他们添加在容器中
    ```
    
    ```properties
    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
    org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
    org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
    org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
    org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
    org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
    org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
    org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
    org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
    org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
    org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
        org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
        org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
        org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
        org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
        org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
    org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
    org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
    org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
    org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
    org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
    org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
    org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
    org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
    org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
    org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
    org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
    org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
    org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
    org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
    org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
    org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
    org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
    org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
    org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
    org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
    org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
    org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
    org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
    org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
    org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
    org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
    org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
    org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
    org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
    org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
    org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
    org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
        org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
    org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
    org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
    org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
    org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
    org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
    org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
    org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
    org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
    org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
    org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
    org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
    org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
    org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
    org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
    org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
    org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
    org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
    org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
    org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
    org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
    org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
    org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
    org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
    org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
    ```
    
    **每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中;用他们来做自动配置**
    
* 以HttpEncodingAutoConfiguration(Http编码自动配置)为例解释自动配置原理;

    ```java
    import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
    import org.springframework.web.filter.CharacterEncodingFilter;
    
    @Configuration // 表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
    @EnableConfigurationProperties(HttpEncodingProperties.class) // 启动指定类的ConfigurationProperties功能;将配置文件中对应的值和HttpEncodingProperties绑定起来;并把HttpEncodingProperties加入到ioc容器中
    @ConditionalOnWebApplication // Spring底层@Conditional注解(Spring注解版)，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效; 判断当前应用是否是web应用，如果是，当前配置类生效
    @ConditionalOnClass(CharacterEncodingFilter.class) // 判断当前项目有没有这个类CharacterEncodingFilter;SpringMVC中进行乱码解决的过滤器;
    @ConditionalOnProperty(prefix="spring.http.encoding",value="enabled",matchIfMissing= true) // 判断配置文件中是否存在某个配置 spring.http.encoding.enabled; matchIfMissing 如果不存在，判断也是成立的
    // 即使我们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的;
    public class HttpEncodingAutoConfiguration{
    
        // 他已经和SpringBoot的配置文件映射了
        private final HttpEncodingProperties properties;
    
        // 只有一个有参构造器的情况下，参数的值就会从容器中拿
        public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
            this.properties = properties;
        }
    
        @Bean // 给容器中添加一个组件，这个组件的某些值需要从properties中获取
        @ConditionalOnMissingBean(CharacterEncodingFilter.class) // 判断容器没有这个组件?
        public CharacterEncodingFilter characterEncodingFilter() {
            CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
            filter.setEncoding(this.properties.getCharset().name());
            filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
            filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
            return filter;
        }
    }
    ```
    **所有在配置文件中能配置的属性都是在xxxxProperties类中封装着;配置文件能配置什么就可以参照某个功能对应的这个属性类**

    ```java
    @ConfigurationProperties(prefix = "spring.http.encoding") //从配置文件中获取指定的值和bean的属 性进行绑定
    public class HttpEncodingProperties{
        public static final Charset DEFAULT_CHARSET = Charset.forName("UTF‐8");
    }
    ```

* 精髓:

    1. SpringBoot启动会加载大量的自动配置类

    1. 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类;

    1. 我们再来看这个自动配置类中到底配置了哪些组件;(只要我们要用的组件有，我们就不需要再来配置了)

    1. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这 些属性的值;

* @Conditional派生注解(Spring注解版原生的@Conditional作用)

    必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效, 加在类上就对类中所有的bean都生效
    
    |@Conditional|扩展注解作用(判断是否满足当前指定条件)|
    |-----|-----|
    |@ConditionalOnJava|系统的java版本是否符合要求|
    |@ConditionalOnBean|容器中存在指定Bean, 及其子类 Bean|
    |@ConditionalOnMissingBean|容器中不存在指定Bean, 及其子类 Bean|
    |@ConditionalOnExpression|满足SpEL表达式指定|
    |@ConditionalOnClass|系统中有指定的类文件|
    |@ConditionalOnMissingClass|系统中没有指定的类|
    |@ConditionalOnSingleCandidate|容器中只有一个指定的Bean，或者这个Bean是首选Bean|
    |@ConditionalOnProperty|系统中指定的属性是否有指定的值|
    |@ConditionalOnResource|类路径下是否存在指定资源文件|
    |@ConditionalOnWebApplication|当前是web环境|
    |@ConditionalOnNotWebApplication|当前不是web环境|
    |@ConditionalOnJndiJNDI|存在指定项|


## 日志

小张开发一个大型系统:

1. System.out.println("");将关键数据打印在控制台;去掉?写在一个文件? 

2. 框架来记录系统的一些运行时信息;日志框架 ; zhanglogging.jar; 

3. 高大上的几个功能?异步模式?自动归档?xxxx? zhanglogging-good.jar? 

4. 将以前框架卸下来?换上新的框架，重新修改之前相关的API;zhanglogging-prefect.jar; 

5. JDBC---数据库驱动; 写了一个统一的接口层;日志门面(日志的一个抽象层);logging-abstract.jar; 给项目中导入具体的日志实现就行了;我们之前的日志框架都是实现的抽象层;


市面上的日志框架:

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

### 使用SLF4J

1. 如何在系统中使用SLF4J

    以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法; 给系统里面导入slf4j的jar和 logback的实现jar

    ```java
    import org.slf4j.Logger; 
    import org.slf4j.LoggerFactory;
    
    public class HelloWorld{
    
        public static void main(String[] args) {
    
            Logger logger = LoggerFactory.getLogger(HelloWorld.class);
            logger.info("Hello World");
        }
    }
    ```

    ![http://www.miaomiaoqi.cn/images/springboot/long1.png](http://www.miaomiaoqi.cn/images/springboot/log1.png)

    每一个日志的实现框架都有自己的配置文件。使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件

2. 遗留问题

    a(slf4j+logback): Spring(commons-logging)、Hibernate(jboss-logging)、MyBatis、xxxx统一日志记录，即使是别的框架和我一起统一使用slf4j进行输出?

    ![http://www.miaomiaoqi.cn/images/springboot/log2.png](http://www.miaomiaoqi.cn/images/springboot/log2.png)

    如何让系统中所有的日志都统一到slf4j
    
    1. **将系统中其他日志框架先排除出去**
    
    2. **用中间包来替换原有的日志框架**
    
    3. **我们导入slf4j其他的实现**

### SpringBoot的日志关系

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter</artifactId>
</dependency>
```

SpringBoot使用它来做日志功能;

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐logging</artifactId>
</dependency>
```

![http://www.miaomiaoqi.cn/images/springboot/log3.png](http://www.miaomiaoqi.cn/images/springboot/log3.png)

总结: 

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录

2. SpringBoot也把其他的日志都替换成了slf4j+logback

3. 中间替换包

**SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可**

### 日志使用

* 默认配置

    SpringBoot默认帮我们配置好了日志;

    ```java
    //记录器
    Logger logger = LoggerFactory.getLogger(getClass()); @Test
    public void contextLoads() {
        // 日志的级别由低到高 trace < debug < info < warn < error
        // 可以调整输出的日志级别;日志就只会在这个级别以以后的高级别生效       logger.trace("这是trace日志...");
        logger.debug("这是debug日志..."); //SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别, root级别
        logger.info("这是info日志...");
        logger.warn("这是warn日志...");
        logger.error("这是error日志...");
    }
    ```
    
* 日志输出格式

    ```
    %d表示日期时间，
    %thread表示线程名，
    %‐5level:级别从左显示5个字符宽度
    %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
    %msg:日志消息，
    %n是换行符
    
    %d{yyyy‐MM‐dd HH:mm:ss.SSS} [%thread] %‐5level %logger{50} ‐ %msg%n
    ```

    SpringBoot修改日志的默认配置

    ```properties
    logging.level.com.miaoqi=trace
    
    # logging.path=
    # 不指定路径在当前项目下生成springboot.log日志
    # 可以指定完整的路径;
    # logging.file=G:/springboot.log
    
    # 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹;使用 spring.log 作为默认文件logging.path=/spring/log
    
    # 在控制台输出的日志的格式
    logging.pattern.console=%d{yyyy‐MM‐dd}[%thread]%‐5level%logger{50}‐%msg%n
    # 指定文件中日志输出的格式
    logging.pattern.file=%d{yyyy‐MM‐dd}===[%thread]===%‐5level===%logger{50}====%msg%n
    ```

* 指定配置

    给类路径下放上每个日志框架自己的配置文件即可;SpringBoot就不使用他默认配置的了

    |Logging System|Customization|
    |-----|-----|
    |Logback|logback-spring.xml , logback-spring.groovy , logback.xml or logback.groovy|
    |Log4j2|log4j2-spring.xml or log4j2.xml|
    |JDK (Java Util Logging)|logging.properties|

    logback.xml:直接就被日志框架识别了;

    logback-spring.xml:日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot 的高级Profile功能

    ```xml
    <springProfile name="staging">
        <!‐‐ configuration to be enabled when the "staging" profile is active ‐‐> 
        可以指定某段配置只在某个环境下生效
    </springProfile>
    
    <layout class="ch.qos.logback.classic.PatternLayout">
        <springProfile name="dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
        <springProfile name="!dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
    </layout>
    ```

* 切换日志框架

    可以按照slf4j的日志适配图，进行相关的切换; 

    slf4j+log4j的方式;

    ```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring‐boot‐starter‐web</artifactId>
      <exclusions>
        <exclusion>
          <artifactId>logback‐classic</artifactId>
          <groupId>ch.qos.logback</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j‐over‐slf4j</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j‐log4j12</artifactId>
    </dependency>
    ```

## SpringMVC自动配置

1. Spring MVC auto-configuration

    Spring Boot 自动配置好了SpringMVC 以下是SpringBoot对SpringMVC的默认配置:(WebMvcAutoConfiguration)
    
    * Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
    
        * 自动配置了ViewResolver(视图解析器:根据方法的返回值得到视图对象(View)，视图对象决定如何 渲染(转发?重定向?))
    
        * ContentNegotiatingViewResolver:组合所有的视图解析器的
        
        * 如何定制:我们可以自己给容器中添加一个视图解析器;自动的将其组合进来;
    
    * Support for serving static resources, including support for WebJars (see below).静态资源文件夹路 径,webjars
    
    * Static index.html support. 静态首页访问
    
    * Custom Favicon support (see below). favicon.ico
    
    * 自动注册了 of Converter , GenericConverter , Formatter beans
    
        * Converter:转换器; public String hello(User user):类型转换使用Converter
    
        * Formatter 格式化器; 2017.12.17===Date;


2. 自己扩展SpringMVC

    ```xml
    <mvc:view‐controller path="/hello" view‐name="success"/>
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean></bean>
        </mvc:interceptor>
    </mvc:interceptors>
    ```

    编写一个配置类(@Configuration)，是WebMvcConfigurerAdapter类型;**不能标注@EnableWebMvc;** 既保留了所有的自动配置，也能用我们扩展的配置;

    ```java
    //使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
    @Configuration
    public class MyMvcConfig extends WebMvcConfigurerAdapter {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            // super.addViewControllers(registry);
            //浏览器发送 /miaoqi 请求来到 success
            registry.addViewController("/miaoqi").setViewName("success");
        }
        
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**").excludePathPatterns("/index.html")
                   .excludePathPatterns("/").excludePathPatterns("/user/login").excludePathPatterns("/hello")
                   .excludePathPatterns("/error/**");
        }
    }
    ```

    原理:

    1. WebMvcAutoConfiguration是SpringMVC的自动配置类
    1. 在做其他自动配置时会导入;@Import(EnableWebMvcConfiguration.class)
    1. 容器中所有的WebMvcConfigurer都会一起起作用
    1. 我们的配置类也会被调用; 效果:SpringMVC的自动配置和我们的扩展配置都会起作用

3. 全面接管SpringMVC

    SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置;所有的SpringMVC的自动配置都失效了

    我们需要在配置类中添加@EnableWebMvc即可;

    ```java
    @EnableWebMvc
    @Configuration
    public class MyMvcConfig extends WebMvcConfigurerAdapter {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            // super.addViewControllers(registry);
            // 浏览器发送 /miaoqi 请求来到 success
            registry.addViewController("/miaoqi").setViewName("success");
    }
    }
    ```
```
    
原理: 为什么自己标注@EnableWebMvc注解会导致SpringMVC自动配置就失效
    
    1. @EnableWebMvc的核心引入了DelegatingWebMvcConfiguration.class
    
        ```java
        @Import(DelegatingWebMvcConfiguration.class)
        public @interface EnableWebMvc{
        }
```

2. DelegatingWebMvcConfiguration继承WebMvcConfigurationSupport
   
        ```java
        @Configuration
        public class Delegating WebMvcConfiguration extends WebMvcConfigurationSupport{
        }
        ```
    
    
    
    3. WebMvcAutoConfiguration中如果没有WebMvcConfigurationSupport才会进行自动配置, 然而@EnableWebMvc引入了该类, 导致自动配置失效
    
        ```java
        @Configuration
        @ConditionalOnWebApplication
        @ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class})
        @ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
        @AutoConfigureOrder(-2147483638)
        @AutoConfigureAfter({DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class})
        public class WebMvcAutoConfiguration {
        }
        ```
    
    4. @EnableWebMvc将WebMvcConfigurationSupport组件导入进来
    
    5. 导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能
    
    6. 最终导致自动配置失效

## 注册 Web 三大组件【Servlet、Filter、Listener】

由于 SpringBoot 默认是以 jar 包的方式启动嵌入式的 Servlet 容器来启动 SpringBoot 的 web 应用，没有 web.xml 文件

注册三大组件

```java
@Configuration
public class MyServerConfig {

    // servlet
    @Bean
    public ServletRegistrationBean myServlet() {
        return new ServletRegistrationBean(new MyServlet(), "/myServlet");
    }

    // filter
    @Bean
    public FilterRegistrationBean myFilter() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new MyFilter());
        registrationBean.setUrlPatterns(Arrays.asList("/hello", "/myServlet"));
        return registrationBean;
    }

    // listener
    @Bean
    public ServletListenerRegistrationBean myListener() {
        ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(
                new MyListener());
        return registrationBean;
    }
}
```

### 注册 Filter

方式一: @ServletComponentScan 扫描 Filter 所在的包

```java
@WebFilter(filterName = "myFilter",urlPatterns = "/*")
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    }

    @Override
    public void destroy() {
    }
}

@SpringBootApplication
@EnableAutoConfiguration
@EnableWebMvc
@ServletComponentScan(basePackages = "com.fanyin.eghm")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(EghmApplication.class, args);
    }
}
```

方式二

```java
@Configuration
public class FilterConfig {
    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new MyFilter2());
        bean.addUrlPatterns("/*");
        return bean;
    }
}
```



## SpringTask

* 在企业级应用中，经常会制定一些“计划任务”，即在某个时间点做某件事情，核心是以时间为关注点，即在一个特定的时间点，系统执行指定的一个操作。常见的任务调度框架有Quartz和SpringTask等

* Cron表达式

    Cron表达式是一个字符串，字符串以5或6个空格隔开，分为6或7个域，每一个域代表一个含义，Cron有如下两种语法格式： 

    * Seconds Minutes Hours DayofMonth Month DayofWeek Year

    * Seconds Minutes Hours DayofMonth Month DayofWeek

    每一个域可出现的字符如下： 
    
    * Seconds:可出现", - * /"四个字符，有效范围为0-59的整数 
    
    * Minutes:可出现", - * /"四个字符，有效范围为0-59的整数 
    
    * Hours:可出现", - * /"四个字符，有效范围为0-23的整数 
    
    * DayofMonth:可出现", - * / ? L W C"八个字符，有效范围为1-31的整数 
    
    * Month:可出现", - * /"四个字符，有效范围为1-12的整数或JAN-DEc 
    
    * DayofWeek:可出现", - * / ? L C #"四个字符，有效范围为1-7的整数或SUN-SAT两个范围。1表示星期天，2表示星期一， 依次类推 
    
    * Year:可出现", - * /"四个字符，有效范围为1970-2099年
    
    每一个域都使用数字，但还可以出现如下特殊字符，它们的含义是： 
    
    * \*：表示匹配该域的任意值，假如在Minutes域使用\*, 即表示每分钟都会触发事件。
    
    * ?:只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和 DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13 13 15 20 * ?, 其中最后一位只能用？，而不能使用\*，如果使用*表示不管星期几都会触发，实际上并不是这样。 
    
    * -:表示范围，例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次 
    
    * /：表示起始时间开始触发，然后每隔固定时间触发一次，例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次. 
    
    * ,:表示列出枚举值值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。 
    
    * L:表示最后，只能出现在DayofWeek和DayofMonth域，如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。 
    
    * W: 表示有效工作日(周一到周五),只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一 到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份 
    
    * LW:这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。 
    
    * \#:用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。

* Cron举例

        0 0 10,14,16 * * ? 每天上午10点，下午2点，4点 
        0 0/30 9-17 * * ? 朝九晚五工作时间内每半小时 
        0 0 12 ? * WED 表示每个星期三中午12点 
        0 0 12 * * ? 每天中午12点触发 
        0 15 10 ? * * 每天上午10:15触发 
        0 15 10 * * ? 每天上午10:15触发 
        0 15 10 * * ? * 每天上午10:15触发 
        0 15 10 * * ? 2005 2005年的每天上午10:15触发 
        0 * 14 * * ? 在每天下午2点到下午2:59期间的每1分钟触发 
        0 0/5 14 * * ? 在每天下午2点到下午2:55期间的每5分钟触发 
        0 0/5 14,18 * * ? 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 
        0 0-5 14 * * ? 在每天下午2点到下午2:05期间的每1分钟触发 
        0 10,44 14 ? 3 WED 每年三月的星期三的下午2:10和2:44触发 
        0 15 10 ? * MON-FRI 周一至周五的上午10:15触发 
        0 15 10 15 * ? 每月15日上午10:15触发 
        0 15 10 L * ? 每月最后一日的上午10:15触发 
        0 15 10 ? * 6L 每月的最后一个星期五上午10:15触发 
        0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发 
        0 15 10 ? * 6#3 每月的第三个星期五上午10:15触发

* 在SpringBoot中使用SpringTask

    1. 开启任务注解

        ```java
        @EnableAsync // 开启异步注解
        @EnableScheduling // 开启定时任务注解
        @SpringBootApplication
        public class SpringBoot10TaskApplication {
            public static void main(String[] args) {
                SpringApplication.run(SpringBoot10TaskApplication.class, args);
            }
    }
        ```

    1. 编写定时任务类
    
        ```java
        @Service
        public class ScheduledService {
        
            /**
             * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）
             * 0 * * * * MON-FRI
             *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
             *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
             *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
             *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
             *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
             */
            /*
             * second(秒 0-59), minute(分 0-59), hour(时 0-23), day of month(日 1-31), month(月 1-12), day of week(周几 0-7或SUN-SAT)
             * 0 * * * * MON-FRI
             */
            //    @Scheduled(cron = "0 * * ? * MON-FRI")
            //    @Scheduled(cron = "0,1,2,3 * * ? * MON-FRI") // , 代表枚举
            //    @Scheduled(cron = "0-3 * * ? * MON-FRI") // - 代表区间
            @Scheduled(cron = "0/4 * * * * MON-FRI") // / 代表步长, 从0秒开始每4秒启动一次
            // ? 日和星期冲突匹配, 指定了一个的值, 另外一个就需要指定为?, 代表放弃
            // L 最后
            // W 工作日
            // # 星期 4#2 第二个星期4
            public void hello() {
                System.out.println("hello...");
            }
        }
        ```



## 自定义拦截器

### 编写拦截器

实现 HandlerInterceptor 接口, 在 java8 中有了默认方法, 实现这个接口只需要重写需要的方法即可

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }

}
```

**继承 HandlerInterceptorAdapter, HandlerInterceptorAdapter也是实现了 HandlerInterceptor 接口, 本质是一样的**

```java
public class LoginInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }

}
```

### 注册拦截器

在 SpringBoot2.0 之前可以继承 WebMvcConfigurerAdapter 重写方法进行拦截器的注册

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;

    @Autowired
    private CurrentUserMethodArgumentResolver currentUserMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        currentUserMethodArgumentResolver.setUserModelClass(Member.class);
        argumentResolvers.add(currentUserMethodArgumentResolver);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authorizationInterceptor);
    }

}
```

**但是根据官方标注WebMvcConfigurerAdapter过时了，因为java8接口具有默认实现，然后想通过继承WebMvcConfigurationSupport实现添加。**

```java
@Configuration
public class WebConfig extends WebMvcConfigurationSupport {

    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;

    @Autowired
    private CurrentUserMethodArgumentResolver currentUserMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        currentUserMethodArgumentResolver.setUserModelClass(Member.class);
        argumentResolvers.add(currentUserMethodArgumentResolver);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authorizationInterceptor);
    }

}
```

**出现的问题：静态资源的访问的问题，在静态资源的访问的过程中，如果继承了 WebMvconfigureSupport 的方法的时候，SpringBoot 中的自动的配置会失效。 @ConditionalOnMissingBean({WebMvcConfigurationSupport.class}) 表示的是在WebMvcConfigurationSupport 类被注册的时候，SpringMVC的自动的配置会失效，就需要你自己进行配置, 最终我们自己实现 WebMvcConfigurer 接口, 重写自己需要的方法即可**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;

    @Autowired
    private CurrentUserMethodArgumentResolver currentUserMethodArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        currentUserMethodArgumentResolver.setUserModelClass(Member.class);
        argumentResolvers.add(currentUserMethodArgumentResolver);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
      	// 1. 加入的顺序就是拦截器执行的顺序
      	// 2. 按顺序执行所有拦截器的 preHandle
      	// 3. 所有的 preHandle 执行完再执行全部 postHandle 最后是 afterHandle
        registry.addInterceptor(authorizationInterceptor).addPathPatterns("/**");
      	registry.addInterceptor(studentInterceptor).addPathPatterns("/**");
    }

}
```



## 数据访问

### 整合JDBC

引入 jdbc 相关 jar 包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql‐connector‐java</artifactId>
    <scope>runtime</scope>
</dependency>
```

配置数据源

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver‐class‐name: com.mysql.jdbc.Driver
```

默认是用 org.apache.tomcat.jdbc.pool.DataSource 作为数据库连接池; 

数据源的相关配置都在 **DataSourceProperties** 里面;

**自动配置原理: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;**

**在 DataSourceAutoConfiguration 中使用 Import 加载多种数据源组件**

```java
@Configuration
@Conditional(PooledDataSourceCondition.class)
@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
// @Import 是真正加载数据库连接池组件的代码, 先加载的组件先生效
@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
		DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
		DataSourceJmxConfiguration.class })
protected static class PooledDataSourceConfiguration {

}
```

**在到 DataSourceConfiguration 中查看源码, 以 Hikari 为例**

```java
@Configuration
// HikariDataSource 这个类要存在
@ConditionalOnClass(HikariDataSource.class)
// 容器中没有其他 DataSource 的实例
@ConditionalOnMissingBean(DataSource.class)
// 在配置文件中是否配置了 spring.datasource.type 并且值为 com.zaxxer.hikari.HikariDataSource, 如果没有配置这个属性也会匹配到
@ConditionalOnProperty(name = "spring.datasource.type",
        havingValue = "com.zaxxer.hikari.HikariDataSource", matchIfMissing = true)
// 满足以上 3 个条件, 该类就会作为配置类生效, 加载类中的方法生成 HikariDataSource 组件
// 根据 @Import 的顺序, 后加载的后配置, 就会发现容器中存在 DataSource 的组件, 就不会加载了
static class Hikari {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public HikariDataSource dataSource(DataSourceProperties properties) {
        HikariDataSource dataSource = createDataSource(properties,
                HikariDataSource.class);
        if (StringUtils.hasText(properties.getName())) {
            dataSource.setPoolName(properties.getName());
        }
        return dataSource;
    }

}
```

**参考DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池;可以使用 spring.datasource.type 指定自定义的数据源类型, 前提是 Spring 内置了这些数据源的自动配置才可以**

SpringBoot默认可以支持如下数据源, 各个版本支持的数据源不一样, 默认加载的顺序也不一样自行查看

```properties
org.apache.tomcat.jdbc.pool.DataSource, HikariDataSource, BasicDataSource, Dbcp2
```

### 整合Druid数据源

在 SpringBoot 的 `DataSourceConfiguration` 类中没有内置根据条件加载 Druid 数据源, 所以就不能通过 spring 的配置来改变数据源了, **早期时候 Druid 的 jar 包也没有提供自动装配, 这时候就需要我们手动创建数据源使用, 后期 Druid 提供了自动装配, 我们只需要写配置文件即可, 其他数据源类似**

**引入 Druid 数据源相关 jar 包**

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.21</version>
</dependency>
```

**早期版本手动配置 Druid 数据源**

```java
@Configuration
public class DruidConfig {

  	// 数据源的基础配置通过 @ConfigurationProperties 使配置文件内容与 bean 关联起来
  	// 因为这里返回的是接口, 一些 druid 的特殊配置没有 set 方法, 需要手动配置再返回给容器
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid() {
      	// 其他特有数据手动配置
        DruidDataSource dataSource = new DruidDataSource();
    		dataSource.setDruidInitialSize(5);
      	return dataSource;
    }

    // 配置Druid的监控
    // 1. 配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        bean.setInitParameters(initParams);
        return bean;
    }

    // 2. 配置一个监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```

**后期版本只需要修改配置文件即可实现自动装配**

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://192.168.0.128:3306/test?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: 123456
    driverClassName: com.mysql.cj.jdbc.Driver
    druid:
      # 连接池的配置信息
      # 初始化时建立物理连接的个数
      initial-size: 5
      # 连接池最小连接数
      min-idle: 5
      # 连接池最大连接数
      max-active: 30
      # 获取连接时最大等待时间，单位毫秒
      max-wait: 60000
      # 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
      test-while-idle: true
      # 既作为检测的间隔时间又作为testWhileIdel执行的依据
      time-between-connect-error-millis: 60000
      # 销毁线程时检测当前连接的最后活动时间和当前时间差大于该值时，关闭当前连接
      min-evictable-idle-time-millis: 30000
      # 用来检测连接是否有效的sql 必须是一个查询语句
      # mysql中为 select 'x'
      # oracle中为 select 1 from dual
      validation-query: select 'x'
      # 申请连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true
      test-on-borrow: false
      # 归还连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true
      test-on-return: false
      # 是否缓存preparedStatement,mysql5.5+建议开启
      pool-prepared-statements: true
      # 当值大于0时poolPreparedStatements会自动修改为true
      max-pool-prepared-statement-per-connection-size: 20
      # 合并多个DruidDataSource的监控数据
      use-global-data-source-stat: false
      # 配置扩展插件
      filters: stat,wall,slf4j
      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
      connect-properties: druid.stat.mergeSql=false;druid.stat.slowSqlMillis=5000
      # 定时输出统计信息到日志中，并每次输出日志会导致清零（reset）连接池相关的计数器。
      time-between-log-stats-millis: 300000
      # 配置DruidStatFilter
      web-stat-filter:
        enabled: true
        url-pattern: '/*'
        exclusions: '*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*'
      # 配置DruidStatViewServlet
      stat-view-servlet:
        # 是否启用StatViewServlet（监控页面）默认值为false（考虑到安全问题默认并未启动，如需启用建议设置密码或白名单以保障安全）
        enabled: true
        url-pattern: '/druid/*'
        # IP白名单(没有配置或者为空，则允许所有访问)
        allow: 127.0.0.1,192.168.0.1
        # IP黑名单 (存在共同时，deny优先于allow)
        deny: 192.168.0.128
        # 禁用HTML页面上的“Reset All”功能
        reset-enable: false
        # 登录名
        login-username: admin
        # 登录密码
        login-password: admin
```

**排除自动配置, 如果我们的项目不需要操作数据库, 可以排除数据源的自动装配**

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
```

### 整合MyBatis

**引入Mybatis整合SpringBoot所需的jar包**

```xml
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>1.3.1</version>
</dependency>
```

#### 注解版整合

@Mapper 注解需要 MyBatis-3.4.0 以上版本, 3.4.0以上的版本又会导致 MyBatis 早期的分页拦截器不可使用, 需要修改

```java
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id = #{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id = #{id}")
    public int deleteByDeptId(Integer id);

    @Options(useGeneratedKeys = true, keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update dept set departmentName = #{departmentName} where id = #{id}")
    public int updateDept(Department department);

}
```

自定义MyBatis的配置规则;给容器中添加一个ConfigurationCustomizer;

```java
@Configuration
public class MyBatisConfig{
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){
            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            } 
        };
    } 
}
```

#### 配置版整合

1. 扫描所有Mapper文件所在的包

    ```java
    @MapperScan(basePackages = "com.miaoqi.springboot.mapper")
    @SpringBootApplication
    public class SpringBoot06DataMybatisApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
    	}
    }
    ```

2. 编写接口对应的 xml 文件来编写 sql 语句

#### 整合配置文件

**修改 SpringBoot 配置文件, 这里的配置是原来 mybatis 的全局配置文件如果不需要可省略**

```yaml
mybatis:
  # 可以指定 mybatis 自己的配置文件, 如果不需要可以不配
  config-location: classpath:mybatis/mybatis-config.xml
  # 如果 xml 和 mapper 在同一个目录下, 可以不配该条配置
  mapper-locations: classpath:mybatis/mapper/*.xml
```



**我们一般在 maven 环境下整合 mybatis 此处就有一个问题会发生**

```
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)
```

**如果出现上面那个错误, 一般的原因是Mapper interface和xml文件的定义对应不上，需要检查包名，namespace，函数名称等能否对应上，需要比较细致的对比，我经常就是写错了一两个字母搞的很长时间找不到错误**

按以下步骤一一执行：

1. 检查 xml 文件所在的 package 名称是否和 interface 对应的 package 名称一一对应

2. 检查 xml 文件的 namespace 是否和 xml 文件的 package 名称一一对应

3. 检查函数名称能否对应上

4. 去掉 xml 文件中的中文注释

5. 随意在 xml 文件中加一个空格或者空行然后保存

**但是还会出现一个不属于上面任何一种情况的错误, 这个问题的发生的原因是, 如果我们将 mapper 对应的 xml 文件与 mapper 放在同一目录, 一般都是 src/main/java/ 下, 这样使用 maven 打包的时候, maven 是不会将 xml 打包进编译后的目录中的, (因为 maven 认为src/main/java 下只是 java 的源代码路径, 所以不会打包资源文件, 而将 xml, properties 等资源文件放在 src/main/resources 目录下是可以被打包进去的), 这个时候就需要我们在 pom.xml 文件中加入一条配置, 告诉 maven 将src/main/java 路径下的某个资源也打进 jar 包就可以了, 配置如下**

```xml
<build>
	<resources>
        <resource>
            <directory>src/main/java</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*Mapper.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

**如果配置了上边的插件会覆盖 maven 默认的加载路径, 导致 src/main/resources 路径下的资源加载不到, 所以还要添加另外的配置如下**

```xml
<build>
	<resources>
        <resource>
            <directory>src/main/java</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*Mapper.xml</include>
            </includes>
        </resource>
        <resource>
        	<directory>src/main/resources</directory>
        	<filtering>true</filtering>
        	<includes>
        		<include>**/*.xml</include>
        	</includes>
        </resource>
    </resources>
</build>
```

#### MyBatis整合多数据源

**实际开发中有可能一个项目需要连接多个库, 默认情况下 SpringBoot 使用默认的 SqlSessionFactory, 这时需要我们手动指定多 SqlSessionFactory 取代默认的配置, 需要分别创建 DataSource, SqlSessionFactory 并且手动关联, 就不能使用自动配置了**

```java
@Configuration
@MapperScan(value = "com.miaoqi.mysql.dc",sqlSessionFactoryRef = "dcSqlSessionFactory")
public class DcMySqlConfig {

    @Autowired
    private ResourceLoader resourceLoader = new DefaultResourceLoader();

    @Value("${spring.datasource.dc_url}")
    private String dbUrl;

    @Value("${spring.datasource.dc_username}")
    private String username;

    @Value("${spring.datasource.dc_password}")
    private String password;

    @Value("${spring.datasource.dc_driverClassName}")
    private String driverClassName;

    @Value("${spring.datasource.dc_initialSize}")
    private int initialSize;

    @Value("${spring.datasource.dc_minIdle}")
    private int minIdle;

    @Value("${spring.datasource.dc_maxActive}")
    private int maxActive;

    @Value("${spring.datasource.dc_maxWait}")
    private int maxWait;

    @Value("${spring.datasource.dc_timeBetweenEvictionRunsMillis}")
    private int timeBetweenEvictionRunsMillis;

    @Value("${spring.datasource.dc_minEvictableIdleTimeMillis}")
    private int minEvictableIdleTimeMillis;

    @Value("${spring.datasource.dc_validationQuery}")
    private String validationQuery;

    @Value("${spring.datasource.dc_testWhileIdle}")
    private boolean testWhileIdle;

    @Value("${spring.datasource.dc_testOnBorrow}")
    private boolean testOnBorrow;

    @Value("${spring.datasource.dc_testOnReturn}")
    private boolean testOnReturn;

    @Value("${spring.datasource.dc_filters}")
    private String filters;

    @Value("${spring.datasource.dc_logSlowSql}")
    private String logSlowSql;

    @Bean(name = "dcDataSource")
    public DataSource dataSource() throws Exception {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(dbUrl);
        datasource.setUsername(ConfigTools.decrypt(username));
        datasource.setPassword(ConfigTools.decrypt(password));
        datasource.setDriverClassName(driverClassName);
        datasource.setInitialSize(initialSize);
        datasource.setMinIdle(minIdle);
        datasource.setMaxActive(maxActive);
        datasource.setMaxWait(maxWait);
        datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        datasource.setValidationQuery(validationQuery);
        datasource.setTestWhileIdle(testWhileIdle);
        datasource.setTestOnBorrow(testOnBorrow);
        datasource.setTestOnReturn(testOnReturn);
        datasource.setFilters(filters);

        return datasource;
    }

    @Bean(name = "dcDataSourceTransactionManager")
    public DataSourceTransactionManager transactionManager() throws Exception {
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean(name = "dcSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource());
        factory.setVfs(SpringBootVFS.class);
        factory.setConfigLocation(this.resourceLoader.getResource("classpath:dc-mybatis-config.xml"));
        return factory.getObject();
    }
}
```

```java
@Configuration
@MapperScan(value = "com.miaoqi.mysql.mapper",sqlSessionFactoryRef = "rdsSqlSessionFactory")
public class MySqlConfig {
    
    @Autowired
    private ResourceLoader resourceLoader = new DefaultResourceLoader();

    @Value("${spring.datasource.url}")
    private String dbUrl;

    @Value("${spring.datasource.username}")
    private String username;

    @Value("${spring.datasource.password}")
    private String password;
    
    @Value("${spring.datasource.driverClassName}")
    private String driverClassName;

    @Value("${spring.datasource.initialSize}")
    private int initialSize;

    @Value("${spring.datasource.minIdle}")
    private int minIdle;
    
    @Value("${spring.datasource.maxActive}")
    private int maxActive;

    @Value("${spring.datasource.maxWait}")
    private int maxWait;

    @Value("${spring.datasource.timeBetweenEvictionRunsMillis}")
    private int timeBetweenEvictionRunsMillis;

    @Value("${spring.datasource.minEvictableIdleTimeMillis}")
    private int minEvictableIdleTimeMillis;

    @Value("${spring.datasource.validationQuery}")
    private String validationQuery;

    @Value("${spring.datasource.testWhileIdle}")
    private boolean testWhileIdle;

    @Value("${spring.datasource.testOnBorrow}")
    private boolean testOnBorrow;

    @Value("${spring.datasource.testOnReturn}")
    private boolean testOnReturn;

    @Value("${spring.datasource.filters}")
    private String filters;

    @Value("${spring.datasource.logSlowSql}")
    private String logSlowSql;
    
    @Primary
    @Bean(name = "rdsDataSource")
    public DataSource dataSource() throws Exception {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(dbUrl);
        datasource.setUsername(ConfigTools.decrypt(username));
        datasource.setPassword(ConfigTools.decrypt(password));
        datasource.setDriverClassName(driverClassName);
        datasource.setInitialSize(initialSize);
        datasource.setMinIdle(minIdle);
        datasource.setMaxActive(maxActive);
        datasource.setMaxWait(maxWait);
			  datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
		    datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        datasource.setValidationQuery(validationQuery);
        datasource.setTestWhileIdle(testWhileIdle);
        datasource.setTestOnBorrow(testOnBorrow);
        datasource.setTestOnReturn(testOnReturn);
        datasource.setFilters(filters);
    	  return datasource;
    }
    
    @Primary
    @Bean(name = "rdsDataSourceTransactionManager")
    public DataSourceTransactionManager transactionManager(DataSource dataSource) {
    		return new DataSourceTransactionManager(dataSource);
    }
    
    @Primary
    @Bean(name = "rdsSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
    	SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setVfs(SpringBootVFS.class);
        factory.setConfigLocation(this.resourceLoader.getResource("classpath:mybatis-config.xml"));
        return factory.getObject();
	    }
}
```

**手动配置了SqlSessionFactory, 从不同的DataSource中获取连接, 注入到不同的mapper中, 即可实现多数据源**



#### 通用 Mapper

通用 Mapper 的作者也为自己的插件编写了启动器，我们直接引入即可

**需要注意的是, 如果使用了通用 Mapper 那么我们的 @MapperScan 就需要使用 tk.mybatis 包下的注解, 而不是 MyBatis 的**

```xml
<!-- 通用mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.0.2</version>
</dependency>
```

不需要任何配置, 直接使用即可

```java
@Mapper
public interface UserMapper extends tk.mybatis.mapper.common.Mapper<User>{
}
```

通常会自己定义一个 BaseMapper 方便使用

```java
@RegisterMapper
public interface BaseMapper<T> extends Mapper<T>, IdListMapper<T, Long>, InsertListMapper<T> {
}
```



#### 分页助手启动器

引入分页助手相关 jar 包

```xml
<!-- 分页助手启动器 -->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.2.3</version>
</dependency>
```

代码中直接使用即可

```java
// 开始分页
PageHelper.startPage(page, rows);

PageInfo<Spu> pageInfo = new PageInfo<>(spus);
return pageInfo;
```

## AOP应用

### 全局异常处理器, 对Controller层加入异常通知, 新建全局异常处理器类

@ControllerAdvice 是 Spring 提供的注解, 改注解监听了所有 Controller 类抛出的异常

```java
/**
 * 全局异常捕获处理器
 * 1. 捕获返回json格式
 * 2. 捕获返回页面
 *
 * @author miaoqi
 * @date 2018/9/4
 */
@ControllerAdvice(basePackages = "com.miaoqi.controller")
public class GlobalExceptionHandler {
    
    // @ResponseBody 返回json格式
    // ModelAndView 返回页面
    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    public Map<String, Object> errorResult(RuntimeException e) {
        // 实际开发中, 将异常信息写到日志中(发邮件通知管理者)
        Map<String, Object> errorResultMap = new HashMap<>();
        errorResultMap.put("errorCode", "500");
        errorResultMap.put("errorMsg", "全局捕获异常系统错误: " + e);
        return errorResultMap;
    }
}
```
### 全局参数拦截

自定义切面类, 拦截所有的 Controller 方法进行参数的输出

```java
@Component
@Aspect
public class HttpRequestLogAOP {

    private static final Logger logger = LoggerFactory.getLogger(HttpRequestLogAOP.class);

    @Pointcut("execution(public * com.miaoqi.controller.*.*(..))")
    private void controllerAspect(){}

    public void preHandle(HttpServletRequest request) {
        String headers = getRequestHeaders(request);
        String params = getRequestParameter(request);
        logger.info("api request -> [url: {} {}], [headers --> {}] [params --> {}]", request.getMethod(), request.getRequestURL(), headers, params);
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object object) {
        String headers = getResponseHeaders(response);
        logger.info("api response -> [url: {} {}], [headers --> {}] [data --> {}]", request.getMethod(), request.getRequestURL(), headers, JSON.toJSONString(object));
    }

    // 环绕通知
    @Around("controllerAspect()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
        this.preHandle(request);
        Object object = pjp.proceed();
        HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getResponse();
        this.afterCompletion(request, response, object);
        return object;
    }

    /**
     * 解析 ResponseHeader 参数
     *
     * @param response
     */
    private String getResponseHeaders(HttpServletResponse response) {
        StringBuffer sb = new StringBuffer();
        try {
            Collection<String> headers = response.getHeaderNames();
            headers.forEach(s -> {
                if (!s.toLowerCase().contains("password")) {
                    sb.append(s + ":");
                    Collection<String> values = response.getHeaders(s);
                    values.forEach(v -> sb.append(v));
                    sb.append(",");
                }
            });
        } catch (Exception e) {
            logger.warn("获取请求 Headers 异常, 该异常不必惊慌 ", e);
        }
        return sb.toString();
    }

    /**
     * 解析 RequestHeader 参数
     *
     * @param request
     */
    private String getRequestHeaders(HttpServletRequest request) {
        StringBuffer sb = new StringBuffer();
        try {
            Enumeration<String> headers = request.getHeaderNames();
            while (headers.hasMoreElements()) {
                String header = headers.nextElement();
                //过滤掉密码
                if (header.toLowerCase().contains("password")) {
                    continue;
                }
                sb.append(header + ":");
                Enumeration<String> hvs = request.getHeaders(header);
                while (hvs.hasMoreElements()) {
                    sb.append(hvs.nextElement());
                }
                sb.append(", ");
            }
        } catch (Exception e) {
            logger.warn("获取请求 Headers 异常, 该异常不必惊慌 ", e);
        }
        return sb.toString();
    }

    /**
     * 解析 param 参数
     *
     * @param request
     * @return
     */
    private String getRequestParameter(HttpServletRequest request) {
        StringBuffer sb = new StringBuffer();
        try {
            Enumeration<String> enu = request.getParameterNames();
            while (enu.hasMoreElements()) {
                String paraName = enu.nextElement();
                //过滤掉密码
                if (paraName.toLowerCase().contains("password")) {
                    continue;
                }
                String[] arrs = request.getParameterValues(paraName);
                if (arrs != null && arrs.length > 1) {
                    sb.append(paraName).append("=");
                    for (String arr : arrs) {
                        sb.append(arr + ",");
                    }
                } else {
                    sb.append(paraName).append("=").append(request.getParameter(paraName) + ";");
                }
            }
        } catch (Exception e) {
            logger.warn("获取请求 Params 异常, 该异常不必惊慌 ", e);
        }
        return sb.toString();
    }   
}
```


### 全局响应拦截器

实现 ResponseBodyAdvice 接口, 对 RestController 接口的返回值进行统一处理

```java
@RestControllerAdvice
public class GlobalResponseHandler implements ResponseBodyAdvice<Object> {

    /**
     * 判断是否支持全局响应
     *
     * @author miaoqi
     * @date 2019-07-31
     * @param methodParameter
     * @param converterType
     * @return
     */
    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> converterType) {
        // methodParameter.getDeclaringClass() 拿到类声明, 判断类上是否有注解
        if (methodParameter.getDeclaringClass().isAnnotationPresent(IgnoreResponseAdvice.class)) {
            return false;
        }
        // methodParameter.getMethod() 拿到方法声明, 判断方法上是否有注解
        if (methodParameter.getMethod().isAnnotationPresent(IgnoreResponseAdvice.class)) {
            return false;
        }
        return true;
    }

    /**
     * 全局响应拦截器
     *
     * @author miaoqi
     * @date 2019-07-31
     * @param body
     * @param returnType
     * @param selectedContentType
     * @param selectedConverterType
     * @param request
     * @param serverHttpResponse
     * @return
     */
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType,
            Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request,
            ServerHttpResponse serverHttpResponse) {
        CommonResponse<Object> response = new CommonResponse<>(0, "");
        if (null == body) {
            return response;
        } else if (body instanceof CommonResponse) {
            response = (CommonResponse<Object>) body;
        } else {
            response.setData(body);
        }
        return response;
    }
}



/**
 * 自定义注解, 用来忽略全局响应处理
 *
 * @author miaoqi
 * @date 2019-07-31
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface IgnoreResponseAdvice {
}
```



### 自定义注解解析器

**自定义注解解析器的原理实际是还是利用了AOP功能**

首先定义一个LogAnnotation注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LogAnnotation {

}
```
**编写切面类, 重点就在于切面表达式的编写不是一个切入点表达式, 而是一个注解的表达式**

```java
@Component
@Aspect
public class ControllerLogAspect {

    private static final Logger logger = LoggerFactory.getLogger(ControllerLogAspect.class);

    @Autowired(required = false)
    HttpServletRequest request;

    @Pointcut("@annotation(com.miaoqi.aop.LogAnnotation)")
    public void aspect(){	}

    @Before("aspect()")
    public void before(JoinPoint joinPoint) {
        try {
            if (request != null) {
                Object rd = request.getAttribute("requestData");
                if (rd != null) {
                    RequestData requestData = (RequestData) rd;
                    String token = request.getHeader("Authorization");
                    if (StringUtils.isNotBlank(token)) {
                        token = token.substring(7);
                    }
                    CommonLogger.doCommonLogInfo(logger,requestData.getBody(), token);
                }
            }
        } catch (Exception e) {
            LoggerUtil.error(logger,e.getMessage(),e);
        }

    }

    @AfterReturning(value = "aspect()", returning = "rtv")
    public void after(JoinPoint joinPoint, Object rtv) {
        try {
            if (request != null) {
                Object rd = request.getAttribute("requestData");
                if (rd != null) {
                    RequestData requestData = (RequestData) rd;
                    CommonLogger.doCommonLogInfo(logger, requestData.getBody(), JSON.toJSONString(rtv));
                }
            }
        } catch (Exception e) {
            LoggerUtil.error(logger,e.getMessage(),e);
        }
    }

    @AfterThrowing(pointcut = "aspect()", throwing = "ex")
    public void afterThrow(JoinPoint joinPoint, Exception ex) {
        if (request != null) {
            Object rd = request.getAttribute("requestData");
            if (rd != null) {
                RequestData requestData = (RequestData) rd;
                CommonLogger.doCommonLogError(logger, requestData.getBody(), ex.toString());
            }
        }
    }
}
```


### 自定义缓存注解

新建缓存注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface XRedisCache {

    /**
     * 缓存前缀
     */
    RedisKeyPrefixEnum prefixEnum();

    /**
     * 缓存 key
     */
    String prefixKey() default "";

    /**
     * 缓存过期时间, 默认 1
     */
    int timeout() default 1;

    /**
     * 缓存时间单元, 默认小时
     */
    TimeUnit unit() default TimeUnit.HOURS;

}
```

新建缓存注解解析器

```java
/**
 * 从 redis 中获取缓存之, 如果未找到就查库
 *
 * @author miaoqi
 * @date 2019-07-17
 */
@Aspect
@Component
@Slf4j(topic = XLoggerUtil.DEFAULT_LOG_NAME)
public class XRedisCacheAspects {

    @Autowired
    private RedisTemplate redisTemplate;

    @Around("@annotation(cn.xdf.calc.aop.XRedisCache)")
    public Object cacheAdvice4XCache(ProceedingJoinPoint joinPoint) throws Throwable {
        BoundValueOperations<String, String> valueOps;
        String redisKey;

        Object result;
        try {
            // 获取 redis 中的缓存值
            redisKey = this.getRedisKey(joinPoint);
            valueOps = this.redisTemplate.boundValueOps(redisKey);
            String cacheData = valueOps.get();
            // 缓存命中
            if (!StringUtils.isEmpty(cacheData)) {
                // 转换为 jackson 的 javaType
                JavaType javaType = XJsonUtil.getJavaType(this.getMethod(joinPoint).getGenericReturnType());
                // 多级泛型反序列化
                return XJsonUtil.fromJson(cacheData, javaType);
            }
        } catch (Exception e) {
            log.error("query data from redis error", e);
        }

        // 缓存未命中, 执行目标方法获取返回值(查库)
        result = joinPoint.proceed();

        try {
            // 设置缓存
            redisKey = this.getRedisKey(joinPoint);
            int timeout = this.getRedisCacheAnnotation(joinPoint).timeout();
            TimeUnit unit = this.getRedisCacheAnnotation(joinPoint).unit();
            valueOps = this.redisTemplate.boundValueOps(redisKey);
            valueOps.set(XJsonUtil.toJson(result), timeout, unit);
        } catch (Exception e) {
            log.error("set data to redis error", e);
        }
        return result;
    }

    /**
     * 获取 redisKey
     *
     * @author miaoqi
     * @date 2019-07-17
     * @param joinPoint
     * @return
     */
    private String getRedisKey(ProceedingJoinPoint joinPoint) {
        XRedisCache cache = this.getRedisCacheAnnotation(joinPoint);
        RedisKeyPrefixEnum redisKeyPrefixEnum = cache.prefixEnum();
        String key = cache.prefixKey();
        return redisKeyPrefixEnum.of(key);
    }

    /**
     * 获取注解信息
     *
     * @author miaoqi
     * @date 2019-07-17
     * @param joinPoint
     * @return
     */
    private XRedisCache getRedisCacheAnnotation(ProceedingJoinPoint joinPoint) {
        // 获取注解信息
        return this.getMethod(joinPoint).getAnnotation(XRedisCache.class);
    }

    private Method getMethod(ProceedingJoinPoint joinPoint) {
        // 判断是否是方法签名
        Signature signature = joinPoint.getSignature();
        // 判断是否是方法签名
        if (!(signature instanceof MethodSignature)) {
            throw new RuntimeException("signature is not method signature");
        }
        // 获取方法签名
        MethodSignature methodSignature = (MethodSignature) (joinPoint.getSignature());
        return methodSignature.getMethod();
    }
}
```





### 自定义参数解析器

开发过程中, 在Controller层接收参数SpringMVC提供了@RequestParam进行参数解析, 但在一些特殊情况下这个注解的功能不够用, 比如后台返回页面时对id进行了AES加密操作, 前台传参时, id参数都是经过AES加密后的数据, 这时就需要自定义参数解析器进行解密后传入

Spring提供了自定义参数解析器的功能

1. 自定义两个注解, 该注解类似 @RequestParam

    AESRequestParam

    ```java
    @Target(value = ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface AESRequestParam {
    
        String value() default "";
    
        boolean required() default false;
    
    }
    ```

    DESRequestParam

    ```java
    @Target(value = ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface DESRequestParam {
    
    }
    ```

1. 自定义参数解析器实现HandlerMethodArgumentResolver接口, 该接口有两个方法需要实现, supprts方法用于判断是否满足解析条件, resolve方法真正进行解析

    ```java
    @Component
    public class AESHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
        @Override
        public boolean supportsParameter(MethodParameter parameter) {
            System.out.println("判断是否支持AES解密");
            // 获取方法名
            System.out.println(parameter.getMethod().getName());
            // 获取参数名称
            System.out.println(parameter.getParameterName());
            // 判断是否包含注解
            return parameter.hasParameterAnnotation(AESRequestParam.class);
        }
    
        @Override
        public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
            System.out.println("进到了AES参数解析器中");
            // 获取HttpServletRequest
            HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
            // 获取注解信息
            AESRequestParam requestParam = parameter.getParameterAnnotation(AESRequestParam.class);
            // 获取传递过来的值, 如果AESRequestParam指定了value属性按照value属性获取, 否则按照参数名获取
            String parameterValue;
            if (!StringUtils.isEmpty(requestParam.value())) {
                parameterValue = request.getParameter(requestParam.value());
            } else {
                parameterValue = request.getParameter(parameter.getParameterName());
            }
    
            // 此处模拟RquestParam的require属性
            if (requestParam.required() && Objects.isNull(parameterValue)) {
                throw new IllegalArgumentException("参数不合法");
            }
            // 模拟进行AES解密
            return this.deCrypt(parameterValue);
        }
    
        private Object deCrypt(String parameterValue) {
            return parameterValue + "jiemi";
        }
    }
    ```

1. 将参数解析器注入到容器中

    ```java
    @Configuration
    public class MyConfig extends WebMvcConfigurerAdapter {
    
        @Override
        public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
            argumentResolvers.add(new AESHandlerMethodArgumentResolver());
            argumentResolvers.add(new DESHandlerMethodArgumentResolver());
            super.addArgumentResolvers(argumentResolvers);
        }
    }
    ```

1. Controller层在入参上标明注解

    ```java
    @RestController
    @RequestMapping("/param")
    public class ParamParserController {
    
        @RequestMapping("/test")
        public String testParamParser(@AESRequestParam(value = "param", required = true) String param1, String param2) {
            System.out.println("------------------");
            System.out.println(param1);
            System.out.println(param2);
            return "success";
        }
    
    }
    ```

1. 浏览器中访问接口

        http://localhost:8089/param/test?param=aa&param2=ll

    第一次访问控制台打印如下

        判断是否支持AES解密
        testParamParser
        param1
        进到了AES参数解析器中
        判断是否支持AES解密
        testParamParser
        param2
        判断是否支持DES解密
        testParamParser
        param2
        ------------------
        aajiemi
        ll

    第二次访问控制台打印如下

        进到了AES参数解析器中
        ------------------
        vvvvjiemi
        ll

第一次访问接口, 会依次调用参数解析器根据supportsParameter方法的返回值判断是否进行参数解析, 如果返回为true, 则立刻进入到resolveArgument方法进行参数解析, 并将返回值赋值给标注的参数, 并且不会进入到后续的参数解析器中, 如果返回为false, 则进入到下一个参数解析器中继续判断. **每个参数都会执行一次参数解析器链**

第二次访问接口, 会根据之前supportsParameter方法的返回值直接进入resolveArgument方法中, 原因是SpringMVC对参数解析器加了缓存

```java
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
    // parameter是参数对象
    // 从缓存中, 根据参数对象获取参数解析器
    HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
	if (result == null) {
		// 解析器为空的话, 遍历所有的参数解析器
		for (HandlerMethodArgumentResolver methodArgumentResolver : this.argumentResolvers) {
            if (logger.isTraceEnabled()) {
                logger.trace("Testing if argument resolver [" + methodArgumentResolver + "] supports [" +
      parameter.getGenericParameterType() + "]");
            }
            // 根据supportsParameter方法的返回值判断是否支持解析
            if (methodArgumentResolver.supportsParameter(parameter)) {
                result = methodArgumentResolver;
                // 将这个参数解析器以参数对象为key, 参数解析器为value放入缓存中
                this.argumentResolverCache.put(parameter, result);
                break;
            }
        }
	}
	return result;
}
```

### AOP 执行顺序

**Filter 的优先级永远大于 Interceptor, Interceptor 优先级永远大于自定义 AOP**

**Filter 使用 FilterRegistrationBean 注册的优先级高于 @ServletComponentScan 扫描 @WebFilter 的, FilterRegistrationBean 注册可以设定 order, order 越小优先级越高, @WebFilter 无法控制顺序**

**Interceptor 的顺序就是注册时候的顺序, 先注册的先执行**

**自定义的 AOP 可以通过 `@Order(1)` 注解, 或者实现 `Ordered` 接口指明 AOP 的顺序, 值越小优先级越高**

#### 正常执行流程

`filter4...pre -> filter1...pre -> filter2...pre -> filter3...pre -> inteceptor1...pre -> inteceptor2...pre -> @Around1 -> @Before1 -> @Around2 -> @Before2 -> 目标方法 -> @Around2 -> @After2 -> @AfterReturning2 -> @Around1 -> @After1 -> @AfterReturning1 -> 全局响应处理 -> inteceptor2...post -> inteceptor1...post -> inteceptor2...after -> inteceptor1...after -> filter3...after -> filter2...after -> filter1...after -> filter4...after`

#### 异常执行流程

`filter4...pre -> filter1...pre -> filter2...pre -> filter3...pre -> inteceptor1...pre -> inteceptor2...pre -> @Around1 -> @Before1 -> @Around2 -> @Before2 -> 目标方法 -> @After2 -> @AfterThrowing2 -> @After1 -> @AfterThrowing1 -> 全局异常处理 -> inteceptor2...after -> inteceptor1...after -> filter3...after -> filter2...after -> filter1...after -> filter4...after`

**异常时 @Around 的后续不会执行, 不会执行全局响应, 拦截器的 post 方法不会执行**

## SpringBoot 整合 Swagger2

Swagger是一款RESTful接口的文档在线自动生成、功能测试功能框架。一个规范和完整的框架，用于生成、描述、调用和可视化RESTful风格的Web服务，加上swagger-ui，可以有很好的呈现。

当我们在后台的接口修改了后，swagger可以实现自动的更新，而不需要人为的维护这个接口进行测试。

### 引入 jar 包

我使用的是 2.8.0 版本, 在 2.9.2 版本有些问题, 所以降低了版本

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.8.0</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.8.0</version>
</dependency>
```

### 拦截器放开 Swagger 相关资源

如果使用到了拦截器, 需要对 Swagger 相关的资源进行放行, 以下都是需要放行的资源, 各版本略有不同, 可以自己尝试

`/swagger-ui.html`

`/v2/**`

`/swagger-resources/**`

`*swagger*`

`*api-docs*`

`*csrf*`

### 编写 Swagger 配置类

要使用 Swagger 要进行一些配置, 需要编写一个 Swagger 的配置类

```java
// 是否开启 swagger，正式环境一般是需要关闭的（避免不必要的漏洞暴露！），可根据 springboot 的多环境配置进行设置
// @ConditionalOnProperty(prefix = "swagger", value = "enabled", matchIfMissing = true)
@Configuration
@EnableSwagger2
public class SwaggerConfig {

		// swagger2的配置文件，这里可以配置swagger2的一些基本的内容，比如扫描的包等等
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                // 设置 swagger-ui.html 页面上的一些元素信息
                .apiInfo(this.apiInfo()).select()
                // 定义要扫描的路径, 可以扫描包, 类注解, 方法注解
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                // 定义要生成文档的 Api 的 url 路径规则
                .paths(PathSelectors.any()).build();
    }

  	// swagger2 的一些基本信息
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 标题
                .title("xx 服务接口文档")
                // 描述
                .description("xx 服务接口文档").termsOfServiceUrl("http://localhost:8080")
                // 文档版本
                .version("1.0").build();
    }

}
```

### 编写 Controller 类

Controller 就是我们平时编写的对外接口的类

```java
@RestController
// 这个注解是用在Controller类上面的，可以对Controller做必要的说明
@Api(value = "用户接口服务", tags = {"用户操作接口"})
public class UserController {

    // 创建线程安全的Map
    static Map<Integer, User> users = Collections.synchronizedMap(new HashMap<Integer, User>());

    /**
     * 根据ID查询用户
     *
     * @param id
     * @return
     */
    // 作用在具体的方法上，其实就是对一个具体的API的描述
    @ApiOperation(value = "获取用户详细信息", notes = "根据url的id来获取用户详细信息")
    // 作用在具体的方法上, 对 API 参数的描述
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Integer", paramType = "path")
    @RequestMapping(value = "user/{id}", method = RequestMethod.GET)
    public ResponseEntity<JsonResult> getUserById(@PathVariable(value = "id") Integer id) {
        JsonResult r = new JsonResult();
        try {
            User user = users.get(id);
            r.setResult(user);
            r.setStatus("ok");
        } catch (Exception e) {
            r.setResult(e.getClass().getName() + ":" + e.getMessage());
            r.setStatus("error");
            e.printStackTrace();
        }
        return ResponseEntity.ok(r);
    }

    /**
     * 查询用户列表
     *
     * @return
     */
    @ApiOperation(value = "分页获取用户列表", notes = "分页获取用户列表")
    @ApiImplicitParams({
      @ApiImplicitParam(name = "pageNum", value = "页码", required = false, dataType = "Integer", paramType = "query", defaultValue = "1"),
      @ApiImplicitParam(name = "pageSize", value = "每页记录数", required = false, dataType = "Integer", paramType = "query", defaultValue = "20")
    })
    @RequestMapping(value = "users", method = RequestMethod.GET)
    public ResponseEntity<JsonResult> getUserList(
            @RequestParam(value = "pageNum", required = false, defaultValue = "1") Integer pageNum,
            @RequestParam(value = "pageSize", required = false, defaultValue = "20") Integer pageSize) {
        JsonResult r = new JsonResult();
        try {
            List<User> userList = new ArrayList<User>(users.values());
            r.setResult(userList);
            r.setStatus("ok");
        } catch (Exception e) {
            r.setResult(e.getClass().getName() + ":" + e.getMessage());
            r.setStatus("error");
            e.printStackTrace();
        }
        return ResponseEntity.ok(r);
    }

    /**
     * 添加用户
     *
     * @param user
     * @return
     */
    @ApiOperation(value = "创建用户", notes = "根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataTypeClass = User.class, paramType = "body")
    @RequestMapping(value = "user", method = RequestMethod.POST)
    public ResponseEntity<JsonResult> add(@RequestBody User user) {
        JsonResult r = new JsonResult();
        try {
            users.put(user.getId(), user);
            r.setResult(user.getId());
            r.setStatus("ok");
        } catch (Exception e) {
            r.setResult(e.getClass().getName() + ":" + e.getMessage());
            r.setStatus("error");

            e.printStackTrace();
        }
        return ResponseEntity.ok(r);
    }

    /**
     * 根据id删除用户
     *
     * @param id
     * @return
     */
    @ApiOperation(value = "删除用户", notes = "根据url的id来指定删除用户")
    // 路径传参一定要指定 paramType="path"
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long", paramType = "path")
    @RequestMapping(value = "user/{id}", method = RequestMethod.DELETE)
    public ResponseEntity<JsonResult> delete(@PathVariable(value = "id") Integer id) {
        JsonResult r = new JsonResult();
        try {
            users.remove(id);
            r.setResult(id);
            r.setStatus("ok");
        } catch (Exception e) {
            r.setResult(e.getClass().getName() + ":" + e.getMessage());
            r.setStatus("error");

            e.printStackTrace();
        }
        return ResponseEntity.ok(r);
    }

    /**
     * 根据id修改用户信息
     *
     * @param user
     * @return
     */
    @ApiOperation(value = "更新信息", notes = "根据url的id来指定更新用户信息")
    @ApiImplicitParams({
      // 路径传参一定要指定 paramType="path"
      @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long", paramType = "path"), 
      @ApiImplicitParam(name = "user", value = "用户实体user", required = true, dataTypeClass = User.class)
    })
    @RequestMapping(value = "user/{id}", method = RequestMethod.PUT)
    public ResponseEntity<JsonResult> update(@PathVariable("id") Integer id, @RequestBody User user) {
        JsonResult r = new JsonResult();
        try {
            User u = users.get(id);
            u.setUsername(user.getUsername());
            u.setAge(user.getAge());
            users.put(id, u);
            r.setResult(u);
            r.setStatus("ok");
        } catch (Exception e) {
            r.setResult(e.getClass().getName() + ":" + e.getMessage());
            r.setStatus("error");

            e.printStackTrace();
        }
        return ResponseEntity.ok(r);
    }

    // 使用该注解忽略这个API
    @ApiIgnore
    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    public String jsonTest() {
        return " hi you!";
    }

}
```

### 编写实体类与响应类

实体类 User 对象

```java
public class User {

		// 实体的属性
    @ApiModelProperty(value = "订单ID", example = "123")
    private int id = 1;
    @ApiModelProperty(value = "用户名", example = "xxxx")
    private String username;
    @ApiModelProperty(value = "年龄", example = "13")
    private int age = 1;
    private Date ctm;

    // Getter And Setter

}
```

编写响应类

```java
// ApiModel 可以修饰响应对象
@ApiModel("返回对象")
public class JsonResult {

    @ApiModelProperty("状态吗")
    private String status = null;

    @ApiModelProperty("返回结果")
    private Object result = null;

    // Getter Setter

}
```



### 访问 Swgger 页面

启动项目, 在地址栏中输入 `http://localhost:8080/swagger-ui.html` 即可访问 swagger 页面

### 相关注解

|      作用      |     Annotation     |            使用位置             |
| :------------: | :----------------: | :-----------------------------: |
|  描述对象属性  | @ApiModelProperty  |    用在出入参数对象的字段上     |
|   协议集描述   |        @Api        |       用在 Controller 上        |
|    协议描述    |   @ApiOperation    |    用在 Controller 的方法上     |
|  Response 集   |   @ApiResponses    |    用在 Controller 的方法上     |
|    Response    |    @ApiResponse    |   用在@ApiResponses 或方法上    |
|  非对象参数集  | @ApiImplicitParams |    用在 Controller 的方法上     |
| 非对象参数描述 | @ApiImplicitParam  | 用在@ApiImplicitParams 或方法上 |
|    响应对象    |     @ApiModel      |        用在返回对象类上         |

