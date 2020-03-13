---
layout: post
title: ThreadLocal 学习
categories: [Java]
description: 
keywords: 
---

* content
{:toc}


## 简介

ThreadLocal 类提供了线程局部 (thread-local) 变量。这些变量与普通变量不同，每个线程都可以通过其 get 或 set方法来**访问自己的独立初始化的变量副本**。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。

ThreadLocal顾名思义是线程私有的局部变量存储容器，可以理解成每个线程都有自己专属的存储容器，它用来存储线程私有变量，其实它只是一个外壳，内部真正存取是一个Map，后面会仔细讲解。每个线程可以通过`set()`和`get()`存取变量，多线程间无法访问各自的局部变量，相当于在每个线程间建立了一个隔板。只要线程处于活动状态，它所对应的ThreadLocal实例就是可访问的，线程被终止后，它的所有实例将被垃圾收集。总之记住一句话：**ThreadLocal存储的变量属于当前线程**。

## 整体认识

![http://www.milky.show/images/java/threadlocal/th_1.png](http://www.milky.show/images/java/threadlocal/th_1.png)

ThreadLocal 中的嵌套内部类 ThreadLocalMap，这个类本质上是一个 map，和 HashMap 之类的实现相似，依然是 key-value 的形式，其中有一个内部类 Entry，其中 key 可以看做是 ThreadLocal 实例，**但是其本质是持有 ThreadLocal 实例的弱引用**

在 ThreadLocal 中并没有对于 ThreadLocalMap 的引用，ThreadLocalMap 的引用在 Thread 类中，代码如下。每个线程在向 ThreadLocal 里塞值的时候，其实都是向自己所持有的 ThreadLocalMap 里塞入数据；读的时候同理，首先从自己线程中取出自己持有的 ThreadLocalMap，然后再根据 ThreadLocal 引用作为 key 取出 value，基于以上描述，ThreadLocal 实现了变量的线程隔离（当然，毕竟变量其实都是从自己当前线程实例中取出来的）。

```java
public class Thread implements Runnable {
    ......
    
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
    ......
}
```

## ThreadLocalMap源码分析

下面一步一步介绍ThreadLocalMap源码分析的相关源码，在分析ThreadLocalMap的同时，也会介绍与ThreadLocalMap关联的ThreadLocal中的方法（这样分析完ThreadLocalMap，ThreadLocal基本就搞定了），可能有些需要前后结合才能真正理解。

### 成员变量

```java
/**
* 初始容量 —— 必须是2的冥
*/
private static final int INITIAL_CAPACITY = 16;

/**
* 存放数据的table，Entry类的定义在下面分析
* 同样，数组长度必须是2的冥。
*/
private Entry[] table;

/**
* 数组里面entrys的个数，可以用于判断table当前使用量是否超过负因子。
*/
private int size = 0;

/**
* 进行扩容的阈值，表使用量大于它的时候进行扩容。
*/
private int threshold; // Default to 0

/**
* 定义为长度的2/3
*/
private void setThreshold(int len) {
	threshold = len * 2 / 3;
}
```

### 存储结构\-\-Entry

```java
/**
 * Entry继承WeakReference，并且用ThreadLocal作为key.如果key为null
 * (entry.get() == null)表示key不再被引用，表示ThreadLocal对象被回收
 * 因此这时候entry也可以从table从清除。
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Entry继承WeakReference,使用弱引用，可以将ThreadLocal对象的生命周期和线程生命周期解绑，持有对ThreadLocal的弱引用，可以使得ThreadLocal在没有其他强引用的时候被回收掉，这样可以避免因为线程得不到销毁导致ThreadLocal对象无法被回收。

### ThreadLocal中的set()

先看看ThreadLocal中set()源码。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocal.ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocal.ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocal.ThreadLocalMap(this, firstValue);
}
```

*   代码很简单，获取当前线程，并获取当前线程的ThreadLocalMap实例（从getMap(Thread t)中很容易看出来）。

*   如果获取到的map实例不为空，调用map.set()方法，否则调用构造函数 ThreadLocal.ThreadLocalMap(this, firstValue)实例化map。

可以看出来线程中的ThreadLocalMap使用的是延迟初始化，在第一次调用get()或者set()方法的时候才会进行初始化。下面来看看构造函数`ThreadLocalMap(ThreadLocal firstKey, Object firstValue)` 。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //初始化table
    table = new ThreadLocal.ThreadLocalMap.Entry[INITIAL_CAPACITY];
    //计算索引
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    //设置值
    table[i] = new ThreadLocal.ThreadLocalMap.Entry(firstKey, firstValue);
    size = 1;
    //设置阈值
    setThreshold(INITIAL_CAPACITY);
}
```

主要说一下计算索引，`firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1)`。

*   关于`& (INITIAL_CAPACITY - 1)`,这是取模的一种方式，对于2的幂作为模数取模，用此代替`%(2^n)`，这也就是为啥容量必须为2的幂，在这个地方也得到了解答，至于为什么可以这样这里不过多解释，原理很简单。

*   关于`firstKey.threadLocalHashCode`：

```java
private final int threadLocalHashCode = nextHashCode();

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
private static AtomicInteger nextHashCode = new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;
```

定义了一个AtomicInteger类型，每次获取当前值并加上HASH_INCREMENT，`HASH_INCREMENT = 0x61c88647`,关于这个值和`斐波那契散列`有关，其原理这里不再深究，感兴趣可自行搜索，其主要目的就是为了让哈希码能均匀的分布在2的n次方的数组里, 也就是`Entry[] table`中。

在了解了上面的源码后，终于能进入正题了，下面开始进入ThreadLocalMap中的set()。

### ThreadLocalMap中的哈希冲突

ThreadLocalMap使用`线性探测法`来解决哈希冲突，线性探测法的地址增量di = 1, 2, ... , m-1，其中，i为探测次数。该方法一次探测下一个地址，直到有空的地址后插入，若整个空间都找不到空余的地址，则产生溢出。假设当前table长度为16，也就是说如果计算出来key的hash值为14，如果table[14]上已经有值，并且其key与当前key不一致，那么就发生了hash冲突，这个时候将14加1得到15，取table[15]进行判断，这个时候如果还是冲突会回到0，取table[0],以此类推，直到可以插入。

按照上面的描述，`可以把table看成一个环形数组`。

先看一下线性探测相关的代码，从中也可以看出来table实际是一个环：

```java
/**
 * 获取环形数组的下一个索引
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * 获取环形数组的上一个索引
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

### ThreadLocalMap的set()及其set()相关代码如下

```java
private void set(ThreadLocal<?> key, Object value) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    //计算索引，上面已经有说过。
    int i = key.threadLocalHashCode & (len - 1);

    /**
     * 根据获取到的索引进行循环，如果当前索引上的table[i]不为空，在没有return的情况下，
     * 就使用nextIndex()获取下一个（上面提到到线性探测法）。
     */
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // table[i]上key不为空，并且和当前key相同，更新value
        if (k == key) {
            e.value = value;
            return;
        }
        /**
         * table[i]上的key为空，说明被回收了（上面的弱引用中提到过）。
         * 这个时候说明改table[i]可以重新使用，用新的key-value将其替换,并删除其他无效的entry
         */
        if (k == null) {
            this.replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 找到为空的插入位置，插入值，在为空的位置插入需要对size进行加1操作
    tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
    int sz = ++size;

    /**
     * cleanSomeSlots用于清除那些e.get()==null，也就是table[index] != null && table[index].get()==null
     * 之前提到过，这种数据key关联的对象已经被回收，所以这个Entry(table[index])可以被置null。
     * 如果没有清除任何entry,并且当前使用量达到了负载因子所定义(长度的2/3)，那么进行rehash()
     */
    if (!this.cleanSomeSlots(i, sz) && sz >= threshold) {
        this.rehash();
    }
}


