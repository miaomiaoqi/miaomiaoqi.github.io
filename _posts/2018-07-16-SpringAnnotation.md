---
layout: post
title:  "Spring注解学习"
date:   2017-07-16 14:34:32
categories: Java
tags: Spring
author: miaoqi
---

* content
{:toc}


## 核心容器注解

* @Configuration

    告诉Spring当前类是一个配置类等同于原来的配置文件

        @Configuration
        public class MainConfig {
        
        }

* @Bean

    给容器中注册一个bean; 类型为返回值类型, id默认是方法名

        @Bean("person")
        public Person person() {
            return new Person("list", 20);
        }

* @ComponentScan

    value: 指定要扫描的包, 会扫描包下所有的类

    excludeFilters = Filter[] 按照指定规则排除哪些组件

    includeFilters = Filter[] 按照指定规则只包含哪些组件, 需要配合useDefaultFilters = false属性, 因为@ComponentScan默认是全扫描

    * FilterType.ANNOTATION: 按照注解
    
            排除Controller和Service注解
            @ComponentScan(value = "com.miaoqi", excludeFilters = {
                @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
            })
    
            只扫描Controller和Service
            @ComponentScan(value = "com.miaoqi", includeFilters = {
                @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
            }, useDefaultFilters = false)
    
    * FilterType.ASSIGNABLE_TYPE: 按照给定的类型

            @ComponentScan(value = "com.miaoqi", includeFilters = {
                @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class}),
                @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BookDao.class)
            }, useDefaultFilters = false)

            BookDao又被重新包含进来了
    
    * FilterType.ASPECTJ: 使用ASPECTJ表达式, 不太常用
    
    * FilterType.REGEX: 使用正则表达式      
    
    * FilterType.CUSTOM: 使用自定义规则, 必须是TypeFilter的实现类

            @ComponentScan(value = "com.miaoqi", includeFilters = {
                @Filter(type = FilterType.CUSTOM, classes = MyTypeFilter.class)
            }, useDefaultFilters = false)


            public class MyTypeFilter implements TypeFilter {
                /**
                 * metadataReader: 读取到的当前正在扫描的类的信息
                 * metadataReaderFactory: 可以获取到其他任何类的信息
                 *
                 */
                public boolean match(MetadataReader metadataReader,
                        MetadataReaderFactory metadataReaderFactory) throws IOException {
                    // 获取当前类的注解信息
                    AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
                    // 获取当前类的信息
                    ClassMetadata classMetadata = metadataReader.getClassMetadata();
                    // 获取当前类的信息(类路径...)
                    Resource resource = metadataReader.getResource();
            
                    String className = classMetadata.getClassName();
                    System.out.println("--->" + className);
                    if (className.contains("er")) {
                        return true;
                    }
                    return false;
                }
            }

* @Scope

    Spring容器中的对象默认是单实例的, 可以通过该注解配置

         
        ConfigurableBeanFactory#SCOPE_PROTOTYPE
        ConfigurableBeanFactory#SCOPE_SINGLETON
        org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
        org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
         
        prototype: 多实例, ioc容器启动并不会去调用方法创建对象, 每次获取的时候才会调用方法创建对象
        singleton: 单实例(默认值), ioc容器启动会调用方法创建对象放到ioc容器中, 以后每次请求就是直接从容器中获取
        request: 同一次请求
        session: 同一个session创建一个实例

        @Scope("prototype")
        @Bean("person")
        public Person person() {
            System.out.println("给容器中添加Person...");
            return new Person("李四", 25);
        }
        
* @Lazy

    针对单实例bean起作用, 单实例bean默认在容器启动的时候创建对象, 配置该注解后, 容器启动时不创建对象, 第一次使用(获取)Bean创建对象并初始化, 实现懒加载
    
        @Lazy
        @Bean("person")
        public Person person() {
            System.out.println("给容器中添加Person...");
            return new Person("李四", 25);
        }
    
