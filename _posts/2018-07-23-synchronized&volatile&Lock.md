---
layout: post
title: "synchronized&volatile&Lock"
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



## 我们经常说到的, 有序性、可见性、原子性, synchronized又是怎么做到的呢?

### 有序性

我在Volatile章节已经说过了CPU会为了优化我们的代码, 会对我们程序进行重排序. 

**as-if-serial**

不管编译器和CPU如何重排序, 必须保证在单线程情况下程序的结果是正确的, 还有就是有数据依赖的也是不能重排序的. 

```java
int a = 1;
int b = a;
```

这两段是怎么都不能重排序的, b的值依赖a的值, a如果不先赋值, 那就为空了. 

### 可见性

同样在Volatile章节我介绍到了现代计算机的内存结构, 以及JMM(Java内存模型), 这里我需要说明一下就是JMM并不是实际存在的, 而是一套规范, 这个规范描述了很多java程序中各种变量(线程共享变量)的访问规则, 以及在JVM中将变量存储到内存和从内存中读取变量这样的底层细节, Java内存模型是对共享数据的可见性、有序性、和原子性的规则和保障. 

大家感兴趣, 也记得去了解计算机的组成部分, cpu、内存、多级缓存等, 会帮助更好的理解java这么做的原因. 

<img src="http://www.milky.show/images/java/synchronized/syn_14.png" alt="http://www.milky.show/images/java/synchronized/syn_14.png" style="zoom:67%;" />

### 原子性

其实他保证原子性很简单, 确保同一时间只有一个线程能拿到锁, 能够进入代码块这就够了. 

这几个是我们使用锁经常用到的特性, 那synchronized他自己本身又具有哪些特性呢? 

### 可重入性

synchronized锁对象的时候有个计数器, 他会记录下线程获取锁的次数, 在执行完对应的代码块之后, 计数器就会-1, 直到计数器清零, 就释放锁了. 

那可重入有什么好处呢? 

可以避免一些死锁的情况, 也可以让我们更好封装我们的代码. 

### 不可中断性

不可中断就是指, 一个线程获取锁之后, 另外一个线程处于阻塞或者等待状态, 前一个不释放, 后一个也一直会阻塞或者等待, 不可以被中断. 

值得一提的是, Lock的tryLock方法是可以被中断的. 



## Volatile 关键字

volatile 关键字具有许多特性, **其中最重要的特性就是保证了用 volatile 修饰的变量对所有线程的可见性**. 

这里的可见性是什么意思呢? 当一个线程修改了变量的值, 新的值会立刻同步到主内存当中. 而其他线程读取这个变量的时候, 也会从主内存中拉取最新的变量值. 

为什么volatile关键字可以有这样的特性? 这得益于java语言的先行发生原则(happens-before). 

**总结**

* volatile 特性之一: 

    保证变量在线程之间的可见性. 可见性的保证是基于 CPU 的内存屏障指令, 被 JSR-133 抽象为 happens-before 原则. 

* volatile 特性之二: 

    阻止编译时和运行时的指令重排. 编译时JVM编译器遵循内存屏障的约束, 运行时依靠 CPU 屏障指令来阻止重排. 

* happens-before 是 JSR-133 规范之一

* 内存屏障是 happens-before 的实现

## Happens-before(JVM 规定指令重排序必须遵守的规则)

Happens-before 是两个事件的结果之间的关系, 如果一个事件发生在另一个事件之前, 结果必须反映, 即使这些事件实际上是乱序执行的(通常是优化程序流程). 提供跨线程的内存可见性

这里所谓的事件, 实际上就是各种指令操作, 比如读操作, 写操作, 初始化操作, 锁操作等等. 

在Java内存模型中, 如果一个操作执行的结果需要对另一个操作可见, 那么这两个操作之间必然存在 Happens-before 关系

先行发生原则作用于很多场景下, 包括同步锁, 线程启动, 线程终止, volatile

Happens-before 规则如下:

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



## Synchronized 关键字

synchronized 关键字是Java并发编程中线程同步的常用手段之一, 其作用有三个:

1. 互斥性: 确保线程互斥的访问同步代, 锁自动释放, 多个线程操作同个代码块或函数必须排队获得锁, 

2. 可见性: 保证共享变量的修改能够及时可见, 获得锁的线程操作完毕后会将所数据刷新到共享内存区[1]

3. 有序性: 有效解决重排序问题

synchronized 用法有三个:

1. 修饰实例方法
2. 修饰静态方法
3. 修饰代码块

### 修饰实例方法

`synchronized`关键词作用在方法的前面, 用来锁定方法, 其实默认锁定的是`this`对象. 

```java
public class Thread1 implements Runnable{
    // 共享资源(临界资源)
    static int i=0;
    // 如果没有synchronized关键字, 输出小于20000
    public synchronized void increase(){
        i++;
    }
    public void run() {
        for(int j=0;j<10000;j++){
            increase();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        Thread1 t=new Thread1();
        Thread t1=new Thread(t);
        Thread t2=new Thread(t);
        t1.start();
        t2.start();
        t1.join(); // 主线程等待t1执行完毕
        t2.join(); // 主线程等待t2执行完毕
        System.out.println(i);
    }
}
```

### 修饰静态方法

`synchronized`还是修饰在方法上, 不过修饰的是静态方法, 等价于锁定的是`Class`对象, 

```java
public class Thread1 {
    // 共享资源(临界资源)
    static int i = 0;
    // 如果没有synchronized关键字, 输出小于20000
    public static synchronized void increase() {
        i++;
    }
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                for (int j = 0; j < 10000; j++) {
                    increase();
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int j = 0; j < 10000; j++) {
                    increase();
                }
            }
        });
        t1.start();
        t2.start();
        t1.join(); // 主线程等待t1执行完毕
        t2.join(); // 主线程等待t2执行完毕
        System.out.println(i);
    }
}
```

### 修饰代码块

用法是在函数体内部对于要修改的参数区间用`synchronized`来修饰, 相比与锁定函数这个范围更小, 可以指定锁定什么对象. 

```java
public class Thread1 implements Runnable {
    //共享资源(临界资源)
    static int i = 0;

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            //获得了String的类锁
            synchronized (String.class) {
                i++;
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        Thread1 t = new Thread1();
        Thread t1 = new Thread(t);
        Thread t2 = new Thread(t);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```

**总结**

1. synchronized 修饰的实例方法, 多线程并发访问时, `只能有一个线程进入`, 获得对象内置锁, 其他线程阻塞等待, 但在此期间线程仍然可以访问其他方法. 
2. synchronized 修饰的静态方法, 多线程并发访问时, 只能有一个线程进入, 获得类锁, 其他线程阻塞等待, 但在此期间线程仍然可以访问其他方法. 
3. synchronized 修饰的代码块, 多线程并发访问时, 只能有一个线程进入, 根据括号中的对象或者是类, 获得相应的对象内置锁或者是类锁
4. 每个类都有一个类锁, 类的每个对象也有一个内置锁, 它们是`互不干扰`的, 也就是说一个线程可以同时获得类锁和该类实例化对象的内置锁, 当线程访问非synchronzied修饰的方法时, 并不需要获得锁, 因此不会产生阻塞. 

### 管程

管程: (英语: Monitors, 也称为监视器) 在操作系统中是很重要的概念, 管程其实指的是`管理共享变量以及管理共享变量的操作过程`. 有点扮演中介的意思, 管程管理一堆对象, 多个线程同一时候只能有一个线程来访问这些东西. 

