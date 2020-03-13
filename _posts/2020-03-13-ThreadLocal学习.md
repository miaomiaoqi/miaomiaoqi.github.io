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