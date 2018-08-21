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

## Profile

Spring提供的对不同环境提供不同配置功能的支持, 可以通过激活配置快速切换环境

* 多Profile文件

    在主配置文件编写的时候, 文件名可以是 applicaiton-{profile}.properties/yml

    ![http://miaomiaoqi.github.io/images/springboot/profile.png](http://miaomiaoqi.github.io/images/springboot/profile.png)

* yml支持多文档块方式

* 激活指定profile