/**
 * 替换无效entry
 */
private void replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    ThreadLocal.ThreadLocalMap.Entry e;

    /**
     * 根据传入的无效entry的位置（staleSlot）,向前扫描
     * 一段连续的entry(这里的连续是指一段相邻的entry并且table[i] != null),
     * 直到找到一个无效entry，或者扫描完也没找到
     */
    int slotToExpunge = staleSlot;//之后用于清理的起点
    for (int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len)) {
        if (e.get() == null) {
            slotToExpunge = i;
        }
    }

    /**
     * 向后扫描一段连续的entry
     */
    for (int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();

        /**
         * 如果找到了key，将其与传入的无效entry替换，也就是与table[staleSlot]进行替换
         */
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            //如果向前查找没有找到无效entry，则更新slotToExpunge为当前值i
            if (slotToExpunge == staleSlot) {
                slotToExpunge = i;
            }
            this.cleanSomeSlots(this.expungeStaleEntry(slotToExpunge), len);
            return;
        }

        /**
         * 如果向前查找没有找到无效entry，并且当前向后扫描的entry无效，则更新slotToExpunge为当前值i
         */
        if (k == null && slotToExpunge == staleSlot) {
            slotToExpunge = i;
        }
    }

    /**
     * 如果没有找到key,也就是说key之前不存在table中
     * 就直接最开始的无效entry——tab[staleSlot]上直接新增即可
     */
    tab[staleSlot].value = null;
    tab[staleSlot] = new ThreadLocal.ThreadLocalMap.Entry(key, value);

    /**
     * slotToExpunge != staleSlot,说明存在其他的无效entry需要进行清理。
     */
    if (slotToExpunge != staleSlot) {
        this.cleanSomeSlots(this.expungeStaleEntry(slotToExpunge), len);
    }
}