1. 管程可以看做一个软件模块, 它是将共享的变量和对于这些共享变量的操作封装起来, 形成一个具有一定接口的功能模块, 进程可以调用`管程`来实现进程级别的并发控制. 
2. 进程只能互斥的使用管程, 即当一个进程使用管程时, 另一个进程必须等待. 当一个进程使用完管程后, 它必须释放管程并唤醒等待管程的某一个进程. 

管程解决互斥问题相对简单, 把共享变量以及共享变量的操作都封装在一个类中

<img src="http://www.milky.show/images/java/synchronized/syn_1.png" alt="http://www.milky.show/images/java/synchronized/syn_1.png" />

当线程 A 和线程 B 需要获取共享变量 count 时, 就需要调用 get 和 set 方法, 而get和set方法则保证互斥性, 保证每次只能有一个线程访问. 

生活中举例管程比如链家店长分配给每一个中介管理一部分二手房源, 多个客户通过中介进行房屋买卖. 

1. 中介就是管程. 
2. 多个二手房源被一个中介管理中, 就是一个管程管理着多个系统资源. 
3. 多个客户就相当于多个线程. 

### Synchronzied 的底层原理

我们知道在 Java 的 JVM 内存区域中一个对象在堆区创建, 创建后的对象由三部分组成. 

<img src="http://www.milky.show/images/java/synchronized/syn_2.png" alt="http://www.milky.show/images/java/synchronized/syn_2.png" style="zoom: 50%;" />

这三部分功能如下: 

1. 填充数据: 由于虚拟机要求对象起始地址必须是8字节的整数倍. 填充数据不是必须存在的, 仅仅是为了`字节对齐`. 

    - 由于虚拟机要求对象起始地址必须是8字节的整数倍, 填充数据不是必须存在的, 仅仅是为了字节对齐. 

        Tip: 不知道大家有没有被问过一个空对象占多少个字节? 就是8个字节, 是因为对齐填充的关系哈, 不到8个字节对其填充会帮我们自动补齐. 

2. 实例变量: 存放类的`属性数据`信息, 包括`父类`的属性信息, 这部分内存按4字节对齐. 

3. 对象头: 主要包括两部分 `Klass Point`跟 `Mark Word``

    `Klass Point`(类型指针): 是对象指向它的类元数据的指针, 虚拟机通过这个指针来确定这个对象是哪个类的实例. 

    `Mark Word`(标记字段): 这一部分用于储存对象自身的运行时数据, 如`哈希码`, `GC`分代年龄, `锁状态标志`, `锁指针`等, 这部分数据在32bit和64bit的虚拟机中大小分别为32bit和64bit, 考虑到虚拟机的空间效率, Mark Word被设计成一个`非固定`的数据结构以便在极小的空间中存储尽量多的信息, 它会根据对象的状态复用自己的存储空间(跟[ConcurrentHashMap](https://mp.weixin.qq.com/s?__biz=MzI4NjI1OTI4Nw==&mid=2247487563&idx=1&sn=0a223ae2bba963e3ac40b7ce6d9ecd56&scene=21#wechat_redirect)里的标志位类似), 详细情况如下图: 

    `Mark Word`状态表示位如下: 

    <img src="http://www.milky.show/images/java/synchronized/syn_3.png" alt="http://www.milky.show/images/java/synchronized/syn_3.png" style="zoom:67%;" />

    `synchronized`不论是修饰方法还是代码块, 都是通过持有修饰对象的`锁`来实现同步, `synchronized`锁对象是存在对象头`Mark Word`. 其中轻量级锁和偏向锁是`Java6`对`synchronized`锁进行优化后新增加的, 这里我们主要分析一下重量级锁也就是通常说synchronized的对象锁, 锁标识位为10, 其中指针指向的是`monitor`对象(也称为管程或监视器锁)的起始地址. 每个对象都存在着一个 monitor[4] 与之关联. 

    <img src="http://www.milky.show/images/java/synchronized/syn_4.png" alt="http://www.milky.show/images/java/synchronized/syn_4.png" style="zoom:67%;" />

<img src="http://www.milky.show/images/java/synchronized/syn_13.png" alt="http://www.milky.show/images/java/synchronized/syn_13.png" style="zoom:67%;" />

### 反汇编查看

分析对象的`monitor`前我们先通过反汇编看下同步方法跟同步方法块在汇编语言级别是什么样的指令. 

```java
public class SynchronizedTest {
    public synchronized void doSth(){
        System.out.println("Hello World");
    }
    public void doSth1(){
        synchronized (SynchronizedTest.class){
            System.out.println("Hello World");
        }
    }
}
```

`javac SynchronizedTest.java` 然后`javap -c SynchronizedTest`反编译后看汇编指令如下: 

```java
public synchronized void doSth();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED //  这是重点 方法锁
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  
         3: ldc           #3    
         5: invokevirtual #4                  
         8: return

  public void doSth1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                 
         2: dup
         3: astore_1
         4: monitorenter  //   进入同步方法
         5: getstatic     #2                  
         8: ldc           #3                  
        10: invokevirtual #4                
        13: aload_1
        14: monitorexit  //正常时 退出同步方法
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit  // 异常时 退出同步方法
        21: aload_2
        22: athrow
        23: return
```

我们可以看到Java编译器为我们生成的字节码. 在对于doSth和doSth1的处理上稍有不同. 也就是说. JVM对于同步方法和同步代码块的处理方式不同. 对于同步方法, JVM采用`ACC_SYNCHRONIZED`标记符来实现同步. 对于同步代码块. JVM采用`monitorenter`, `monitorexit`两个指令来实现同步. 

- ACC_SYNCHRONIZED

    方法级的同步是`隐式`的. 同步方法的常量池中会有一个`ACC_SYNCHRONIZED`标志. 当某个线程要访问某个方法的时候, 会检查是否有`ACC_SYNCHRONIZED`, 如果有设置, 则需要先获得`监视器锁`, 然后开始执行方法, 方法执行之后再释放监视器锁. 这时如果其他线程来请求执行方法, 会因为无法获得监视器锁而被阻断住. 值得注意的是, 如果在方法执行过程中, 发生了异常, 并且方法内部并没有处理该异常, 那么在异常被抛到方法外面之前监视器锁会被自动释放. ACC_SYNCHRONIZED会去隐式调用刚才的两个指令: monitorenter 和 monitorexit. 

- monitorenter 跟 monitorexit

    **monitorenter**
    
    (重入锁的体现)每个对象都有一个监视器锁(monitor), 当 monitor 被占用时就会处于锁定状态, 线程执行 monitorenter 指令时尝试获取 monitor 的所有权. 
    
    1. 如果 monitor 的进入数为 0, 则该线程进入 monitor, 然后将进入数设置为 1, 该线程即为 monitor 的所有者. 
    2. 如果线程已经占有该 monitor, 只是重新进入, 则进入 monitor 的进入数加 1. 
    3. 如果其他线程已经占用了 monitor, 则该线程进入阻塞状态, 直至 monitor 的进入数为 0, 再重新尝试获取 monitor 的所有权. 
    
    **monitorexit**
    
    1. 执行 monitorexit 的线程必须是 object 所对应的 monitor 的所有者. 
    
    2. 指令执行时, monitor 的进入数减 1, 如果减 1 后进入数为 0, 那么线程就会退出 monitor, 不再是这个 monitor 的所有者. 其他被这个 monitor 阻塞的线程可以尝试去获取这个 monitor 的所有权. 
    
    可以把执行`monitorenter`指令理解为加锁, 执行`monitorexit`理解为释放锁. 每个对象维护着一个记录着被锁次数的计数器. 未被锁定的对象的该计数器为 0, 当一个线程获得锁(执行`monitorenter`)后, 该计数器自增变为 1, 当同一个线程再次获得该对象的锁的时候, 计数器再次自增. 当同一个线程释放锁(执行`monitorexit`指令)的时候, 计数器再自减. 当计数器为0的时候. 锁将被释放, 其他线程便可以获得锁. 

结论: 同步方法和同步代码块底层都是通过`monitor`来实现同步的. 两者区别: 同步方式是通过方法中的`access_flags`中设置`ACC_SYNCHRONIZED`标志来实现, 同步代码块是通过`monitorenter`和`monitorexit`来实现. 

### monitor解析

monitor 监视器源码是 C++ 写的, 在虚拟机的 ObjectMonitor.hpp 文件中. 

我看了下源码, 他的数据结构长这样: 

```c++
 ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;  // 线程重入次数
    _object       = NULL;  // 存储Monitor对象
    _owner        = NULL;  // 持有当前线程的owner
    _WaitSet      = NULL;  // wait状态的线程列表
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;  // 单向列表
    FreeNext      = NULL ;
    _EntryList    = NULL ;  // 处于等待锁状态block状态的线程列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
  }
