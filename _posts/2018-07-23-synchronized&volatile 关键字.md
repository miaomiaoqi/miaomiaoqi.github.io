---
layout: post
title: "synchronized&volatile 关键字"
categories: [Java]
description:
keywords:
---

* content
{:toc}






## Java内存模型(JMM)

Java 内存模型是根据英文Java Memory Model(JMM)翻译过来的. 其实  JMM 并不像 JVM 内存结构一样是真实存在的. 他只是一个抽象的概念. JSR-133: Java Memory Model and Thread Specification 中描述了, JMM是和多线程相关的, 他描述了一组规则或规范, 这个规范定义了一个线程对共享变量的写入时对另一个线程是可见的. 

那么, 简单总结下, Java 的多线程之间是通过共享内存进行通信的, 而由于采用共享内存进行通信, 在通信过程中会存在一系列如可见性, 原子性, 顺序性等问题, 而JMM就是围绕着多线程通信以及与其相关的一系列特性而建立的模型. JMM 定义了一些语法集, 这些语法集映射到 Java 语言中就是 volatile, synchronized 等关键字. 

在JMM中, 我们把多个线程间通信的共享内存称之为主内存, 而在并发编程中多个线程都维护了一个自己的本地内存(这是个抽象概念), 其中保存的数据是主内存中的数据拷贝. 而 JMM 主要是控制本地内存和主内存之间的数据交互的. 

* 主内存(Main Memory)

    主内存可以简单理解为计算机当中的内存, 但又不完全等同. 主内存被所有的线程所共享, 对于一个共享变量(比如静态变量, 或是堆内存中的实例)来说, 主内存当中存储了它的“本尊”. 

* 工作内存(Working Memory)

    工作内存可以简单理解为计算机当中的CPU高速缓存, 但又不完全等同. 每一个线程拥有自己的工作内存, 对于一个共享变量来说, 工作内存当中存储了它的“副本”. 
    

线程对共享变量的所有操作都必须在工作内存进行, 不能直接读写主内存中的变量. 不同线程之间也无法访问彼此的工作内存, 变量值的传递只能通过主内存来进行. **因为直接操作主存速度太慢了**

## volatile 关键字

volatile 关键字具有许多特性, **其中最重要的特性就是保证了用 volatile 修饰的变量对所有线程的可见性**. 

这里的可见性是什么意思呢? 当一个线程修改了变量的值, 新的值会立刻同步到主内存当中. 而其他线程读取这个变量的时候, 也会从主内存中拉取最新的变量值. 

为什么volatile关键字可以有这样的特性? 这得益于java语言的先行发生原则(happens-before). 

## Happens-before(JVM 规定指令重排序必须遵守的规则)

Happens-before 是两个事件的结果之间的关系, 如果一个事件发生在另一个事件之前, 结果必须反映, 即使这些事件实际上是乱序执行的(通常是优化程序流程). 提供跨线程的内存可见性

这里所谓的事件, 实际上就是各种指令操作, 比如读操作, 写操作, 初始化操作, 锁操作等等. 

在Java内存模型中, 如果一个操作执行的结果需要对另一个操作可见, 那么这两个操作之间必然存在 Happens-before 关系

先行发生原则作用于很多场景下, 包括同步锁, 线程启动, 线程终止, volatile

Happens-before规则如下:

* 程序次序规则: 同一线程内, 按照代码出现的顺序, 前面的代码先行于后面的代码, 准确的说是控制流顺序, 因为要考虑到分支和循环结构
* 管程锁定规则: 一个 unlock 操作先行发生于后面(时间上)对同一个锁的 lock 操作
* volatile 变量规则: 对一个 volatile 变量的写操作先行发生于后面(时间上)对这个变量的读操作
* 线程启动规则: Thread 的 start() 方法先行发生于这个线程的每一个操作
* 线程终止规则: 线程的所有操作都先行于此线程的终止检测. 可以通过 Thread.join() 方法结束, Thread.isAlive() 的返回值等手段检测线程的终止
* 线程中断规则: 对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生, 可以通过 Thread.interrupt() 方法检测线程是否中断
* 对象终结规则: 一个对象的初始化完成先行于发生它的 finalize() 方法的开始
* 传递性: 如果操作 A 先行于操作 B, 操作 B 先行于操作 C, 那么操作 A 先行于操作 C