/**
 * 连续段清除
 * 根据传入的staleSlot,清理对应的无效entry——table[staleSlot],
 * 并且根据当前传入的staleSlot,向后扫描一段连续的entry(这里的连续是指一段相邻的entry并且table[i] != null),
 * 对可能存在hash冲突的entry进行rehash，并且清理遇到的无效entry.
 *
 * @param staleSlot key为null,需要无效entry所在的table中的索引
 * @return 返回下一个为空的solt的索引。
 */
private int expungeStaleEntry(int staleSlot) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;

    // 清理无效entry，置空
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    //size减1，置空后table的被使用量减1
    size--;

    ThreadLocal.ThreadLocalMap.Entry e;
    int i;
    /**
     * 从staleSlot开始向后扫描一段连续的entry
     */
    for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        //如果遇到key为null,表示无效entry，进行清理.
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            //如果key不为null,计算索引
            int h = k.threadLocalHashCode & (len - 1);
            /**
             * 计算出来的索引——h，与其现在所在位置的索引——i不一致，置空当前的table[i]
             * 从h开始向后线性探测到第一个空的slot，把当前的entry挪过去。
             */
            if (h != i) {
                tab[i] = null;
                while (tab[h] != null) {
                    h = nextIndex(h, len);
                }
                tab[h] = e;
            }
        }
    }
    //下一个为空的solt的索引。
    return i;
}

/**
 * 启发式的扫描清除，扫描次数由传入的参数n决定
 *
 * @param i 从i向后开始扫描（不包括i，因为索引为i的Slot肯定为null）
 *
 * @param n 控制扫描次数，正常情况下为 log2(n) ，
 * 如果找到了无效entry，会将n重置为table的长度len,进行段清除。
 *
 * map.set()点用的时候传入的是元素个数，replaceStaleEntry()调用的时候传入的是table的长度len
 *
 * @return true if any stale entries have been removed.
 */
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        ThreadLocal.ThreadLocalMap.Entry e = tab[i];
        if (e != null && e.get() == null) {
            //重置n为len
            n = len;
            removed = true;
            //依然调用expungeStaleEntry来进行无效entry的清除
            i = this.expungeStaleEntry(i);
        }
    } while ((n >>>= 1) != 0);//无符号的右移动，可以用于控制扫描次数在log2(n)
    return removed;
}


private void rehash() {
    //全清理
    this.expungeStaleEntries();

    /**
     * threshold = 2/3 * len
     * 所以threshold - threshold / 4 = 1en/2
     * 这里主要是因为上面做了一次全清理所以size减小，需要进行判断。
     * 判断的时候把阈值调低了。
     */
    if (size >= threshold - threshold / 4) {
        this.resize();
    }
}

/**
 * 扩容，扩大为原来的2倍（这样保证了长度为2的冥）
 */
private void resize() {
    ThreadLocal.ThreadLocalMap.Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    ThreadLocal.ThreadLocalMap.Entry[] newTab = new ThreadLocal.ThreadLocalMap.Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        ThreadLocal.ThreadLocalMap.Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            //虽然做过一次清理，但在扩容的时候可能会又存在key==null的情况。
            if (k == null) {
                //这里试试将e.value设置为null
                e.value = null; // Help the GC
            } else {
                //同样适用线性探测来设置值。
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null) {
                    h = nextIndex(h, newLen);
                }
                newTab[h] = e;
                count++;
            }
        }
    }

    //设置新的阈值
    setThreshold(newLen);
    size = count;
    table = newTab;
}

/**
 * 全清理，清理所有无效entry
 */