```

大家说熟悉的锁升级过程, 其实就是在源码里面, 调用了不同的实现去获取获取锁, 失败就调用更高级的实现, 最后升级完成. 

每个对象都与一个`monitor`相关联, 而`monitor`可以被线程拥有或释放, 在Java虚拟机(HotSpot)中, `monitor`是由`ObjectMonitor`实现的, 其主要数据结构如下(位于HotSpot虚拟机源码ObjectMonitor.hpp文件, C++实现的). 

```java
ObjectMonitor() {
    _count        = 0;      // 记录数
    _recursions   = 0;      // 锁的重入次数
    _owner        = NULL;   // 指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL;   // 调用wait后, 线程会被加入到_WaitSet
    _EntryList    = NULL ;  // 等待获取锁的线程, 会被加入到该列表
}
```

monitor 运行图如下: 

<img src="http://www.milky.show/images/java/synchronized/syn_16.png" alt="http://www.milky.show/images/java/synchronized/syn_16.png" style="zoom:67%;" />

该图可以看出, 任意线程对 object 的访问, 首先要获得 object 的监视器, 如果获取失败, 该线程就进入同步队列, 线程状态变为 blocked , 当 object 的监视器占有者释放后, 在同步队列中的线程就会有机会重新获取该监视器. 

<img src="http://www.milky.show/images/java/synchronized/syn_5.png" alt="http://www.milky.show/images/java/synchronized/syn_5.png" style="zoom:67%;" />

对于一个 synchronized 修饰的方法(代码块)来说: 

1. 当多个线程同时访问该方法, 那么这些线程会先被放进`_EntryLis`t队列, 此时线程处于`blocked`状态
2. 当一个线程获取到了对象的`monitor`后, 那么就可以进入`running`状态, 执行方法块, 此时, `ObjectMonitor`对象的`_owner`指向当前线程, `_count`加1表示当前对象锁被一个线程获取. 
3. 当`running`状态的线程调用`wait()`方法, 那么当前线程释放`monitor`对象, 进入`waiting`状态, `ObjectMonitor`对象的`_owner变为`null, `_count`减1, 同时线程进入`_WaitSet`队列, 直到有线程调用`notify()`方法唤醒该线程, 则该线程进入`_EntryList`队列, 竞争到锁再进入`_owner`区. 
4. 如果当前线程执行完毕, 那么也释放`monitor`对象, `ObjectMonitor`对象的`_owner`变为null, `_count`减1. 

因为监视器锁(monitor)是依赖于底层的操作系统的`Mutex Lock`来实现的, 而操作系统实现线程之间的切换时需要从`用户态转换到核心态`(具体可看 CXUAN 写的 OS 哦), 这个状态之间的转换需要相对比较长的时间, 时间成本相对较高, 这也是早期的`synchronized`效率低的原因. 庆幸在 Java 6 之后 Java 官方对从 JVM 层面对`synchronized`较大优化最终提升显著, Java 6 之后, 为了减少获得锁和释放锁所带来的性能消耗, 引入了锁升级的概念. 



## 锁升级

synchronized 锁有四种状态, `无锁`, `偏向锁`, `轻量级锁`, `重量级锁`. 这几个状态会随着竞争状态逐渐升级, 锁可以`升级`但不能`降级`, 但是偏向锁状态可以被重置为无锁状态. 科学性的说这些锁之前我们先看个简单通俗的例子来加深印象. 

简单版本: 

<img src="http://www.milky.show/images/java/synchronized/syn_8.png" alt="http://www.milky.show/images/java/synchronized/syn_8.png"  />

升级方向: 

<img src="http://www.milky.show/images/java/synchronized/syn_9.png" alt="http://www.milky.show/images/java/synchronized/syn_9.png" />

Tip: 切记这个升级过程是不可逆的, 最后我会说明他的影响, 涉及使用场景. 

看完他的升级, 我们就来好好聊聊每一步怎么做的吧. 

### 通俗说各种锁

偏向锁, 轻量级锁和重量级锁之间的关系, 首先打个比方[5]: 假设现在厕所只有一个位置, 每个使用者都有打开门锁的钥匙. 必须打开门锁才能使用厕所. 其中小明, 小红理解为两个线程, 上厕所理解为执行同步代码, 门锁理解为同步代码的锁

1. 小明今天吃坏了东西需要反复去厕所, 如果小明每次都要开锁就很耽误时间, 于是门锁将小明的脸记录下来(假设那个锁是智能锁), 下次小明再来的时候门锁会自动识别出是小明来了, 然后自动开锁, 这样就省去了小明拿钥匙开门的过程, 此时门锁就是`偏向锁`, 也可以理解为偏向小明的锁. 
2. 接下来, 小红又去上厕所, 试图将厕所的门锁设置为偏向自己的偏向锁, 于是发现门锁无法偏向自己, 因为此时门锁已是偏向小明的偏向锁. 于是小红很生气, 要求门锁撤销对小明的偏向, 当然, 小明也不同意门锁偏向小红. 于是等小明用完厕所之后, 门锁撤销了对任何人的偏向(只要出现竞争的情况, 就会撤销偏向锁). 这个过程就是撤销偏向锁. 此时`门锁升级为轻量级锁`. 
3. 等小明出来以后, 轻量级锁正式生效 . 下一次小明和小红同时来厕所, 谁跑的快谁先走到门前, 开门后将门锁拿进厕所, 并将门锁打开以后拿进厕所里, 将门反锁, 于是在门外原来放门锁的位置放置了一个`有人`的标志(这个标识可以理解为指向门锁的指针, 或者理解为作为锁的Java对象头的`Mark Word`值), 这时, 小红看到有人以后很着急, 催着里面的人出来时马上进去, 于是不断的来敲门, 问小明什么时候出来. 这个过程就是`自旋`. 
4. 反复敲了几次以后, 小明受不了了, 对小红喊话, 说你别敲了, 等我用完厕所我告诉你, 于是小红去一边等着(线性阻塞). 此时门锁升级为`重量级锁`. 升级为重量级锁的后果就是, 小红不再反复敲门, 小明在上完厕所以后必须告诉小红一声, 否则小红就会一直等着. 

结论: 

偏向锁在只有一个人上厕所时非常高效, 省去了开门的过程. 轻量级锁在有多人上厕所但是每个人使用的特别快的时候, 比较高效, 因为会出现这种现象, 小红敲门的时候正好赶上小明出来, 这样就省得小明出来告诉小红以后小红才能进去, 但是这样可能会出现小红敲门失败的情况(就是敲门时小明还没用完). 重量级锁相比与轻量级锁的多了一步小明呼唤小红的步骤, 但是却省掉了小红反复去敲门的过程, 但是能保证小红去厕所时厕所一定是没人的. 

### 偏向锁

之前我提到过了, 对象头是由Mark Word和Klass pointer 组成, 锁争夺也就是对象头指向的Monitor对象的争夺, 一旦有线程持有了这个对象, 标志位修改为1, 就进入偏向模式, 同时会把这个线程的ID记录在对象的Mark Word中. 

这个过程是采用了CAS乐观锁操作的, 每次同一线程进入, 虚拟机就不进行任何同步的操作了, 对标志位+1就好了, 不同线程过来, CAS会失败, 也就意味着获取锁失败. 

偏向锁在1.6之后是默认开启的, 1.5中是关闭的, 需要手动开启参数是xx:-UseBiasedLocking=false. 

<img src="http://www.milky.show/images/java/synchronized/syn_10.png" alt="http://www.milky.show/images/java/synchronized/syn_10.png" style="zoom:67%;" />

经过 HotSpot 的作者大量的研究发现大多数时候是`不存在锁竞争`的, 经常是一个线程多次获得同一个锁, 因此如果每次都要`竞争锁`会增大很多没有必要付出的代价, 为了降低获取锁的代价, 才引入的偏向锁. 核心思想: 

如果一个线程获得了锁, 那么锁就进入偏向模式, 此时`Mark Word` 的结构也变为偏向锁结构, 当这个线程再次请求锁时, 无需再做任何同步操作, 即获取锁的过程, 这样就省去了大量有关锁申请的操作, 从而也就提供程序的性能. 所以, 对于没有锁竞争的场合, 偏向锁有很好的优化效果, 毕竟极有可能连续多次是同一个线程申请相同的锁. 但是对于锁竞争比较激烈的场合, 偏向锁就失效了, 因为这样场合极有可能每次申请锁的线程都是不相同的, 因此这种场合下不应该使用偏向锁, 否则会得不偿失, 需要注意的是, 偏向锁失败后, 并不会立即膨胀为重量级锁, 而是先升级为轻量级锁. 

具体流程: 当线程 1 访问代码块并获取锁对象时, 会在 java 对象头和栈帧中记录偏向的锁的`threadID`, 因为`偏向锁不会主动释放锁`, 因此以后线程1再次获取锁的时候, 需要`比较`当前线程的`threadID`和Java对象头中的`threadID`是否一致, 如果一致(还是线程1获取锁对象), 则无需使用 CAS 来加锁, 解锁；如果不一致(其他线程, 如线程 2 要竞争锁对象, 而偏向锁不会主动释放因此还是存储的线程 1 的 threadID), 那么需要查看Java对象头中记录的线程 1`是否存活`, 如果没有存活, 那么锁对象被重置为无锁状态, 其它线程(线程2)可以竞争将其设置为偏向锁；如果存活, 那么立刻查找该线程(线程1)的栈帧信息, 如果还是需要继续持有这个锁对象, 那么暂停当前线程 1, 撤销偏向锁, 升级为 `轻量级锁`, 如果线程 1 不再使用该锁对象, 那么将锁对象状态设为无锁状态, 重新偏向新的线程. 

**偏向锁的撤销**

偏向锁使用了一种等到竞争出现才释放锁的机制, 所以当其他线程尝试竞争偏向锁时, 持有偏向锁的线程才会释放锁. 

<img src="http://www.milky.show/images/java/synchronized/syn_17.png" alt="http://www.milky.show/images/java/synchronized/syn_17.png" style="zoom:67%;" />

如图, 偏向锁的撤销, 需要等待全局安全点(在这个时间点上没有正在执行的字节码). 它会首先暂停拥有偏向锁的线程, 然后检查持有偏向锁的线程是否还存活,  如果线程不处于活动状态, 则将对象头设置成无锁状态；如果线程仍然活着, 拥有偏向锁的栈会被执行, 遍历偏向对象的锁记录, 栈中的锁记录和对象头的 mark word 要么重新偏向于其他线程, 要么恢复到无锁或者标记对象不适合作为偏向锁, 最后唤醒暂停的线程

<img src="http://www.milky.show/images/java/synchronized/syn_18.png" alt="http://www.milky.show/images/java/synchronized/syn_18.png" style="zoom: 67%;" />

**如何关闭偏向锁**

偏向锁在 Java6 和 Java 7 中是默认启用的, 但是它在应用程序启动几秒钟后才激活, 如果必要可以使用 JVM 参数来关闭延迟:  -XX:BiasedLockingStartupDelay=0. 如果你确定应用程序里所有的锁通常情况下处于竞争状态, 可以通过 JVM 参数关闭偏向锁:  -XX:-UseBiasedLocking=false, 那么程序默认进入轻量级锁状态. 

### 轻量级锁

轻量级锁考虑的是竞争锁对象的`线程不多`, 而且线程持有锁的`时间也不长`的情景. 因为阻塞线程需要高昂的耗时实现CPU从`用户态转到内核态`的切换, 如果刚刚阻塞不久这个锁就被释放了, 那这个代价就有点得不偿失了, 因此这个时候就干脆不阻塞这个线程, 让它`自旋`这等待锁释放. 

**「原理跟升级」**: 线程1获取轻量级锁时会先把锁对象的对象头`MarkWord`复制一份到线程1的栈帧中创建的用于存储锁记录的空间(称为DisplacedMarkWord), 然后使用CAS把对象头中的内容替换为线程1存储的锁记录(DisplacedMarkWord)的地址；

如果在线程1复制对象头的同时(在线程1CAS之前), 线程2也准备获取锁, 复制了对象头到线程2的锁记录空间中, 但是在线程2CAS的时候, 发现线程1已经把对象头换了, **「线程2的CAS失败, 那么线程2就尝试使用自旋锁来等待线程1释放锁」**. 自旋锁简单来说就是让线程2`在循环中不断CAS尝试获得锁对象`. 

但是如果自旋的**「时间太长」**也不行, 因为自旋是要消耗CPU的, 因此**「自旋的次数是有限制」**的, 比如10次或者100次, 如果自旋次数到了线程1还没有释放锁, 或者线程1还在执行, 线程2还在自旋等待, 那么这个时候轻量级锁就会膨胀为重量级锁. 重量级锁把除了拥有锁的线程都阻塞, 防止CPU空转. 

<img src="http://www.milky.show/images/java/synchronized/syn_6.png" alt="http://www.milky.show/images/java/synchronized/syn_6.png" style="zoom: 50%;" />

还是跟Mark Work 相关, 如果这个对象是无锁的, jvm就会在当前线程的栈帧中建立一个叫锁记录(Lock Record)的空间, 用来存储锁对象的Mark Word 拷贝, 然后把Lock Record中的owner指向当前对象. 

JVM接下来会利用CAS尝试把对象原本的Mark Word 更新会Lock Record的指针, 成功就说明加锁成功, 改变锁标志位, 执行相关同步操作. 

如果失败了, 就会判断当前对象的Mark Word是否指向了当前线程的栈帧, 是则表示当前的线程已经持有了这个对象的锁, 否则说明被其他线程持有了, 继续锁升级, 修改锁的状态, 之后等待的线程也阻塞. 

<img src="http://www.milky.show/images/java/synchronized/syn_11.png" alt="http://www.milky.show/images/java/synchronized/syn_11.png" style="zoom:67%;" />

**加锁**

线程在执行同步块之前, JVM 会先在当前线程的栈帧中创建用于存储锁记录的空间, 并将对象头中的 mark word 复制到锁记录中, 官方称为 Displaced Mark Word. 然后线程尝试使用 CAS 将对象头中的 mark word 替换为指向锁记录的指针. 如果成功, 当前线程获得锁, 如果失败, 表示其他线程竞争锁, 当前线程便尝试使用自旋来获取锁.  

**解锁**

轻量级解锁是, 会使用原子的 CAS 操作将 Displaced Mark Word 替换回到对象头, 如果成功, 则表示没有竞争发生. 如果失败, 表示当前锁存在竞争, 锁就会膨胀成重量级锁. 

下图是两个线程同时争夺锁, 导致锁膨胀的流程图. 

<img src="http://www.milky.show/images/java/synchronized/syn_19.png" alt="http://www.milky.show/images/java/synchronized/syn_19.png" style="zoom:67%;" />

因为自旋会消耗 CPU, 为了避免无用的自旋(比如, 获得锁的线程被阻塞住了), 一旦锁升级成了重量级锁, 就不会再恢复到轻量级锁状态. 当锁处于这个状态下, 其他线程试图获取锁时, 都会被阻塞住, 当持有锁的线程释放锁之后会唤醒这些线程, 被唤醒的线程就会进行新一轮的争夺锁. 

### 自旋锁

我不是在上面提到了 Linux 系统的用户态和内核态的切换很耗资源, 其实就是线程的等待唤起过程, 那怎么才能减少这种消耗呢? 

自旋, 过来的现在就不断自旋, 防止线程被挂起, 一旦可以获取资源, 就直接尝试成功, 直到超出阈值, 自旋锁的默认大小是10次, -XX: PreBlockSpin可以修改. 

自旋都失败了, 那就升级为重量级的锁, 像1.5的一样, 等待唤起咯. 

<img src="http://www.milky.show/images/java/synchronized/syn_12.png" alt="http://www.milky.show/images/java/synchronized/syn_12.png" />

### 重量级锁

大家在看ObjectMonitor源码的时候, 会发现 Atomic::cmpxchg_ptr, Atomic::inc_ptr 等内核函数, 对应的线程就是 park() 和 upark(). 

这个操作涉及用户态和内核态的转换了, 这种切换是很耗资源的, 所以知道为啥有自旋锁这样的操作了吧, 按道理类似死循环的操作更费资源才是对吧? 其实不是, 大家了解一下就知道了. 



### 各种锁的比较

| 锁       | 优点                                                         | 缺点                                           | 适用场景                           |
| -------- | ------------------------------------------------------------ | ---------------------------------------------- | ---------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外的消耗, 和执行非同步方法相比仅存在 纳秒级 的差距 | 如果线程间存在锁竞争, 会带来额外的锁撤销的消耗 | 适用于只有一个线程访问同步块的场景 |
| 轻量级锁 | 竞争的线程不会阻塞, 提高了程序的响应速度                     | 如果始终得不到锁竞争的线程, 使用自旋会消耗 CPU | 追求响应时间, 同步块执行速度非常快 |
| 重量级锁 | 线程竞争不使用自旋, 不会消耗CPU                              | 线程阻塞, 响应时间缓慢                         | 追求吞吐量, 同步块执行速度长       |



### 那用户态和内核态又是啥呢? 

Linux系统的体系结构大家大学应该都接触过了, 分为用户空间(应用程序的活动空间)和内核. 

我们所有的程序都在用户空间运行, 进入用户运行状态也就是(用户态), 但是很多操作可能涉及内核运行, 比我I/O, 我们就会进入内核运行状态(内核态). 

<img src="http://www.milky.show/images/java/synchronized/syn_15.png" alt="http://www.milky.show/images/java/synchronized/syn_15.png" style="zoom:67%;" />

这个过程是很复杂的, 也涉及很多值的传递, 我简单概括下流程: 

1. 用户态把一些数据放到寄存器, 或者创建对应的堆栈, 表明需要操作系统提供的服务. 
2. 用户态执行系统调用(系统调用是操作系统的最小功能单位). 
3. CPU切换到内核态, 跳到对应的内存指定的位置执行指令. 
4. 系统调用处理器去读取我们先前放到内存的数据参数, 执行程序的请求. 
5. 调用完成, 操作系统重置CPU为用户态返回结果, 并执行下个指令. 

所以大家一直说, 1.6之前是重量级锁, 没错, 但是他重量的本质, 是ObjectMonitor调用的过程, 以及Linux内核的复杂运行机制决定的, 大量的系统资源消耗, 所以效率才低. 

还有两种情况也会发生内核态和用户态的切换: 异常事件和外围设备的中断 大家也可以了解下. 



### 锁消除 lock eliminate

消除锁是虚拟机另外一种锁的优化, 这种优化更彻底, Java虚拟机在JIT编译时通过对运行上下文的扫描, 去除不可能存在共享资源竞争的锁, 通过这种方式消, 除没有必要的锁, 可以节省毫无意义的请求锁时间, 我们知道`StringBuffer`是线程安全的, 里面包含锁的存在, 但是如果我们在函数内部使用`StringBuffer`那么代码会在JIT后会自动将锁释放掉哦. 

```java
public void add(String str1,String str2){
    StringBuffer sb = new StringBuffer();
    sb.append(str1).append(str2);
}
```

我们都知道 StringBuffer 是线程安全的, 因为它的关键方法都是被 synchronized 修饰过的, 但我们看上面这段代码, 我们会发现, sb 这个引用只会在 add 方法中使用, 不可能被其它线程引用(因为是局部变量, 栈私有), 因此 sb 是不可能共享的资源, JVM 会自动消除 StringBuffer 对象内部的锁. 



### 锁粗化 lock coarsening

```java
public String test(String str){
    int i = 0;
    StringBuffer sb = new StringBuffer():
    while(i < 100){
        sb.append(str);
        i++;
    }
    return sb.toString():
}
```

JVM 会检测到这样一连串的操作都对同一个对象加锁(while 循环内 100 次执行 append, 没有锁粗化的就要进行 100  次加锁/解锁), 此时 JVM 就会将加锁的范围粗化到这一连串的操作的外部(比如 while 虚幻体外), 使得这一连串操作只需要加一次锁即可. 



### 锁升级流程图

<img src="http://www.milky.show/images/java/synchronized/syn_7.png" alt="http://www.milky.show/images/java/synchronized/syn_7.png" />



对比如下: 

| 锁状态   | 优点                                               | 缺点                                     | 适用场景                                       |
| :------- | :------------------------------------------------- | :--------------------------------------- | :--------------------------------------------- |
| 偏向锁   | 加锁解锁无需额外消耗, 跟非同步方法时间相差纳秒级别 | 如果竞争线程多, 会带来额外的锁撤销的消耗 | 基本没有其他线程竞争的同步场景                 |
| 轻量级锁 | 竞争的线程不会阻塞而是在自旋, 可提高程序响应速度   | 如果一直无法获得会自旋消耗CPU            | 少量线程竞争, 持有锁时间不长, 追求响应速度     |
| 重量级锁 | 线程竞争不会导致CPU自旋跟消耗CPU资源               | 线程阻塞, 响应时间长                     | 很多线程竞争锁, 切锁持有时间长, 追求吞吐量时候 |

PS: ReentrantLock底层实现依赖于特殊的CPU指令, 比如发送lock指令和unlock指令, 不需要用户态和内核态的切换, 所以效率高. 而synchronized底层由监视器锁(monitor)是依赖于底层的操作系统的`Mutex Lock`需要用户态和内核态的切换, 所以效率会低一些. 



偏向锁 - markword 上记录当前线程指针, 下次同一个线程加锁的时候, 不需要争用, 只需要判断线程指针是否同一个, 所以, 偏向锁, 偏向加锁的第一个线程 . hashCode备份在线程栈上 线程销毁, 锁降级为无锁

有争用 - 锁升级为轻量级锁 - 每个线程有自己的LockRecord在自己的线程栈上, 用CAS去争用markword的LR的指针, 指针指向哪个线程的LR, 哪个线程就拥有锁

自旋超过10次, 升级为重量级锁 - 如果太多线程自旋 CPU消耗过大, 不如升级为重量级锁, 进入等待队列(不消耗CPU)-XX:PreBlockSpin



自旋锁在 JDK1.4.2 中引入, 使用 -XX:+UseSpinning 来开启. JDK 6 中变为默认开启, 并且引入了自适应的自旋锁(适应性自旋锁). 

自适应自旋锁意味着自旋的时间(次数)不再固定, 而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定. 如果在同一个锁对象上, 自旋等待刚刚成功获得过锁, 并且持有锁的线程正在运行中, 那么虚拟机就会认为这次自旋也是很有可能再次成功, 进而它将允许自旋等待持续相对更长的时间. 如果对于某个锁, 自旋很少成功获得过, 那在以后尝试获取这个锁时将可能省略掉自旋过程, 直接阻塞线程, 避免浪费处理器资源. 



偏向锁由于有锁撤销的过程 revoke, 会消耗系统资源, 所以, 在锁争用特别激烈的时候, 用偏向锁未必效率高. 还不如直接使用轻量级锁. 



### 自旋锁一定比重量级锁效率高吗

自旋是消耗CPU资源的, 如果锁的时间长, 或者自旋线程多, CPU会被大量消耗

重量级锁有等待队列, 所有拿不到锁的进入等待队列, 不需要消耗CPU资源



### 偏向锁一定比自旋锁效率高吗

**偏向锁是在 JVM 启动 4s 后开启的, 因为 JVM 启动时有很多内存区域需要进行加锁, 这时已经明确知道有多线程去竞争锁, 就不需要开启偏向锁了**

很多情况下我明知道会存在锁的竞争情况, 就不需要加偏向锁了, 如果加了偏向锁还存在一个偏向锁升级的过程, 反而效率会降低

不一定, 在明确知道会有多线程竞争的情况下, 偏向锁肯定会涉及锁撤销, 这时候直接使用自旋锁

JVM启动过程, 会有很多线程竞争(明确), 所以默认情况启动时不打开偏向锁, 过一段儿时间再打开

所以, 当创建对象实例时, 如果偏向锁未启动, 则创建的是普通对象实例, 如果偏向锁已启动, 则为匿名偏向. 



### 用 synchronized 还是 Lock 呢? 

我们先看看他们的区别: 

- synchronized 是关键字, 是 JVM 层面的底层啥都帮我们做了, 而Lock是一个接口, 是JDK层面的有丰富的API. 
- synchronized 会自动释放锁, 而 Lock 必须手动释放锁. 
- synchronized 是不可中断的, Lock 可以中断也可以不中断. 
- 通过 Lock 可以知道线程有没有拿到锁, 而 synchronized 不能. 
- synchronized 能锁住方法和代码块, 而 Lock 只能锁住代码块. 
- Lock 可以使用读锁提高多线程读效率. 
- synchronized 是非公平锁, ReentrantLock 可以控制是否是公平锁. 

两者一个是 JDK 层面的一个是 JVM 层面的, 我觉得最大的区别其实在, 我们是否需要丰富的 api, 还有一个我们的场景. 

**比如我现在是滴滴, 我早上有打车高峰, 我代码使用了大量的 synchronized, 有什么问题? 锁升级过程是不可逆的, 过了高峰我们还是重量级的锁, 那效率是不是大打折扣了? 这个时候你用Lock是不是很好? **

场景是一定要考虑的, 我现在告诉你哪个好都是扯淡, 因为脱离了业务, 一切技术讨论都没有了价值. 

在高争用 高耗时的环境下synchronized效率更高

在低争用 低耗时的环境下CAS效率更高

synchronized到重量级之后是等待队列(不消耗CPU)CAS(等待期间消耗CPU)

 一切以实测为准





## 请描述锁的四种状态和升级过程

在 JDK1.2 时 synchronized 的性能非常差, 在 JDK1.6 后优化了锁的升级状态

**JDK 早期, synchronized 叫做重量级锁, 因为申请锁资源必须通过 kernel(内核), 早期 APP 可以直接系统调用底层硬件**

现代操作系统会分为两层, 内核态和用户态, 自己的 APP 就是用户态, 系统资源就是内核态, 如果用户态想访问内核态的资源需要通过 kernel 的允许, 将用户态转向内核态, 拿到结果后再从内核态返回用户态. 早期 synchronized 加锁就需要申请系统资源, 进行用户态和内核态的转换, 所以是重量级锁, 经过优化后在某些情况下不需要通过内核态就可以解决, 比如 CAS(轻量级锁), 只需要用户态就可以完成

<img src="http://www.milky.show/images/mashibing/synchronized/syn_3.png" alt="http://www.milky.show/images/mashibing/synchronized/syn_3.png" style="zoom: 33%;" />

### 无锁态

最后是 001

### 偏向锁

最后是 101, 用户态完成

将自己的线程号写进了 markword 上, 此时只是贴了一个自己的 id 上去, 并没有发生锁的竞争

多数Sychronized方法, 在很多情况下, 其实只有一个线程运行, 例如: StringBuffer中的一些sync方法(append()方法), Vector中的一些sync方法

**只要有另外的线程来竞争, 就会升级为轻量级锁**

具体操作是**markword上记录当前线程指针, 下次同一个线程加锁的时候, 不需要争用, 只用判断线程指针是否是同一个, 所以, 偏向锁, 偏向的是加锁的第一个线程, hashCode备份在线程栈上, 线程销毁, 锁降级为无锁. **

一旦有其他线程竞争, 偏向锁就会升级为轻量级锁, 每个线程有自己的LockRecord在自己的线程栈上, 用CAS去争用markword的LR的指针, 指针指向哪个线程的LR, 哪个线程就拥有锁. 如果自旋次数超过十次, 升级为重量级锁. 或者有多个线程在等待, 即超过1/2cpu核数, 升级为重量级锁. 以前通过参数可以调, 现在JVM做了优化, 自适应自旋, 合适的时候自己升级为重量锁. 

**偏向锁默认是启动的, 但会默认延迟4秒钟. 因为JVM虚拟机自己有一些默认启动的线程, 里面有好多sync代码, 这些sync代码启动时就知道肯定会有竞争, 如果使用偏向锁, 就会造成偏向锁不断的进行锁撤销, 锁升级的操作, 效率较低. **

### 轻量级锁, 自旋锁, 无锁(CAS)

最后是 00, 用户态完成

当发生了多线程争锁的情况, 哪个线程能将自己的 id 写进 markword 谁就能获得锁, 此时是在用户空间完成的 CAS 操作, 此时偏向锁的线程也会发起争锁, 但是会有一定优势

CAS 适合操作特别快或者线程数较小的情况, 因为自旋会消耗 CPU 资源

轻量级锁的应用场景往往是锁执行的时间短, 自旋少次就能拿到锁, 或线程数少, 竞争数少. 自旋锁是需要占用CPU资源的(while循环), 而重量级锁有等待队列, 所有拿不到锁的进入等待队列, 不需要消耗CPU资源. 



### 重量级锁

最后是 10, 需要向内核态申请

向操作系统申请锁

为什么有了自旋锁, 还需要重量级锁, 因为重量级锁不需要消耗 CPU 资源, 会将其他线程放到一个队列中(waitset),  因此不需要自旋的过程了



## Lock

ReentrantLock, 重入锁, 是JDK5中添加在并发包下的一个高性能的工具。顾名思义, ReentrantLock支持同一个线程在未释放锁的情况下重复获取锁。

每一个东西的出现一定是有价值的。既然已经有了元老级的synchronized, 而且synchronized也支持重入, 为什么Doug Lea还要专门写一个ReentrantLock呢？

### ReentrantLock 与 synchronized 的比较

**性能上的比较**

首先, ReentrantLock 的性能要优于 synchronized。下面通过两段代码比价一下。 首先是 synchronized: 

```java
public class LockDemo2 {
    private static final Object lock = new Object(); // 定义锁对象
    private static int count = 0; // 累加数
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        CountDownLatch cdl = new CountDownLatch(100);
        // 启动100个线程对count累加, 每个线程累加1000000次
        // 调用add函数累加, 通过synchronized保证多线程之间的同步
        for (int i=0;i<100;i++) {
            new Thread(() -> {
                for (int i1 = 0; i1 <1000000; i1++) {
                    add();
                }
                cdl.countDown();
            }).start();
        }
        cdl.await();
        System.out.println("Time cost: " + (System.currentTimeMillis() - start) + ", count = " + count);
    }

    private static void add() {
        synchronized (lock) {
            count++;
        }
    }
}
```

然后是 ReentrantLock: 

```java
public class LockDemo3 {
    private static Lock lock = new ReentrantLock(); // 重入锁
    private static int count = 0;
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        CountDownLatch cdl = new CountDownLatch(100);
        for (int i=0;i<100;i++) {
            new Thread(() -> {
                for (int i1 = 0; i1 <1000000; i1++) {
                    add();
                }
                cdl.countDown();
            }).start();
        }
        cdl.await();
        System.out.println("Time cost: " + (System.currentTimeMillis() - start) + ", count = " + count);
    }
    // 通过ReentrantLock保证线程之间的同步
    private static void add() {
        lock.lock();
        count++;
        lock.unlock();
    }
}
```

下面是运行多次的结果对比: 

|        | synchronized | ReentrantLock |
| ------ | ------------ | ------------- |
| 第一次 | 4620ms       | 3360ms        |
| 第二次 | 4086ms       | 3138ms        |
| 第三次 | 4650ms       | 3408ms        |

总体来看, ReentrantLock的平均性能要比synchronized好20%左右。

> 更严谨的描述一下这个性能的对比: 当存在大量线程竞争锁时, 多数情况下ReentrantLock的性能优于synchronized。
> 因为在JDK6中对synchronized做了优化, 在锁竞争不激烈的时候, 多数情况下锁会停留在偏向锁和轻量级锁阶段, 这两个阶段性能是很好的。当存在大量竞争时, 可能会膨胀为重量级锁, 性能下降, 此时的ReentrantLock应该是优于synchronized的。

**获取锁公平性的比较**

公平性是啥概念呢？如果是公平的获取锁, 就是说多个线程之间获取锁的时候要排队, 依次获取锁；如果是不公平的获取锁, 就是说多个线程获取锁的时候一哄而上, 谁抢到是谁的。

由于synchronized是基于monitor机制实现的, 它只支持非公平锁；但ReentrantLock同时支持公平锁和非公平锁。

**综述**

除了上文所述, ReentrantLock还有一些其他synchronized不具备的特性, 这里来总结一下。

|                  | synchronized                                         | ReentrantLock                                                |
| ---------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| 性能             | 相对较差                                             | 优于 synchronized 20% 左右                                   |
| 公平性           | 只支持非公平锁                                       | 同时支持公平锁与非公平锁                                     |
| 尝试获取锁的支持 | 不支持, 一旦到了同步块, 且没有获取到锁, 就阻塞在这里 | 支持, 通过 tryLock 方法实现, 可通过其返回值判断是否成功获取锁, 所以即使获取锁失败也不会阻塞在这里 |
| 超时的获取锁     | 不支持, 如果一直获取不到锁, 就会一直等待下去         | 支持, 通过 tryLock(time, TimeUnit) 方法实现, 如果超时了还没获取锁, 就放弃获取锁, 不会一直阻塞 |
| 是否可响应中断   | 不支持, 不可响应线程的 interrupt 信号                | 支持, 通过 lockInterruptibly 方法实现, 通过此方法获取锁之后, 线程可以响应 interrupt 信号, 并抛出 InterruptedException 异常 |
| 等待条件的支持   | 支持, 通过 wait, notify, notifyAll 来实现            | 支持, 通过 Condition 接口实现, 支持多个 Condition, 比 synchronized 更灵活 |





### 可重入功能的实现原理

ReentrantLock的实现基于队列同步器（AbstractQueuedSynchronizer, 后面简称AQS）, 其原理大致为: 当某一线程获取锁后, 将state值+1, 并记录下当前持有锁的线程, 再有线程来获取锁时, 判断这个线程与持有锁的线程是否是同一个线程, 如果是, 将state值再+1, 如果不是, 阻塞线程。 当线程释放锁时, 将state值-1, 当state值减为0时, 表示当前线程彻底释放了锁, 然后将记录当前持有锁的线程的那个字段设置为null, 并唤醒其他线程, 使其重新竞争锁。

```java
// acquires的值是1
final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取state的值
    int c = getState();
    // 如果state的值等于0, 表示当前没有线程持有锁
    // 尝试将state的值改为1, 如果修改成功, 则成功获取锁, 并设置当前线程为持有锁的线程, 返回true
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // state的值不等于0, 表示已经有其他线程持有锁
    // 判断当前线程是否等于持有锁的线程, 如果等于, 将state的值+1, 并设置到state上, 获取锁成功, 返回true
    // 如果不是当前线程, 获取锁失败, 返回false
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