* @Conditional({Condition})

    按照一定条件进行判断, 满足条件给容器中注册bean, 该注解也可以放在类上, 代表当前类中所有的bean都要进行条件判断

        @Conditional({WindowsCondition.class})
        @Bean("bill")
        public Person person01() {
            return new Person("Bill Gates", 62);
        }


        public class WindowsCondition implements Condition {
            /**
             * ConditionContext: 判断条件能使用的上下文
             * AnnotatedTypeMetadata: 注释信息
             */
            public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
                // 是否mac系统
                // 1. 获取到IOC容器使用的bean工厂
                ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
                // 2. 获取类加载器
                ClassLoader classLoader = context.getClassLoader();
                // 3. 获取环境信息
                Environment environment = context.getEnvironment();
                // 4. 获取bean定义的注册类
                BeanDefinitionRegistry registry = context.getRegistry();
        
                if (environment.getProperty("os.name").contains("Windows")) {
                    return true;
                }
                return false;
            }
        }

* @Import

    快速给容器中导入组件

    * 直接使用要导入到容器中的组件, 容器中就会自动注册这个组件, id默认是全类名

            @Import({Color.class, Red.class})
            public class MainConfig2 {
            }

    * ImportSelector接口: 返回需要导入的组件的全类名数组
        
            @Configuration
            @Import({Color.class, Red.class, MyImportSelector.class})
            public class MainConfig2 {}


            public class MyImportSelector implements ImportSelector  {
                // 返回值就是要导入的组件的全类名
                // AnnotationMetadata: 当前标注@Import注解的类的所有信息
                @Override
                public String[] selectImports(AnnotationMetadata importingClassMetadata) {
                    return new String[]{"com.miaoqi.bean.Blue", "com.miaoqi.bean.Yellow"};
                }
            }

    * ImportBeanDefinitionRegistrar接口: 手动注册

            @Configuration
            @Import({Color.class, Red.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
            public class MainConfig2 {}


            public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
            /**
             * AnnotationMetadata: 当前类的注解信息
             * BeanDefinitionRegistry: bean定义的注册类
             *      把所有要添加到容器中的bean, 调用
             *      BeanDefinitionRegistry.registerBeanDefinition方法手动注册
             */
            @Override
            public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
                boolean red = registry.containsBeanDefinition("com.miaoqi.bean.Red");
                boolean blue = registry.containsBeanDefinition("com.miaoqi.bean.Blue");
                if (red && blue) {
                    // 指定bean定义信息, (Bean的类型..)
                    RootBeanDefinition bean = new RootBeanDefinition(RainBow.class);
                    // 注册一个Bean, 指定bean名称
                    registry.registerBeanDefinition("rainBow", bean);
                }
            }
        }
        
* FactoryBean

    Spring提供的FactoryBean(工厂Bean)


        // 创建一个Spring提供的工厂Bean, 用于创建其他Bean
        public class ColorFactoryBean implements FactoryBean<Color> {
            // 返回一个Color对象, 这个对象会添加到容器中
            @Override
            public Color getObject() throws Exception {
                System.out.println("ColorFactoryBean...");
                return new Color();
            }
        
            // bean类型
            @Override
            public Class<?> getObjectType() {
                return Color.class;
            }
        
            // 是否单例
            @Override
            public boolean isSingleton() {
                return true;
            }
        }

        @Configuration
        @Import({Color.class, Red.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
        public class MainConfig2 {
        
            @Bean
            public FactoryBean colorFactoryBean() {
                return new ColorFactoryBean();
            }
        }
        
        @Test
        public void testImport() {
            printBeans(ctx);
    
            // 从FactoryBean获取的是调用getObject创建的对象,
            // Bean的名字是FactoryBean的id
            Object b2 = ctx.getBean("colorFactoryBean");
            Object b3 = ctx.getBean("colorFactoryBean");
            System.out.println(b2 == b3);
            // 如果想获取获取FactoryBean本身就加个&
            Object b4 = ctx.getBean("&colorFactoryBean");
            System.out.println(b4.getClass());
        }


* 目前为止给容器中注册bean的方式有以下几种:

    * 包扫描 + 组件标注注解(@Controller/@Service/@Repository/@Component)

        有局限性, 只能注册进来自己写的带有组件标注的类, 第三方包中的类如果没有注解就注册不进来

    * @Bean

        导入的第三方包中的类可以注册进来

    * @Import

    * @FactoryBean

## Bean的生命周期

bean的生命周期: bean创建---初始化---销毁的过程

我们可以自定义初始化和销毁方法, 容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

初始化:

对象创建完成, 并赋值好, 调用初始化方法

销毁:

单实例: 容器关闭的时候

多实例: 容器不会管理这个bean, 需要手动调用销毁方法


* 指定init-method和destroy-method

        public class Car {
            public Car() {
                System.out.println("car constructor...");
            }
        
            public void init() {
                System.out.println("car...init...");
            }
        
            public void destroy() {
                System.out.println("car...destroy...");
            }
        }

        @Bean(initMethod = "init", destroyMethod = "destroy")
        public Car car() {
            return new Car();
        }

* 实现InitializingBean接口(初始化), DisposableBean接口(销毁)

        @Component
        public class Cat implements InitializingBean, DisposableBean {ss
            public Cat() {
                System.out.println("cat constructor...");
            }
            
            @Override
            public void afterPropertiesSet() throws Exception {
                System.out.println("cat...afterPropertiesSet...");
            }
            
            @Override
            public void destroy() throws Exception {
                System.out.println("cat...destroy...");
            }
        }

* 可以使用JSR250, @PostConstructor(初始化), @PreDestroy(销毁)

        @Component
        public class Dog {
            public Dog() {
                System.out.println("dog...constructor...");
            }
            
            @PostConstruct
            public void init() {
                System.out.println("dog...PostConstruct...");
            }
            
            @PreDestroy
            public void destory() {
                System.out.println("dog...PreDestroy...");
            }
        }

* BeanPostProcessor, 后置处理器, 在所有的bean初始化方法前后进行处理

        @Component
        public class MyBeanPostProcessor implements BeanPostProcessor {
            @Override
            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + beanName);
                return bean;
            }
            
            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("postProcessAfterInitialization..." + beanName + "=>" + beanName);
                return bean;
            }
        }

    Spring底层例如ApplicationContextAware为Bean注入applicationContext就是通过BeanProcessor实现的

