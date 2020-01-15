---
layout: post
title: "Spring注解学习"
categories: [Framework]
description:
keywords:
---

* content
{:toc}


## IOC

* @Configuration

    告诉Spring当前类是一个配置类等同于原来的配置文件

    ```java
    @Configuration
    public class MainConfig {
    
    }
    ```

* @Bean

    给容器中注册一个bean; 类型为返回值类型, id默认是方法名

    ```java
    @Bean("person")
    public Person person() {
        return new Person("list", 20);
    }
    ```

* @ComponentScan

    value: 指定要扫描的包, 会扫描包下所有的类

    excludeFilters = Filter[] 按照指定规则排除哪些组件

    includeFilters = Filter[] 按照指定规则只包含哪些组件, 需要配合useDefaultFilters = false属性, 因为@ComponentScan默认是全扫描

    * FilterType.ANNOTATION: 按照注解
    
        ```java
        排除Controller和Service注解
        @ComponentScan(value = "com.miaoqi", excludeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
        })
        
        只扫描Controller和Service
        @ComponentScan(value = "com.miaoqi", includeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
        }, useDefaultFilters = false)
        ```

    * FilterType.ASSIGNABLE_TYPE: 按照给定的类型
    
        ```java
        @ComponentScan(value = "com.miaoqi", includeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class}),
            @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BookDao.class)
        }, useDefaultFilters = false)
        
        BookDao又被重新包含进来了
        ```
    
    * FilterType.ASPECTJ: 使用ASPECTJ表达式, 不太常用

    * FilterType.REGEX: 使用正则表达式      
    
    * FilterType.CUSTOM: 使用自定义规则, 必须是TypeFilter的实现类
    
        ```java
        @ComponentScan(value = "com.miaoqi", includeFilters = {
            @Filter(type = FilterType.CUSTOM, classes = MyTypeFilter.class)
        }, useDefaultFilters = false)
        ```
        ```java
        public class MyTypeFilter implements TypeFilter {
        		/**
        			* metadataReader: 读取到的当前正在扫描的类的信息
              * metadataReaderFactory: 可以获取到其他任何类的信息
              *
              */
          	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
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
        ```

* @Scope

    Spring容器中的对象默认是单实例的, 可以通过该注解配置

    
    ```java
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
    ```
    
* @Lazy

    针对单实例bean起作用, 单实例bean默认在容器启动的时候创建对象, 配置该注解后, 容器启动时不创建对象, 第一次使用(获取)Bean创建对象并初始化, 实现懒加载
    
    ```java
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给容器中添加Person...");
        return new Person("李四", 25);
    }
    ```
    
* @Conditional({Condition})

    按照一定条件进行判断, 满足条件给容器中注册bean, 该注解也可以放在类上, 代表当前类中所有的bean都要进行条件判断

    ```java
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates", 62);
    }
    ```
    ```java
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
    ```
    
    

* @Import

    快速给容器中导入组件, 先导入的组件先生效

    * 直接使用要导入到容器中的组件, 容器中就会自动注册这个组件, id默认是全类名

        ```java
        @Import({Color.class, Red.class})
        public class MainConfig2 {
    }
        ```
    
    * ImportSelector接口: 返回需要导入的组件的全类名数组
      
        ```java
        @Configuration
        @Import({Color.class, Red.class, MyImportSelector.class})
        public class MainConfig2 {}
        ```
        ```java
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
        ```
        
        ```java
        import org.springframework.beans.factory.support.BeanDefinitionRegistry;
        import org.springframework.beans.factory.support.RootBeanDefinition;
        import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
        import org.springframework.core.type.AnnotationMetadata;
        
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
        ```
        
        

* FactoryBean

    Spring提供的FactoryBean(工厂Bean)
    
    ```java
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Import;
    
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
    ```
    
    


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

    ```java
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
    ```

* 实现InitializingBean接口(初始化), DisposableBean接口(销毁)

    ```java
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
    ```

* 可以使用JSR250, @PostConstructor(初始化), @PreDestroy(销毁)

    ```java
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
    ```

* BeanPostProcessor, 后置处理器, 在所有的bean初始化方法前后进行处理

    ```java
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
    ```
    
    Spring底层例如ApplicationContextAware为Bean注入applicationContext就是通过BeanProcessor实现的

## 属性赋值

* @Value

    * 基本数值

        ```java
        @Value("张三")
        private String name;
        ```

    * SpEL, #{20-2}

        ```java
        @Value("#{20-2}")
        private Integer age;
        ```

    * ${}取出配置文件中的值(在运行的环境变量中)

        ```java
        @Value("${person.nickName}")
        private String nickName;
        ```

    * 为静态属性赋值, static 会随着 class 的加载而加载, 所以不能使用 @Value 注解, 需要提供一个非静态的 set 方法

      ```java
      private static String name;
      
      @Value("${person.name}")
      public void setName(String name){
        Person.name = name;
      }
      ```

      

