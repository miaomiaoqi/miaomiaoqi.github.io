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


