### 非公平锁的实现原理

ReentrantLock有两个构造函数: 

```java
// 无参构造, 默认使用非公平锁（NonfairSync）
public ReentrantLock() {
    sync = new NonfairSync();
}

// 通过fair参数指定使用公平锁（FairSync）还是非公平锁（NonfairSync）
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

sync是ReentrantLock的成员变量, 是其内部类Sync的实例。NonfairSync和FairSync都是Sync类的子类。可以参考如下类关系图: 

![http://www.milky.show/images/java/lock/lock_2.png](http://www.milky.show/images/java/lock/lock_2.png)

Sync继承了AQS, 所以他具备了AQS的功能。同样的, NonfairSync和FairSync都是AQS的子类。

当我们通过无参构造函数获取ReentrantLock实例后, 默认用的就是非公平锁。

下面将通过如下场景描述非公平锁的实现原理: 假设一个线程(t1)获取到了锁, 其他很多没获取到锁的线程(others_t)加入到了AQS的同步队列中等待, 当这个线程执行完, 释放锁后, 其他线程重新非公平的竞争锁。

先来描述一下获取锁的方法: 

```java
final void lock() {
    // 线程t1成功的将state的值从0改为1, 表示获取锁成功
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        // others_t线程们没有获取到锁
        acquire(1);
}
```

如果获取锁失败, 会调用AQS的acquire方法

```java
public final void acquire(int arg) {
    // tryAcquire是个模板方法, 在NonfairSync中实现, 如果在tryAcquire方法中依然获取锁失败, 会将当前线程加入同步队列中等待（addWaiter）
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

tryAcquire的实现如下, 其实是调用了上面的nonfairTryAcquire方法

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

OK, 此时t1获取到了锁, others_t线程们都跑到同步队列里等着了。

某一时刻, t1自己的任务执行完成, 调用了释放锁的方法（unlock）。

```java
public void unlock() {
    // 调用AQS的release方法释放资源
    sync.release(1);
}
public final boolean release(int arg) {
    // tryRelease也是模板方法, 在Sync中实现
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 成功释放锁后, 唤醒同步队列中的下一个节点, 使之可以重新竞争锁
            // 注意此时不会唤醒队列第一个节点之后的节点, 这些节点此时还是无法竞争锁
            unparkSuccessor(h);
        return true;
    }
    return false;
}
protected final boolean tryRelease(int releases) {
    // 将state的值-1, 如果-1之后等于0, 释放锁成功
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

这时锁被释放了, 被唤醒的线程和新来的线程重新竞争锁（不包含同步队列后面的那些线程）。

回到lock方法中, 由于此时所有线程都能通过CAS来获取锁, 并不能保证被唤醒的那个线程能竞争过新来的线程, 所以是非公平的。这就是非公平锁的实现。

这个过程大概可以描述为下图这样子: 

![http://www.milky.show/images/java/lock/lock_3.png](http://www.milky.show/images/java/lock/lock_3.png)

### 公平锁的实现原理

公平锁与非公平锁的释放锁的逻辑是一样的, 都是调用上述的unlock方法, 最大区别在于获取锁的时候。

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;
    // 获取锁, 与非公平锁的不同的地方在于, 这里直接调用的AQS的acquire方法, 没有先尝试获取锁
    // acquire又调用了下面的tryAcquire方法, 核心在于这个方法
    final void lock() {
        acquire(1);
    }

    /**
     * 这个方法和nonfairTryAcquire方法只有一点不同, 在标注为#1的地方
     * 多了一个判断hasQueuedPredecessors, 这个方法是判断当前AQS的同步队列中是否还有等待的线程
     * 如果有, 返回true, 否则返回false。
     * 由此可知, 当队列中没有等待的线程时, 当前线程才能尝试通过CAS的方式获取锁。
     * 否则就让这个线程去队列后面排队。
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // #1
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

通过注释可知, 在公平锁的机制下, 任何线程想要获取锁, 都要排队, 不可能出现插队的情况。这就是公平锁的实现原理。

这个过程大概可以描述为下图这样子: 

![http://www.milky.show/images/java/lock/lock_4.png](http://www.milky.show/images/java/lock/lock_4.png)

### tryLock原理

tryLock做的事情很简单: 让当前线程尝试获取一次锁, 成功的话返回true, 否则false。

其实现, 其实就是调用了nonfairTryAcquire方法来获取锁。

```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

至于获取失败的话, 他也不会将自己添加到同步队列中等待, 直接返回false, 让业务调用代码自己处理。

### 可中断的获取锁

中断, 也就是通过Thread的interrupt方法将某个线程中断, 中断一个阻塞状态的线程, 会抛出一个InterruptedException异常。

如果获取锁是可中断的, 当一个线程长时间获取不到锁时, 我们可以主动将其中断, 可避免死锁的产生。

其实现方式如下: 

```java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

会调用AQS的acquireInterruptibly方法

```java
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    // 判断当前线程是否已经中断, 如果已中断, 抛出InterruptedException异常
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

此时会优先通过tryAcquire尝试获取锁, 如果获取失败, 会将自己加入到队列中等待, 并可随时响应中断。

```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    // 将自己添加到队列中等待
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        // 自旋的获取锁
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            // 获取锁失败, 在parkAndCheckInterrupt方法中, 通过LockSupport.park()阻塞当前线程, 
            // 并调用Thread.interrupted()判断当前线程是否已经被中断
            // 如果被中断, 直接抛出InterruptedException异常, 退出锁的竞争队列
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // #1
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

PS: 不可中断的方式下, 代码#1位置不会抛出InterruptedException异常, 只是简单的记录一下当前线程被中断了。



### 可超时的获取锁

通过如下方法实现, timeout 是超时时间, unit 代表时间的单位（毫秒、秒...）

```java
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

可以发现, 这也是一个可以响应中断的方法。然后调用AQS的tryAcquireNanos方法: 

```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
```

doAcquireNanos 方法与中断里面的方法大同小异, 下面在注释中说明一下不同的地方: 

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    // 计算超时截止时间
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            // 计算到截止时间的剩余时间
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L) // 超时了, 获取失败
                return false;
            // 超时时间大于1000纳秒时, 才阻塞
            // 因为如果小于1000纳秒, 基本可以认为超时了（系统调用的时间可能都比这个长）
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            // 响应中断
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 总结