private void expungeStaleEntries() {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        ThreadLocal.ThreadLocalMap.Entry e = tab[j];
        if (e != null && e.get() == null)
        //使用连续段清理
        {
            this.expungeStaleEntry(j);
        }
    }
}
```

### ThreadLocal中的get()

```java
public T get() {
    // 同set方法类似获取对应线程中的ThreadLocalMap实例
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            T result = (T) e.value;
            return result;
        }
    }
    // 为空返回初始化值
    return this.setInitialValue();
}

/**
 * 初始化设值的方法，可以被子类覆盖。
 */
protected T initialValue() {
    return null;
}

private T setInitialValue() {
    // 获取初始化值，默认为null(如果没有子类进行覆盖)
    T value = this.initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    //不为空不用再初始化，直接调用set操作设值
    if (map != null) {
        map.set(this, value);
    } else {
        // 第一次初始化，createMap在上面介绍set()的时候有介绍过。
        createMap(t, value);
    }
    return value;
}
```

### ThreadLocalMap中的getEntry()

```java
private ThreadLocal.ThreadLocalMap.Entry getEntry(ThreadLocal<?> key) {
    //根据key计算索引，获取entry
    int i = key.threadLocalHashCode & (table.length - 1);
    ThreadLocal.ThreadLocalMap.Entry e = table[i];
    if (e != null && e.get() == key) {
        return e;
    } else {
        return this.getEntryAfterMiss(key, i, e);
    }
}

/**
 * 通过直接计算出来的key找不到对于的value的时候适用这个方法.
 */
private ThreadLocal.ThreadLocalMap.Entry getEntryAfterMiss(ThreadLocal<?> key, int i, ThreadLocal.ThreadLocalMap.Entry e) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key) {
            return e;
        }
        if (k == null) {
            //清除无效的entry
            expungeStaleEntry(i);
        } else {
            //基于线性探测法向后扫描
            i = nextIndex(i, len);
        }
        e = tab[i];
    }
    return null;
}
```

### ThreadLocalMap中的remove()

```java
private void remove(ThreadLocal<?> key) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    // 计算索引
    int i = key.threadLocalHashCode & (len - 1);
    // 进行线性探测，查找正确的key
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            // 调用weakrefrence的clear()清除引用
            e.clear();
            // 连续段清除
            expungeStaleEntry(i);
            return;
        }
    }
}
```

remove()在有上面了解后可以说极为简单了，就是找到对应的table[],调用weakrefrence的clear()清除引用，然后再调用expungeStaleEntry()进行清除。



## ThreadLocal对象的回收

Entry是一个弱引用，是因为它不会影响到ThreadLocal的GC行为，如果是强引用的话，在线程运行过程中，我们不再使用ThreadLocal了，将ThreadLocal置为null，但ThreadLocal在线程的ThreadLocalMap里还有引用，导致其无法被GC回收。而Entry声明为WeakReference，ThreadLocal置为null后，线程的ThreadLocalMap就不算强引用了，ThreadLocal就可以被GC回收了。



## ThreadLocal 六连问

### 内存泄漏原因探索

ThreadLocalMap的生命周期是与线程一样的，但是ThreadLocal却不一定，可能ThreadLocal使用完了就想要被回收，但是此时线程可能不会立即终止，还会继续运行（比如线程池中线程重复利用），如果ThreadLocal对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

如果ThreadLocal没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来ThreadLocalMap中使用这个ThreadLocal的key也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现key为null的value

key为空的话value是无效数据，久而久之，value累加就会导致内存泄漏。

### 怎么解决这个内存泄漏问题

在ThreadLocalMap中，调用 set()、get()、remove()方法的时候，会清理掉key为null的记录。在ThreadLocal设置为null之后，ThreadLocalMap中存在key为null的值，那么就可能发生内存泄漏，只有手动调用`remove()`方法来避免，**所以我们在使用完ThreadLocal变量时，尽量用threadLocal.remove()来清除，避免threadLocal=null的操作**。remove方法是彻底地回收该对象，而通过`threadLocal=null`只是释放掉了ThreadLocal的引用，但是在ThreadLocalMap中却还存在其Entry，后续还需处理。

### JDK开发者是如何避免内存泄漏的

ThreadLocal的设计者也意识到了这一点(内存泄漏)， 他们在一些方法中埋了对key=null的value擦除操作。

这里拿ThreadLocal提供的get()方法举例，它调用了ThreadLocalMap#getEntry()方法，对key进行了校验和对null key进行擦除。

```java
private ThreadLocal.ThreadLocalMap.Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    ThreadLocal.ThreadLocalMap.Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```

如果key为null， 则会调用getEntryAfterMiss()方法，在这个方法中，如果k == null ， 则调用expungeStaleEntry(i);方法。

expungeStaleEntry(i)方法完成了对key=null 的key所对应的value进行赋空， 释放了空间避免内存泄漏。

同时它遍历下一个key为空的entry， 并将value赋值为null， 等待下次GC释放掉其空间。

```java
private int expungeStaleEntry(int staleSlot) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    ThreadLocal.ThreadLocalMap.Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

