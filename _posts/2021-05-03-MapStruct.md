---
layout: post
title: MapStruct
categories: [Java]
description: 
keywords: 
---


* content
{:toc}


## MapStruct 是什么? 

### JavaBean 的困扰

对于代码中 `JavaBean`之间的转换, 一直是困扰我很久的事情. 

在开发的时候我看到业务代码之间有很多的 `JavaBean` 之间的相互转化, 非常的影响观感, 却又不得不存在. 我后来想的一个办法就是通过反射, 或者自己写很多的转换器. 

第一种通过反射的方法确实比较方便, 但是现在无论是 `BeanUtils`, `BeanCopier` 等在使用反射的时候都会影响到性能. 虽然我们可以进行反射信息的缓存来提高性能. 

但是像这种的话, 需要类型和名称都一样才会进行映射, 有很多时候, 由于不同的团队之间使用的名词不一样, 还是需要很多的手动 set/get 等功能. 

第二种的话就是会很浪费时间, 而且在添加新的字段的时候也要进行方法的修改. 不过, 由于不需要进行反射, 其性能是很高的. 

### MapStruct 带来的改变

`MapSturct` 是一个生成类型安全, 高性能且无依赖的 JavaBean 映射代码的注解处理器(annotation processor). 

**抓一下重点：**

1. 注解处理器
2. 可以生成 `JavaBean` 之间那的映射代码
3. 类型安全, 高性能, 无依赖性

从字面的理解, 我们可以知道, 该工具可以帮我们实现 `JavaBean` 之间的转换, 通过注解的方式. 

同时, 作为一个工具类, 相比于手写, 其应该具有便捷, 不容易出错的特点. 



我们的故事要从一个风和日丽的下午开始说起!

这天, 外包韩在位置上写代码～外包韩根据如下定义

- PO(persistant object):持久化对象, 可以看成是与数据库中的表相映射的 java 对象. 最简单的 PO 就是对应数据库中某个表中的一条记录. 
- **VO(view object)**:视图对象, 用于展示层, 它的作用是把某个指定页面（或组件）的所有数据封装起来. 
- **BO(business object)**:业务对象, 主要作用是把业务逻辑封装为一个对象. 这个对象可以包括一个或多个其它的对象. 
- **DTO、DO(省略......)**

将Bean进行逐一分类! 例如一个car_tb的表, 于是他有了两个类, 一个叫CarPo, 里头属性和表字段完全一致. 另一个叫CarVo,用于页面上的Car显示! 但是外包韩在做CarPo到CarVo转换的时候, 代码是这么写的, 伪代码如下:

```java
CarPo carPo = this.carDao.selectById(1L);
CarVo carVo = new CarVo();
carVo.setId(carPo.getId());
carVo.setName(carPo.getName());
//省略一堆
return carVo;
```

*画外音:*看到这一串代码是不是特别亲切, 我接手过一堆外包留下的代码, 就是这么写的, 一坨屎山! 一类几千行, 一半都在set属性. 

恰巧, 阿雄打水路过! 鸡贼的阿雄瞄了一眼外包韩的屏幕, 看到外包韩的这一系列代码! 上去进行一顿教育, 觉得不够优雅! 阿雄觉得, 应该用`BeanUtils.copyProperties`来简化书写, 像下面这样! 

```java
CarPo carPo = this.carDao.selectById(1L);
CarVo carVo = new CarVo();
BeanUtils.copyProperties(carPo, carVo);
return carVo;
```

可是, 外包韩盯着这段代码, 说道:"网上不是说反射效率慢, 你这么写, 没有性能问题么？" 阿雄说道:" 如果是用Apache的BeanUtil类, 确实有很大的性能问题, 像阿里巴巴的代码扫描插件, 都禁止用该类, 如下所示! "