本文首先对比了元老级的锁synchronized与ReentrantLock的不同, ReentrantLock具有一下优势:  *同时支持公平锁与非公平锁* 支持: 尝试非阻塞的一次性获取锁 *支持超时获取锁* 支持可中断的获取锁 * 支持更多的等待条件（Condition）

然后介绍了几个主要特性的实现原理, 这些都是基于AQS的。

ReentrantLock的核心, 是通过修改AQS中state的值来同步锁的状态。 通过这个方式, 实现了可重入。

ReentrantLock具备公平锁和非公平锁, 默认使用非公平锁。其实现原理主要依赖于AQS中的同步队列。

最后, 可中断的机制是内部通过Thread.interrupted()判断当前线程是否已被中断, 如果被中断就抛出InterruptedException异常来实现的。





## 请描述 synchronized 和 Reentrantlock 的底层实现及重入的底层原理

synchronized 和 Reentrantlock 的底层汇编还是 lock cmpxchg 命令

原理弄清楚了, 顺便总结了几点 Synchronized 和 ReentrantLock 的区别

1.  Synchronized 是 JVM 层次的锁实现, ReentrantLock 是 JDK 层次的锁实现
2.  Synchronized 的锁状态是无法在代码中直接判断的, 但是 ReentrantLock 可以通过`ReentrantLock#isLocked`判断
3.  Synchronized 是非公平锁, ReentrantLock 是可以是公平也可以是非公平的
4.  Synchronized 是不可以被中断的, 而`ReentrantLock#lockInterruptibly`方法是可以被中断的
5.  在发生异常时 Synchronized 会自动释放锁(由 javac 编译时自动实现), 而 ReentrantLock 需要开发者在finally块中显示释放锁
6.  ReentrantLock 获取锁的形式有多种: 如立即返回是否成功的 tryLock(),以及等待指定时长的获取, 更加灵活
7.  Synchronized 在特定的情况下**对于已经在等待的线程**是后来的线程先获得锁(上文有说), 而 ReentrantLock 对于**已经在等待的线程**一定是先来的线程先获得锁
