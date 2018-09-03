---
layout: post
title:  "SpringBoot学习"
date:   2017-07-17 14:34:32
categories: Java
tags: Spring
author: miaoqi
---

* content
{:toc}


## HelloWorld

1. 创建一个maven工程;(jar) 

2. 导入spring boot相关的依赖

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
        
3. 编写一个主程序;启动Spring Boot应用
        
        @SpringBootApplication
        public class HelloWorldMainApplication{ 
            public static void main(String[] args) {
                // Spring应用启动起来
                SpringApplication.run(HelloWorldMainApplication.class,args);
            }
        }

4. 编写相关的Controller、Service

        @Controller
        public class HelloController{
        
            @ResponseBody
            @RequestMapping("/hello")
            public String hello(){
                return "Hello World!";
            }
        }
        
5. 运行主程序测试

6. 简化部署
        
        <build>
            <!‐‐ 这个插件，可以将应用打包成一个可执行的jar包;‐‐> <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring‐boot‐maven‐plugin</artifactId>
                </plugin>
            </plugins>
        </build>
将这个应用打成jar包，直接使用java -jar的命令进行执行;


## 探究HelloWorld

### POM文件

* 父项目

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

    以后我们导入依赖默认是不需要写版本;(没有在dependencies里面管理的依赖自然需要声明版本号)

* 启动器

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    spring-boot-starter-web: 
    
    spring-boot-starter:spring-boot场景启动器;帮我们导入了web模块正常运行所依赖的组件;
    
    Spring Boot将所有的功能场景都抽取出来，做成一个个的starters(启动器)，只需要在项目里面引入这些starter 相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

### 主程序