## 属性赋值

* @Value

    * 基本数值

            @Value("张三")
            private String name;

    * SpEL, #{20-2}

            @Value("#{20-2}")
            private Integer age;

    * ${}取出配置文件中的值(在运行的环境变量中)

            @Value("${person.nickName}")
            private String nickName;

* PropertySource

    读取配置文件加载到运行环境变量中

        @Configuration
        @PropertySource({"classpath:person.properties"})
        public class MainConfigOfPropertyValues {
            @Bean
            public Person person() {
                return new Person();
            }
        }

## 依赖注入(DI)

* Autowired

    1. 默认按照类型去容器中找到对应的组件 
    2. 如果找到多个相同类型的组件, 就会将属性的名称作为组件的id去容器中查找
    3. 使用@Qualifier注解就只会按照id去查找, 忽略类型
    4. 默认必须要找到对应的bean, 可以设置required=false属性修改这个规则
    5. 也可以使用@Primary注解, 在多个类型的时候首选注入哪个bean, 与属性名称无关, 配置这个注解不能配置@Qualifier

    装配的优先级: @Qualifier(配置该属性后续均不生效) > @Primary > 属性名称 > 默认型匹配

    * 标注在属性上

            @Service
            public class BookService {
            
                @Autowired
                private BookDao bookDao;
            
                @Override
                public String toString() {
                    return "BookService{" +
                            "bookDao=" + bookDao +
                            '}';
                }
            }

    * 标注在方法上

            @Autowired
            // 标注在方法上时, Spring容器创建对象时, 就会调用方法, 完成赋值
            // 方法使用的参数, 自定义类型的值会从IOC容器中获取
            public void setCar(Car car) {
                this.car = car;
            }

    * 标注在构造器上

            // 构造器要用的参数, 也是从容器中获取
            @Autowired
            public Boss(Car car) {
                this.car = car;
                System.out.println("这是Boss的有参构造器...");
            }

    * 标注在参数上

            public Boss(@Autowired Car car) {
                this.car = car;
                System.out.println("这是Boss的有参构造器...");
            }

* @Qualifier

    使用该注解就会按照指定的id去查找, 忽略类型

        @Service
        public class BookService {
        
            @Qualifier("bookDao2")
            @Autowired
            private BookDao bookDao;
        
            @Override
            public String toString() {
                return "BookService{" +
                        "bookDao=" + bookDao +
                        '}';
            }
        }

* @Primary

    多类型匹配的情况下, 首选项是哪个bean

        @Primary
        @Bean("bookDao2")
        public BookDao bookDao() {
            BookDao bookDao = new BookDao();
            bookDao.setLabel("2");
            return bookDao;
        }

* @Resource

    Java规范, 可以和@Autowired一样实现自动装配, 按照属性名称进行装配

* @Inject

    Java规范

## 扩展原理


## Web




