![https://www.miaomiaoqi.github.io/images/java/m_1.png](https://www.miaomiaoqi.github.io/images/java/m_1.png)

"但是, 如果采用的是像Spring的BeanUtils类, 要在调用次数足够多的时候, 你才能明显的感受到卡顿. "阿雄补充道. 

"哇, 阿雄真棒! "外包韩兴奋不已! 

看着这办公室基情满满的氛围. 一旁正在拖地的清洁工------**扫地烟**, 他决定不再沉默. 

只见扫地烟扔掉手中的拖把, 得瑟的说道"我们不考虑性能. 从拓展性角度看看! BeanUtils还是有很多问题的! "

- 复制对象时字段类型不一致, 导致赋值不上, 你怎么解决？自己拓展？
- 复制对象时字段名称不一致, 例如CarPo里叫carName, CarVo里叫name, 导致赋值不上, 你怎么解决？自己拓展？
- 如果是集合类的复制, 例如List转换为List,你怎么处理？(省略一万字....)

"那应该怎么办呢？"听了扫地烟的描述, 外包韩疑惑的问道! 

"很简单, 其实我们在转换bean的过程中, set这些逻辑是固定的, 唯一变化的就是转换规则. 因此, 如果我们只需要书写转换规则, 转换代码由系统根据规则自动生成, 就方便很多了! 还是用上面的例子, CarPo里叫carName, CarVo里叫name, 属性名称不一致. 我们就通过一个注解

```java
@Mapping(source = "carName", target = "name"), 
```

指定对应转换规则. 系统识别到这个注解, 就会生成代码

```java
carVo.setName(carPo.getCarName())
```

如果能以这样的方式, set代码由系统自动生成, 那么在bean转换逻辑方面, 灵活性将大大加强, 而且这种方式不存在性能问题!"扫地烟补充道! 

"那这些set逻辑, 由什么工具来生成呢？"外包韩和阿雄一起问道! 

"工具的名字叫**MapStruct**! "

ok, 上面的故事到了这里, 就结束了! 不需要问结局, 结局只有一个, 外包韩和阿雄幸福美满的...(省略10000字)... 那么我们开始具体来说一说**MapStruct**! 



## MapStruct的教程

### 用法

引入pom文件如下

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <!-- jdk8以下就使用mapstruct -->
    <artifactId>mapstruct-jdk8</artifactId>
    <version>1.2.0.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.2.0.Final</version>
</dependency>
```

在准备两个实体类, 为了方便演示, 用了`lombok`插件. 准备两个实体类, 一个是CarPo

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CarPo {
    private Integer id;
    private String brand;
}
```

还有一个是CarVo

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CarVo {
    private Integer id;
    private String brand;
}
```

再来一个转换接口

```java
@Mapper
public interface CarCovertBasic {
    CarCovertBasic INSTANCE =
    Mappers.getMapper(CarCovertBasic.class);

    CarVo toConvertVo(CarPo source);
}
```

测试代码如下:

```java
//实际中从数据库取
CarPo carPo = CarPo.builder().id(1)
                           .brand("BMW")
                           .build();
CarVo carVo = CarCovertBasic.INSTANCE.toConvertVo(carPo);
System.out.println(carVo);
```

输出如下

```java
CarVo(id=1, brand=BMW)
```

可以看到, carPo的属性值复制给了carVo. 当然, 在这种情况下, 功能和`BeanUtils`是差不多的, 体现不出优势! 嗯, 我们放在后面说, 我们先来说说原理! 

### 原理

其实原理就是MapStruct插件会识别我们的接口, 生成一个实现类, 在实现类中, 为我们实现了set逻辑! 例如, 上面的例子中, 给CarCovertBasic接口, 实现了一个实现类CarCovertBasicImpl, 我们可以用反编译工具看到源码如下图所示

![https://www.miaomiaoqi.github.io/images/java/m_2.png](https://www.miaomiaoqi.github.io/images/java/m_2.png)

下面, 我们来说说优势

### 优势

**两个类型属性不一致**

此时 CarPo 的一个属性为 carName, 而 CarVo 对应的属性为 name! 

我们在接口上增加对应关系即可, 如下所示

```java
@Mapper
public interface CarCovertBasic {
		CarCovertBasic INSTANCE = Mappers.getMapper(CarCovertBasic.class);

		@Mapping(source = "carName", target = "name")
		CarVo toConvertVo(CarPo source);
}
```

测试代码如下

```java
CarPo carPo = CarPo.builder().id(1)
                       .brand("BMW")
                       .carName("宝马")
                       .build();
CarVo carVo = CarCovertBasic.INSTANCE.toConvertVo(carPo);
System.out.println(carVo);
```

输出如下

```java
CarVo(id=1, brand=BMW, name=宝马)
```

可以看到carVo已经能识别到carPo中的carName属性, 并赋值成功. 反编译的图如下

![https://www.miaomiaoqi.github.io/images/java/m_3.png](https://www.miaomiaoqi.github.io/images/java/m_3.png)

`画外音:`如果有多个映射关系可以用@Mappings注解, 嵌套多个@Mapping注解实现, 后文说明! 

**集合类型转换**

如果我们要从List转换为List怎么办呢？简单, 接口里加一个方法就行

```java
@Mapper
public interface CarCovertBasic {
    CarCovertBasic INSTANCE = Mappers.getMapper(CarCovertBasic.class);

    @Mapping(source = "carName", target = "name")
    CarVo toConvertVo(CarPo source);

    List<CarVo> toConvertVos(List<CarPo> source);
}
```

如代码所示, 我们增加了一个toConvertVos方法即可, mapStruct生成代码的时候, 会帮我们去循环调用toConvertVo方法, 给大家看一下反编译的代码, 就一目了然

![https://www.miaomiaoqi.github.io/images/java/m_4.png](https://www.miaomiaoqi.github.io/images/java/m_4.png)

**类型不一致**

在CarPo加一个属性为Date类型的createTime,而在CarVo加一个属性为String类型的createTime,那么代码如下

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CarPo {
    private Integer id;
    private String brand;
    private String carName;
    private Date createTime;
}

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CarVo {
    private Integer id;
    private String brand;
    private String name;
    private String createTime;
}
```

接口就可以这么写

```java
@Mapper
public interface CarCovertBasic {
    CarCovertBasic INSTANCE = Mappers.getMapper(CarCovertBasic.class);
    @Mappings({
        @Mapping(source = "carName", target = "name"),
        @Mapping(target = "createTime", expression = "java(com.guduyan.util.DateUtil.dateToStr(source.getCreateTime()))")
    })
    CarVo toConvertVo(CarPo source);

    List<CarVo> toConvertVos(List<CarPo> source);
}
```

这样在代码中, 就能解决类型不一致的问题! 在生成set方法的时候, 自动调用DateUtil类进行转换, 由于比较简单, 我就不贴反编译的图了! 

**多对一**

在实际业务情况中, 我们有时候会遇到将两个Bean映射为一个Bean的情况, 假设我们此时还有一个类为AtrributePo,我们要将CarPo和AttributePo同时映射为CarBo,我们可以这么写

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AttributePo {
    private double price;
    private String color;
}

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CarBo {
    private Integer id;
    private String brand;
    private String carName;
    private Date createTime;
    private double price;
    private String color;
}
```

接口改变如下

```java
@Mapper
public interface CarCovertBasic {
    CarCovertBasic INSTANCE = Mappers.getMapper(CarCovertBasic.class);
    @Mappings({
        @Mapping(source = "carName", target = "name"),
        @Mapping(target = "createTime", expression = "java(com.guduyan.util.DateUtil.dateToStr(source.getCreateTime()))")
    })
    CarVo toConvertVo(CarPo source);

    List<CarVo> toConvertVos(List<CarPo> source);

    CarBo toConvertBo(CarPo source1, AttributePo source2);
}
```

直接增加接口即可, 插件在生成代码的时候, 会帮我们自动组装, 看看下面的反编译代码就一目了然. 

![https://www.miaomiaoqi.github.io/images/java/m_5.png](https://www.miaomiaoqi.github.io/images/java/m_5.png)

**其他**

关于MapStruct还有其他很多的高级功能, 我就不一一介绍了. 大家可以参考下面的文档, 在用到的时候自行翻阅即可! 文档地址:https://mapstruct.org/documentation/reference-guide/