## 指令重排序问题

指令重排是指 JVM 在编译 Java 代码的时候, 或者 CPU 在执行 JVM 字节码的时候, 对现有的指令顺序进行重新排序. 

指令重排的目的是为了在不改变程序执行结果的前提下, 优化程序的运行效率. 需要注意的是, 这里所说的不改变执行结果, 指的是不改变单线程下的程序执行结果. 

* 数据依赖性(as-if-serial)

    * 写后读

    * 读后写

    * 写后写
* 指令重排序分类

    * 编译器重排序

    * 处理器重排序
* 为什么要进行指令重排序
* 指令重排序带来的影响
* 竞争与同步

as-if-serial 是看上去像同步的, CPU 为了提高效率, 会将没有依赖关系的语句打乱执行, 提高效率, 最终结果是一致的

## 内存屏障

内存屏障(Memory Barrier)是一种 CPU 指令

内存屏障共分为四种类型: 

* LoadLoad 屏障: 

    抽象场景: Load1; LoadLoad; Load2

    Load1 和 Load2 代表两条读取指令. 在 Load2 要读取的数据被访问前, 保证 Load1 要读取的数据被读取完毕. 

* StoreStore 屏障: 

    抽象场景: Store1; StoreStore; Store2

    Store1 和 Store2 代表两条写入指令. 在 Store2 写入执行前, 保证 Store1 的写入操作对其它处理器可见

* LoadStore 屏障: 

    抽象场景: Load1; LoadStore; Store2

    在 Store2 被写入前, 保证 Load1 要读取的数据被读取完毕. 

* StoreLoad 屏障: 

    抽象场景: Store1; StoreLoad; Load2

    在 Load2 读取操作执行前, 保证 Store1 的写入对所有处理器可见. StoreLoad 屏障的开销是四种屏障中最大的. 

volatile 做了什么? 


在一个变量被 volatile 修饰后, JVM 会为我们做两件事: 

1. 在每个 volatile 写操作前插入 StoreStore 屏障, 在写操作后插入 StoreLoad 屏障. 

2. 在每个 volatile 读操作前插入 LoadLoad 屏障, 在读操作后插入 LoadStore 屏障. 



## DCL(Double Check Lock)单例为什么要加 volatile

首先讲一下对象的创建过程

<img src="http://www.milky.show/images/java/volatile/volatile_1.png" alt="http://www.milky.show/images/java/volatile/volatile_1.png" style="zoom: 33%;" />

1.  在堆内存中开辟一片空间, 此时 m 是默认值 0
2.  构造函数将 m 赋值为 8
3.  将 t 指向堆内存中的对象

所以实际上创建一个对象在字节码层面是分为 3 个步骤的

<img src="http://www.milky.show/images/java/volatile/volatile_2.png" alt="http://www.milky.show/images/java/volatile/volatile_2.png" style="zoom:33%;" />

线程 1 在 DCL 单例中正在执行初始化过程时, **发生了指令重排序**, 导致步骤 3 比 2 先执行, 此时 m 是一个半初始化的状态, 恰巧线程 2 此时过来判断 instance != null 是成立的, 那么线程 2 就拿到了一个初始化不完全的单例对象

所以需要加 volatile 关键字, 防止指令重排序的发生



## 总结

* volatile 特性之一: 

    保证变量在线程之间的可见性. 可见性的保证是基于 CPU 的内存屏障指令, 被 JSR-133 抽象为 happens-before 原则. 

* volatile 特性之二: 

    阻止编译时和运行时的指令重排. 编译时JVM编译器遵循内存屏障的约束, 运行时依靠 CPU 屏障指令来阻止重排. 

* happens-before 是 JSR-133 规范之一

* 内存屏障是 happens-before 的实现