* @SpringBootApplication

    @SpringBootApplication: Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot 就应该运行这个类的main方法来启动SpringBoot应用;

        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @SpringBootConfiguration
        @EnableAutoConfiguration
        @ComponentScan(excludeFilters = {
              @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
              @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
        public @interface SpringBootApplication {

    * @SpringBootConfiguration
    
        标注在某个类上，表示这是一个Spring Boot的配置类;
    
        * @Configuration:配置类上来标注这个注解;
    
            配置类 ----- 配置文件;配置类也是容器中的一个组件;@Component

    * @EnableAutoConfiguration

        以前我们需要配置的东西，Spring Boot帮我们自动配置;
        
        @EnableAutoConfiguration告诉SpringBoot开启自动配置功能;这样自动配置才能生效;

            @AutoConfigurationPackage
            @Import(EnableAutoConfigurationImportSelector.class)
            public @interface EnableAutoConfiguration{

        * @AutoConfigurationPackage

            自动配置包

            **将主配置类(@SpringBootApplication标注的类)的所在包及下面所有子包里面的所有组件扫描到Spring容器;**

        * @Import(EnableAutoConfigurationImportSelector.class)

            Spring的底层注解@Import，给容器中导入一个组件;导入的组件由EnableAutoConfigurationImportSelector.class决定

            将所有需要导入的组件以全类名的方式返回;这些组件就会被添加到容器中;

            会给容器中导入非常多的自动配置类(xxxAutoConfiguration);就是给容器中导入这个场景需要的所有组件， 并配置好这些组件;

            Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将以前我们需要自己配置的东西, 都由SpringBoot来配置, J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar;


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
            
            animal: pets
            
            person:
                lastName: miaoqi
                age: 20

        也允许另一种写法，将所有键值对写成一个行内对象

            person: {lastName: Steve, age: 20}

    * 数组: 一组按次序排列的值，又称为序列(sequence)/列表(list), 一组连词线开头的行, 构成一个数组

        一组连词线开头的行，构成一个数组
            
            pets:
              - Cat
              - Dog
              - Goldfish

        行内写法

            pets: [Cat, Dog, Goldfish]

        数据结构的子成员是一个数组，则可以在该项下面缩进一个空格

            -
             - Cat
             - Dog
             - Goldfish

    * 纯量(scalars)：单个的、不可再分的值

        * 布尔值:

                布尔值用true和false表示

        * 数值:

                分为整数和浮点数

        * Null

            null用 ~ 表示

                parent: ~ 

        * 字符串

            字符串默认不使用引号表示
            
                str: 这是一行字符串

            如果字符串之中包含空格或特殊字符，需要放在引号之中

                str: '内容： 字符串'

            单引号和双引号都可以使用，双引号不会对特殊字符转义：

                s1: '内容\n字符串'
                s2: "内容\n字符串"

            单引号之中如果还有单引号，必须连续使用两个单引号转义

                str: 'labor''s day'

            字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格

                str: 这是一段
                  多行
                  字符串

            多行字符串可以使用“|”保留换行符，也可以使用>折叠换行

                this: |
                  Foo
                  Bar
                that: >
                  Foo
                  Bar

            +表示保留文字块末尾的换行，-表示删除字符串末尾的换行

                s1: |
                  Foo
                
                s2: |+
                  Foo
                 
                 
                s3: |-
                  Foo

                {s1: 'Foo\n', s2: 'Foo\n\n\n', s3: 'Foo'}
            
    * 复合结构, 以上三种类型的组合

            languages:
             - Ruby
             - Perl
             - Python
            websites:
             YAML: yaml.org
             Ruby: ruby-lang.org
             Python: python.org
             Perl: use.perl.org

### 编写配置文件

* 配置文件占位符

        person.lastName=哈哈哈哈哈${random.uuid}
        person.age=${random.int}
        person.birth=2018/08/20
        person.map.k1=v1
        person.map.k2=14
        person.list=a,b,c,d
        person.dog.name=${person.lastName:xxx}小狗
        person.dog.age=15

    ${random.uuid}, 生成随机uuid

    ${person.lastName:xxx}, 取上文中配置的person.lastName的属性值, 如果person.lastName不存在取xxx作为值

### 配置文件加载位置

1. 项目路径下/config/

1. 项目路径下

1. 类路径下/config/

1. 类路径下/

优先级从高到低, 高优先级的内容会覆盖低优先级的内容

### 绑定配置文件中的值

* yaml写法

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

* properties写法
    
        person.last-name=啊发发
        person.age=18
        person.birth=2018/08/20
        person.map.k1=v1
        person.map.k2=14
        person.list=a,b,c
        person.dog.age=15

* @ConfigurationProperties: 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定, prefix = "person": 配置文件中哪个下面的所有属性进行一一映射
    
    只有这个组件是容器中的组件, 才能有容器提供的@ConfigurationProperties功能
    
    默认从全局配置文件中获取
    
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

* 使用@Value注解手动注入值

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

        @PropertySource(value = {"classpath:person.properties"})

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

* 激活指定profile

    1. 在配置文件中指定激活哪个环境

            spring.profiles.active=dev

    1. 命令行方式激活

            java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod

        会覆盖配置文件中的设置

    1. 虚拟机参数

            -Dspring.profiles.active=prod

        -D是默认语法


## 自动配置原理

* SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

    @EnableAutoConfiguration 利用EnableAutoConfigurationImportSelector给容器中导入一些组件
    
    可以查看selectImports()方法的内容;

        List configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置


        SpringFactoriesLoader.loadFactoryNames()
        
        扫描所有jar包类路径下 META‐INF/spring.factories
        把扫描到的这些文件的内容包装成properties对象
        从properties中获取到EnableAutoConfiguration.class类(全类名)对应的值，然后把他们添加在容器中


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

    每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中;用他们来做自动配置;

* 以HttpEncodingAutoConfiguration(Http编码自动配置)为例解释自动配置原理;

        @Configuration // 表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
        @EnableConfigurationProperties(HttpEncodingProperties.class) // 启动指定类的ConfigurationProperties功能;将配置文件中对应的值和HttpEncodingProperties绑定起来;并把HttpEncodingProperties加入到ioc容器中
        @ConditionalOnWebApplication // Spring底层@Conditional注解(Spring注解版)，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效; 判断当前应用是否是web应用，如果是，当前配置类生效
        @ConditionalOnClass(CharacterEncodingFilter.class) // 判断当前项目有没有这个类CharacterEncodingFilter;SpringMVC中进行乱码解决的过滤器;
        @ConditionalOnProperty(prefix="spring.http.encoding",value="enabled",matchIfMissing= true) // 判断配置文件中是否存在某个配置 spring.http.encoding.enabled; matchIfMissing 如果不存在，判断也是成立的
        // 即使我们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的;
        publicclassHttpEncodingAutoConfiguration{
        
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


    **所有在配置文件中能配置的属性都是在xxxxProperties类中封装着;配置文件能配置什么就可以参照某个功能对应的这个属性类**

        @ConfigurationProperties(prefix = "spring.http.encoding") //从配置文件中获取指定的值和bean的属 性进行绑定
        publicclassHttpEncodingProperties{
        
            public static final Charset DEFAULT_CHARSET = Charset.forName("UTF‐8");

* 精髓:

    1. SpringBoot启动会加载大量的自动配置类

    1. 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类;

    1. 我们再来看这个自动配置类中到底配置了哪些组件;(只要我们要用的组件有，我们就不需要再来配置了)

    1. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这 些属性的值;
     

* @Conditional派生注解(Spring注解版原生的@Conditional作用)

    必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效, 加在类上就对类中所有的bean都生效
    
    |@Conditional|扩展注解作用(判断是否满足当前指定条件)|
    |-----|-----|    |@ConditionalOnJava|系统的java版本是否符合要求|    |@ConditionalOnBean|容器中存在指定Bean|    |@ConditionalOnMissingBean|容器中不存在指定Bean|    |@ConditionalOnExpression|满足SpEL表达式指定|    |@ConditionalOnClass|系统中有指定的类|    |@ConditionalOnMissingClass|系统中没有指定的类|    |@ConditionalOnSingleCandidate|容器中只有一个指定的Bean，或者这个Bean是首选Bean|    |@ConditionalOnProperty|系统中指定的属性是否有指定的值|    |@ConditionalOnResource|类路径下是否存在指定资源文件|    |@ConditionalOnWebApplication|当前是web环境|    |@ConditionalOnNotWebApplication|当前不是web环境|    |@ConditionalOnJndiJNDI|存在指定项|


## 日志

小张;开发一个大型系统:

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
    
        importorg.slf4j.Logger; 
        importorg.slf4j.LoggerFactory;
        
        publicclassHelloWorld{
        
            public static void main(String[] args) {
        
                Logger logger = LoggerFactory.getLogger(HelloWorld.class);
                logger.info("Hello World");
            }
        }

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

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring‐boot‐starter</artifactId>
    </dependency>

SpringBoot使用它来做日志功能;

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring‐boot‐starter‐logging</artifactId>
    </dependency>

![http://www.miaomiaoqi.cn/images/springboot/log3.png](http://www.miaomiaoqi.cn/images/springboot/log3.png)

总结: 

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录

2. SpringBoot也把其他的日志都替换成了slf4j+logback

3. 中间替换包

**SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可**

### 日志使用

* 默认配置

    SpringBoot默认帮我们配置好了日志;

        //记录器
        Logger logger = LoggerFactory.getLogger(getClass()); @Test
        public void contextLoads() {
            //System.out.println();
            //日志的级别;
            //由低到高 trace < debug < info < warn < error
            //可以调整输出的日志级别;日志就只会在这个级别以以后的高级别生效       logger.trace("这是trace日志...");
            logger.debug("这是debug日志..."); //SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别, root级别
            logger.info("这是info日志...");
            logger.warn("这是warn日志...");
            logger.error("这是error日志...");
        }

* 日志输出格式

        %d表示日期时间，
        %thread表示线程名，
        %‐5level:级别从左显示5个字符宽度
        %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
        %msg:日志消息，
        %n是换行符
    
        %d{yyyy‐MM‐dd HH:mm:ss.SSS} [%thread] %‐5level %logger{50} ‐ %msg%n

    SpringBoot修改日志的默认配置

        logging.level.com.miaoqi=trace
        
        #logging.path=
        # 不指定路径在当前项目下生成springboot.log日志
        # 可以指定完整的路径;
        #logging.file=G:/springboot.log
        
        # 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹;使用 spring.log 作为默认文件
        logging.path=/spring/log
        
        # 在控制台输出的日志的格式
        logging.pattern.console=%d{yyyy‐MM‐dd}[%thread]%‐5level%logger{50}‐%msg%n
        # 指定文件中日志输出的格式
        logging.pattern.file=%d{yyyy‐MM‐dd}===[%thread]===%‐5level===%logger{50}====%msg%n

* 指定配置

    给类路径下放上每个日志框架自己的配置文件即可;SpringBoot就不使用他默认配置的了

    |Logging System|Customization|
    |-----|-----|    |Logback|logback-spring.xml , logback-spring.groovy , logback.xml or logback.groovy|
    |Log4j2|log4j2-spring.xml or log4j2.xml|    |JDK (Java Util Logging)|logging.properties|

    logback.xml:直接就被日志框架识别了;

    logback-spring.xml:日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot 的高级Profile功能

        <springProfilename="staging">
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

* 切换日志框架

    可以按照slf4j的日志适配图，进行相关的切换; 

    slf4j+log4j的方式;

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

        <mvc:view‐controller path="/hello" view‐name="success"/>
        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/hello"/>
                <bean></bean>
            </mvc:interceptor>
        </mvc:interceptors>

    编写一个配置类(@Configuration)，是WebMvcConfigurerAdapter类型;**不能标注@EnableWebMvc;** 既保留了所有的自动配置，也能用我们扩展的配置;

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

    原理:
    
    1. WebMvcAutoConfiguration是SpringMVC的自动配置类
    1. 在做其他自动配置时会导入;@Import(EnableWebMvcConfiguration.class)
    1. 容器中所有的WebMvcConfigurer都会一起起作用
    1. 我们的配置类也会被调用; 效果:SpringMVC的自动配置和我们的扩展配置都会起作用

3. 全面接管SpringMVC

    SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置;所有的SpringMVC的自动配置都失效了

    我们需要在配置类中添加@EnableWebMvc即可;

        @EnableWebMvc
        @Configuration publicclassMyMvcConfigextendsWebMvcConfigurerAdapter{
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                // super.addViewControllers(registry);
                // 浏览器发送 /miaoqi 请求来到 success
                registry.addViewController("/miaoqi").setViewName("success");
            }
        }

    原理: 为什么自己标注@EnableWebMvc注解会导致SpringMVC自动配置就失效

    1. @EnableWebMvc的核心引入了DelegatingWebMvcConfiguration.class

            @Import(DelegatingWebMvcConfiguration.class)
            public@interfaceEnableWebMvc{

    2. DelegatingWebMvcConfiguration继承WebMvcConfigurationSupport
    
            @Configuration
            public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport{

    3. WebMvcAutoConfiguration中如果没有WebMvcConfigurationSupport才会进行自动配置, 然而@EnableWebMvc引入了该类, 导致自动配置失效

            @Configuration
            @ConditionalOnWebApplication
            @ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class})
            @ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
            @AutoConfigureOrder(-2147483638)
            @AutoConfigureAfter({DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class})
            public class WebMvcAutoConfiguration {

    4. @EnableWebMvc将WebMvcConfigurationSupport组件导入进来

    1. 导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能

    1. 最终导致自动配置失效

## 注册Servlet三大组件【Servlet、Filter、Listener】

* 由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件

* 注册三大组件

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


## 数据访问

1. 整合JDBC

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring‐boot‐starter‐jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql‐connector‐java</artifactId>
            <scope>runtime</scope>
        </dependency>

        spring:
          datasource:
            username: root
            password: 123456
            url: jdbc:mysql://192.168.15.22:3306/jdbc
            driver‐class‐name: com.mysql.jdbc.Driver

    默认是用org.apache.tomcat.jdbc.pool.DataSource作为数据源; 

    数据源的相关配置都在DataSourceProperties里面;

    自动配置原理: org.springframework.boot.autoconfigure.jdbc:

    1. 参考DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池;可以使用 spring.datasource.type指定自定义的数据源类型;
    2. SpringBoot默认可以支持

        org.apache.tomcat.jdbc.pool.DataSource、HikariDataSource、BasicDataSource

1. 整合Druid数据源

        @Configuration
        public class DruidConfig {
        
            @ConfigurationProperties(prefix = "spring.datasource")
            @Bean
            public DataSource druid() {
                return new DruidDataSource();
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

3. 整合MyBatis

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis‐spring‐boot‐starter</artifactId>
            <version>1.3.1</version>
        </dependency>

    * 注解版

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

        自定义MyBatis的配置规则;给容器中添加一个ConfigurationCustomizer;

            @org.springframework.context.annotation.Configuration
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

        扫描所有Mapper

            @MapperScan(basePackages = "com.miaoqi.springboot.mapper")
            @SpringBootApplication
            public class SpringBoot06DataMybatisApplication {
            
            	public static void main(String[] args) {
            		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
            	}
            }

    * 配置文件版

            mybatis:
              config-location: classpath:mybatis/mybatis-config.xml
              # 如果xml和mapper在同一个目录下, 可以不配该条配置
              mapper-locations: classpath:mybatis/mapper/*.xml

        编写xml, springboot配置文件中指定xml位置