* PropertySource

    读取配置文件加载到运行环境变量中

    ```java
    @Configuration
    @PropertySource({"classpath:person.properties"})
    public class MainConfigOfPropertyValues {
        @Bean
        public Person person() {
            return new Person();
        }
    }
    ```

## 依赖注入(DI)

* Autowired

    1. 默认按照类型去容器中找到对应的组件 
    2. 如果找到多个相同类型的组件, 就会将属性的名称作为组件的id去容器中查找
    3. 使用@Qualifier注解就只会按照id去查找, 忽略类型
    4. 默认必须要找到对应的bean, 可以设置required=false属性修改这个规则
    5. 也可以使用@Primary注解, 在多个类型的时候首选注入哪个bean, 与属性名称无关, 配置这个注解不能配置@Qualifier

    装配的优先级: @Qualifier(配置该属性后续均不生效) > @Primary > 属性名称 > 默认型匹配

    * 标注在属性上

        ```java
        @Service
        public class BookService {
        
            @Autowired
            private BookDao bookDao;
        
            @Override
            public String toString() {
                return "BookService{bookDao=" + bookDao + "}";
            }
        }
        ```

    * 标注在方法上

        ```java
        @Autowired
        // 标注在方法上时, Spring容器创建对象时, 就会调用方法, 完成赋值
        // 方法使用的参数, 自定义类型的值会从IOC容器中获取
        public void setCar(Car car) {
            this.car = car;
        }
        ```

    * 标注在构造器上

        ```java
        // 构造器要用的参数, 也是从容器中获取
        @Autowired
        public Boss(Car car) {
            this.car = car;
            System.out.println("这是Boss的有参构造器...");
        }
        ```

    * 标注在参数上

        ```java
        public Boss(@Autowired Car car) {
            this.car = car;
            System.out.println("这是Boss的有参构造器...");
        }
        ```

* @Qualifier

    使用该注解就会按照指定的id去查找, 忽略类型

    ```java
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
    ```

* @Primary

    多类型匹配的情况下, 首选项是哪个bean

    ```java
    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLabel("2");
        return bookDao;
    }
    ```

* @Resource

    Java规范, 可以和@Autowired一样实现自动装配, 按照属性名称进行装配

* @Inject

    Java规范


* 自定义组件想使用Spring底层的一些组件(ApplicationContext, BeanFactory)

    自定义组件实现xxxAware接口, 在创建对象的时候会调用接口规定的方法, 注入相应的组件
    
    ```java
    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.BeanNameAware;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    import org.springframework.stereotype.Component;
    
    @Component
    public class Red implements ApplicationContextAware, BeanNameAware {
    
        private ApplicationContext applicationContext;
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            System.out.println("传入的ioc: " + applicationContext);
            this.applicationContext = applicationContext;
        }
    
        @Override
        public void setBeanName(String name) {
            System.out.println("当前bean的名字: " + name);
        }
    }
    ```
    
    

* Profile

    Spring提供的可以根据当前的环境, 动态激活和切换一系列组件的功能

    开发环境, 测试环境, 生产环境

    指定组件在哪个环境下才会被注册到容器中, 不指定@Profile的话任何情况下都能注册到容器中, 加了@Profile的bean, 只有这个环境被激活的时候才能注册到容器中, 默认启动参数是default环境

    ```java
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Profile;
    import org.springframework.context.annotation.PropertySource;
    
    import java.beans.PropertyVetoException;
    
    @Configuration
    @PropertySource("classpath:dbconfig.properties")
    public class MainConfigOfProfile {
    
        @Value("${db.username}")
        private String user;
    
        @Profile("dev")
        @Bean("devDataSource")
        public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
            ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
            comboPooledDataSource.setUser(user);
            comboPooledDataSource.setPassword(pwd);
            comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/taotao");
            comboPooledDataSource.setDriverClass("com.mysql.jdbc.Driver");
            return comboPooledDataSource;
        }
    
        @Profile("test")
        @Bean("testDataSource")
        public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
            ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
            comboPooledDataSource.setUser(user);
            comboPooledDataSource.setPassword(pwd);
            comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/pinyougoudb");
            comboPooledDataSource.setDriverClass("com.mysql.jdbc.Driver");
            return comboPooledDataSource;
        }
    }
    ```
    ```java
    public class ProfileTest {
        // 1. 使用命令行参数 -Dspring.profiles.active=test
        @Test
        public void test01() {
            AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
            String[] beanNamesForType = ctx.getBeanNamesForType(DataSource.class);
            for (String s : beanNamesForType) {
                System.out.println(s);
            }
            ctx.close();
        }
    }
    ```
    
    

## AOP

AOP: 面向切面编程, 原理是动态代理