同理， set()方法最终也是调用该方法(expungeStaleEntry)， 调用路径: 

```java
set 方法
set(T value) -> map.set(this, value) -> rehash() -> expungeStaleEntries()
remove方法
remove() -> ThreadLocalMap.remove(this) -> expungeStaleentries()
```

这样做， 也只能说尽可能避免内存泄漏， 但并不会完全解决内存泄漏这个问题。比如极端情况下我们只创建ThreadLocal但不调用set、get、remove方法等。所以最能解决问题的办法就是用完ThreadLocal后手动调用remove().



### 手动释放ThreadLocal遗留存储?你怎么去设计/实现？

这里主要是强化一下手动remove的思想和必要性，设计思想与连接池类似。

包装其父类remove方法为静态方法，如果是spring项目， 可以借助于bean的声明周期， 在拦截器的afterCompletion阶段进行调用。

**弱引用导致内存泄漏，那为什么key不设置为强引用**

这个问题就比较有深度了，是你谈薪的小小资本。

如果key设置为强引用， 当threadLocal实例释放后， threadLocal=null， 但是threadLocal会有强引用指向threadLocalMap，threadLocalMap.Entry又强引用threadLocal， 这样会导致threadLocal不能正常被GC回收。

弱引用虽然会引起内存泄漏， 但是也有set、get、remove方法操作对null key进行擦除的补救措施， 方案上略胜一筹。

线程执行结束后会不会自动清空Entry的value

一并考察了你的gc基础。

事实上，当currentThread执行结束后， threadLocalMap变得不可达从而被回收，Entry等也就都被回收了，但这个环境就要求不对Thread进行复用，但是我们项目中经常会复用线程来提高性能， 所以currentThread一般不会处于终止状态。

在 web 项目中, 我们使用 tomcat 容器, 一般会开启一个线程池, 线程的释放会归还到线程池中并不会真正释放, 如果在web项目中调用 ThreadLocal 的 set 方法 将一个对象放入Thread中的成员变量threadLocals(ThreadLocalMap) 中，那么这个对象(ThreadLocal)是永远不会被回收的，因为这个对象永远都被Thread中的成员变量threadLocals引用着。

如果想让垃圾收集器回收它，有两种方法

1.  将该线程从tomcat线程池中去除，当一个线程被回收的时候何况它的成员变量呢，但是tomcat启动一般都会配置一个线程池进行优化，所有该方法不太现实。

2.  调用 ThreadLocal 的 remove 方法 将对象从hread中的成员变量threadLocals 中删除掉。



### **Thread和ThreadLocal有什么联系呢**

Thread和ThreadLocal是绑定的， ThreadLocal依赖于Thread去执行， Thread将需要隔离的数据存放到ThreadLocal(准确的讲是ThreadLocalMap)中, 来实现多线程处理。



### Spring如何处理Bean多线程下的并发问题

ThreadLocal天生为解决相同变量的访问冲突问题， 所以这个对于spring的默认单例bean的多线程访问是一个完美的解决方案。spring也确实是用了ThreadLocal来处理多线程下相同变量并发的线程安全问题。

spring 如何保证数据库事务在同一个连接下执行的？

要想实现jdbc事务， 就必须是在同一个连接对象中操作， 多个连接下事务就会不可控， 需要借助分布式事务完成。那spring 如何保证数据库事务在同一个连接下执行的呢？

DataSourceTransactionManager 是spring的数据源事务管理器， 它会在你调用getConnection()的时候从数据库连接池中获取一个connection， 然后将其与ThreadLocal绑定， 事务完成后解除绑定。这样就保证了事务在同一连接下完成。



## ThreadLocal应用场景

处理较为复杂的业务时，使用ThreadLocal代替参数的显示传递。

ThreadLocal可以用来做数据库连接池保存Connection对象，这样就可以让线程内多次获取到的连接是同一个了（Spring中的DataSource就是使用的ThreadLocal）。

管理Session会话，将Session保存在ThreadLocal中，使线程处理多次处理会话时始终是一个Session。