1. 导入AOP模块, SpringAOP, (spring-aspects)

    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>4.3.12.RELEASE</version>
    </dependency>
    ```

2. 定义一个业务逻辑类(MathCalculator): 在业务逻辑运行的时候进行日志打印(方法之前, 方法之后, 方法出现异常)

    ```java
    public class MathCalculator {
        public int div(int i, int j){
            System.out.println("MathCalculator...div...");
            return i / j;
        }
    }
    ```

3. 定义一个日志切面类(LogAspects): 切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行

    * 前置通知(@Before): logStart: 在目标方法(div)运行之前运行    
    * 后置通知(@After): logEnd: 在目标方法(div)运行之后运行(无论方法正常结束还是异常结束)     
    * 返回通知(@AfterReturning): logReturn: 在目标方法(div)正常返回之后运行         
    * 异常通知(@AfterThrowing): logException: 在目标方法(div)出现异常以后运行
    * 环绕通知(@Around): 动态代理: 手动推进目标方法运行(joinPoint.proceed)

4. 给切面类的目标方法标注何时何地运行
5. 将切面类和业务逻辑类(目标方法所在类)都加入容器中
6. 告诉Spring哪个类是切面类(给切面类上加一个注解@Aspect)

    ```java
    // 告诉Spring当前类是一个切面类
    @Aspect
    public class LogAspects {
    
        // 抽取公共切入点表达式
        // 1. 本类引用 pointcut()
        // 2. 其他切面引用 com.miaoqi.aop.LogAspects.pointcut()
        @Pointcut("execution(public int com.miaoqi.aop.MathCalculator.*(..))")
        public void pointCut() {
        }
    
        // @Before在目标方法之前切入, 切入点表达式(指定在哪个方法切入)
        // @Before("public int com.miaoqi.aop.MathCalculator.*(..)")
        @Before("pointCut()")
        public void logStart(JoinPoint joinPoint) {
            Object[] args = joinPoint.getArgs();
            System.out.println("前置通知, " + joinPoint.getSignature().getName() + "开始...参数列表是: {" + Arrays.asList(args) + "}");
        }
    
        @After("pointCut()")
        public void logEnd(JoinPoint joinPoint) {
            System.out.println("后置通知, " + joinPoint.getSignature().getName() + "结束...");
        }
    
        // 指定result接收返回值
        @AfterReturning(value = "pointCut()", returning = "result")
        public void logReturn(Object result) {
            System.out.println("正常返回通知, 除法正常返回...运行结果: {" + result + "}");
        }
    
        @AfterThrowing(value = "pointCut()", throwing = "exception")
        public void logException(Exception exception) {
            System.out.println("异常返回通知, 除法异常返回...异常信息: {" + exception + "}");
        }
    }
    ```


7. 开启注解版AOP

    ```java
    @EnableAspectJAutoProxy
    @Configuration
    public class MainConfigOfAOP {
    
        // 业务逻辑类加入到容器中
        @Bean
        public MathCalculator mathCalculator() {
            return new MathCalculator();
        }
    
        // 切面类加入到容器中
        @Bean
        public LogAspects logAspects() {
            return new LogAspects();
        }
    }
    ```

## 声明式事物

1. 导入相关依赖

    数据源, 数据库驱动, Spring-jdbc模块

    ```xml
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.44</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>4.3.12.RELEASE</version>
    </dependency>
    ```


2. 配置数据源, JdbcTemplate(Spring提供的简化数据库操作的工具)操作数据

    ```java
    @EnableTransactionManagement
    @Configuration
    @ComponentScan({"com.miaoqi.tx"})
    public class MainConfigOfTx {
    
        @Bean
        public DataSource dataSource() throws PropertyVetoException {
            ComboPooledDataSource dataSource = new ComboPooledDataSource();
            dataSource.setUser("root");
            dataSource.setPassword("miaoqi");
            dataSource.setDriverClass("com.mysql.jdbc.Driver");
            dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mytest");
            return dataSource;
        }
    
        @Bean
        public JdbcTemplate jdbcTemplate() throws PropertyVetoException {
            // Spring对@Configuration类会特殊处理, 给容器中加组件的方法, 多次调用都只是从容器中查找组件
            JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
            return jdbcTemplate;
        }
    
        // 注册事务管理器到容器中
        @Bean
        public PlatformTransactionManager platformTransactionManager() throws PropertyVetoException {
            return new DataSourceTransactionManager(dataSource());
        }
    }
    ```

3. 给方法上标注@Transactional 表示当前方法是一个事物方法

    ```java
    @Service
    public class UserService {
    
        @Autowired
        private UserDao userDao;
    
        @Transactional
        public void insertUser() {
            userDao.insert();
            System.out.println("插入完成");
            int i = 10 / 0;
        }
    }
    
    @Repository
    public class UserDao {
    
        @Autowired
        private JdbcTemplate jdbcTemplate;
    
        public void insert() {
            String sql = "INSERT INTO tbl_user(username, age) VALUES(?, ?)";
            String username = UUID.randomUUID().toString().substring(0, 5);
            jdbcTemplate.update(sql, username, 19);
        }
    }
    ```

4. 配置@EnableTransactionManagement开启事物注解
5. 配置事物管理器




















