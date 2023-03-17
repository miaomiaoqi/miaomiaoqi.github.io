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

ThreadLocal 类提供了线程局部 (thread-local) 变量. 这些变量与普通变量不同, 每个线程都可以通过其 get 或 set方法来**访问自己的独立初始化的变量副本**. ThreadLocal 实例通常是类中的 private static 字段, 它们希望将状态与某一个线程 (例如, 用户 ID 或事务 ID) 相关联. 

ThreadLocal 顾名思义是线程私有的局部变量存储容器, 可以理解成每个线程都有自己专属的存储容器, 它用来存储线程私有变量, 其实它只是一个外壳, 内部真正存取是一个 Map, 后面会仔细讲解. 每个线程可以通过 `set()` 和 `get()` 存取变量, 多线程间无法访问各自的局部变量, 相当于在每个线程间建立了一个隔板. 只要线程处于活动状态, 它所对应的 ThreadLocal 实例就是可访问的, 线程被终止后, 它的所有实例将被垃圾收集. 总之记住一句话:**ThreadLocal 存储的变量属于当前线程**. 

## 整体认识

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_1.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_1.png" style="zoom:67%;" />

ThreadLocal 中的嵌套内部类 ThreadLocalMap, 这个类本质上是一个 map, 和 HashMap 之类的实现相似, 依然是 key-value 的形式, 其中有一个内部类 Entry, 其中 key 可以看做是 ThreadLocal 实例, **但是其本质是持有 ThreadLocal 实例的弱引用**

在 ThreadLocal 中并没有对于 ThreadLocalMap 的引用, ThreadLocalMap 的引用在 Thread 类中, 代码如下. 每个线程在向 ThreadLocal 里塞值的时候, 其实都是向自己所持有的 ThreadLocalMap 里塞入数据；读的时候同理, 首先从自己线程中取出自己持有的 ThreadLocalMap, 然后再根据 ThreadLocal 引用作为 key 取出 value, 基于以上描述, ThreadLocal 实现了变量的线程隔离 (当然, 毕竟变量其实都是从自己当前线程实例中取出来的) . 

```java
public class Thread implements Runnable {
    ......
    
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
    ......
}
```

## ThreadLocalMap 源码分析

下面一步一步介绍 ThreadLocalMap 源码分析的相关源码, 在分析 ThreadLocalMap 的同时, 也会介绍与 ThreadLocalMap 关联的ThreadLocal 中的方法 (这样分析完 ThreadLocalMap, ThreadLocal 基本就搞定了) , 可能有些需要前后结合才能真正理解. 

### 成员变量

```java
/**
* 初始容量 -- 必须是 2 的冥
*/
private static final int INITIAL_CAPACITY = 16;

/**
* 存放数据的 table, Entry 类的定义在下面分析
* 同样, 数组长度必须是2的冥. 
*/
private Entry[] table;

/**
* 数组里面 entrys 的个数, 可以用于判断 table 当前使用量是否超过负因子. 
*/
private int size = 0;

/**
* 进行扩容的阈值, 表使用量大于它的时候进行扩容. 
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
 * Entry 继承 WeakReference, 并且用 ThreadLocal 作为 key.如果 key 为 null
 * (entry.get() == null)表示 key 不再被引用, 表示 ThreadLocal 对象被回收
 * 因此这时候 entry 也可以从 table 从清除. 
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

Entry 继承 WeakReference, 使用弱引用, 可以将 ThreadLocal 对象的生命周期和线程生命周期解绑, 持有对 ThreadLocal 的弱引用, 可以使得 ThreadLocal 在没有其他强引用的时候被回收掉, 这样可以避免因为线程得不到销毁导致 ThreadLocal 对象无法被回收. 

### ThreadLocal 中的 set()

先看看 ThreadLocal 中 set() 源码. 

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

*   代码很简单, 获取当前线程, 并获取当前线程的 ThreadLocalMap 实例 (从 getMap(Thread t) 中很容易看出来) . 

*   如果获取到的 map 实例不为空, 调用 map.set() 方法, 否则调用构造函数 ThreadLocal.ThreadLocalMap(this, firstValue) 实例化 map. 

可以看出来线程中的 ThreadLocalMap 使用的是延迟初始化, 在第一次调用 get() 或者 set() 方法的时候才会进行初始化. 下面来看看构造函数 `ThreadLocalMap(ThreadLocal firstKey, Object firstValue)` . 

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

主要说一下计算索引, `firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1)`. 

*   关于 `& (INITIAL_CAPACITY - 1)`, 这是取模的一种方式, 对于 2 的幂作为模数取模, 用此代替 `%(2^n)`, 这也就是为啥容量必须为2的幂, 在这个地方也得到了解答, 至于为什么可以这样这里不过多解释, 原理很简单. 

*   关于 `firstKey.threadLocalHashCode`

```java
private final int threadLocalHashCode = nextHashCode();

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
private static AtomicInteger nextHashCode = new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;
```

定义了一个 AtomicInteger 类型, 每次获取当前值并加上 HASH_INCREMENT, `HASH_INCREMENT = 0x61c88647`, 关于这个值和`斐波那契散列`有关, 其原理这里不再深究, 感兴趣可自行搜索, 其主要目的就是为了让哈希码能均匀的分布在 2 的 n 次方的数组里, 也就是 `Entry[] table` 中. 

在了解了上面的源码后, 终于能进入正题了, 下面开始进入 ThreadLocalMap 中的 set(). 

### ThreadLocalMap 中的哈希冲突

ThreadLocalMap 使用 `线性探测法` 来解决哈希冲突, 线性探测法的地址增量 di = 1, 2, ... , m-1, 其中, i 为探测次数. 该方法一次探测下一个地址, 直到有空的地址后插入, 若整个空间都找不到空余的地址, 则产生溢出. 假设当前 table 长度为 16, 也就是说如果计算出来 key 的 hash 值为 14, 如果 table[14] 上已经有值, 并且其 key 与当前 key 不一致, 那么就发生了 hash 冲突, 这个时候将 14 加 1 得到 15, 取 table[15] 进行判断, 这个时候如果还是冲突会回到 0, 取 table[0], 以此类推, 直到可以插入. 

按照上面的描述, `可以把 table 看成一个环形数组`. 

先看一下线性探测相关的代码, 从中也可以看出来 table 实际是一个环: 

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

### ThreadLocalMap 的 set() 及其 set() 相关代码如下

```java
private void set(ThreadLocal<?> key, Object value) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    // 计算索引, 上面已经有说过. 
    int i = key.threadLocalHashCode & (len - 1);

    /**
     * 根据获取到的索引进行循环, 如果当前索引上的 table[i] 不为空, 在没有 return 的情况下, 
     * 就使用 nextIndex() 获取下一个 (上面提到到线性探测法) . 
     */
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // table[i] 上 key 不为空, 并且和当前 key 相同, 更新 value
        if (k == key) {
            e.value = value;
            return;
        }
        /**
         * table[i] 上的 key 为空, 说明被回收了 (上面的弱引用中提到过) 
         * 这个时候说明改 table[i] 可以重新使用, 用新的 key-value 将其替换, 并删除其他无效的 entry
         */
        if (k == null) {
            this.replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 找到为空的插入位置, 插入值, 在为空的位置插入需要对 size 进行加 1 操作
    tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
    int sz = ++size;

    /**
     * cleanSomeSlots 用于清除那些 e.get() == null, 也就是 table[index] != null && table[index].get() == null
     * 之前提到过, 这种数据 key 关联的对象已经被回收, 所以这个 Entry(table[index]) 可以被置 null. 
     * 如果没有清除任何 entry, 并且当前使用量达到了负载因子所定义(长度的2/3), 那么进行rehash()
     */
    if (!this.cleanSomeSlots(i, sz) && sz >= threshold) {
        this.rehash();
    }
}


/**
 * 替换无效 entry
 */
private void replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    ThreadLocal.ThreadLocalMap.Entry e;

    /**
     * 根据传入的无效 entry 的位置 (staleSlot) ,向前扫描
     * 一段连续的 entry(这里的连续是指一段相邻的 entry 并且 table[i] != null),
     * 直到找到一个无效 entry, 或者扫描完也没找到
     */
    int slotToExpunge = staleSlot; // 之后用于清理的起点
    for (int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len)) {
        if (e.get() == null) {
            slotToExpunge = i;
        }
    }

    /**
     * 向后扫描一段连续的 entry
     */
    for (int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();

        /**
         * 如果找到了 key, 将其与传入的无效 entry 替换, 也就是与 table[staleSlot] 进行替换
         */
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            // 如果向前查找没有找到无效 entry, 则更新 slotToExpunge 为当前值 i
            if (slotToExpunge == staleSlot) {
                slotToExpunge = i;
            }
            this.cleanSomeSlots(this.expungeStaleEntry(slotToExpunge), len);
            return;
        }

        /**
         * 如果向前查找没有找到无效 entry, 并且当前向后扫描的 entry 无效, 则更新 slotToExpunge 为当前值 i
         */
        if (k == null && slotToExpunge == staleSlot) {
            slotToExpunge = i;
        }
    }

    /**
     * 如果没有找到 key, 也就是说 key 之前不存在 table 中
     * 就直接最开始的无效 entry--tab[staleSlot] 上直接新增即可
     */
    tab[staleSlot].value = null;
    tab[staleSlot] = new ThreadLocal.ThreadLocalMap.Entry(key, value);

    /**
     * slotToExpunge != staleSlot,说明存在其他的无效 entry 需要进行清理. 
     */
    if (slotToExpunge != staleSlot) {
        this.cleanSomeSlots(this.expungeStaleEntry(slotToExpunge), len);
    }
}

/**
 * 连续段清除
 * 根据传入的 staleSlot, 清理对应的无效 entry——table[staleSlot],
 * 并且根据当前传入的 staleSlot, 向后扫描一段连续的 entry(这里的连续是指一段相邻的 entry 并且 table[i] != null),
 * 对可能存在 hash 冲突的 entry 进行 rehash, 并且清理遇到的无效 entry.
 *
 * @param staleSlot key 为 null,需要无效 entry 所在的 table 中的索引
 * @return 返回下一个为空的solt的索引. 
 */
private int expungeStaleEntry(int staleSlot) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;

    // 清理无效 entry, 置空
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    // size 减 1, 置空后 table 的被使用量减 1
    size--;

    ThreadLocal.ThreadLocalMap.Entry e;
    int i;
    /**
     * 从 staleSlot 开始向后扫描一段连续的 entry
     */
    for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        // 如果遇到 key 为 null, 表示无效entry, 进行清理.
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            // 如果 key 不为 null, 计算索引
            int h = k.threadLocalHashCode & (len - 1);
            /**
             * 计算出来的索引——h, 与其现在所在位置的索引——i 不一致, 置空当前的 table[i]
             * 从 h 开始向后线性探测到第一个空的 slot, 把当前的 entry 挪过去. 
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
    // 下一个为空的solt的索引. 
    return i;
}

/**
 * 启发式的扫描清除, 扫描次数由传入的参数n决定
 *
 * @param i 从 i 向后开始扫描 (不包括i, 因为索引为i的Slot肯定为null) 
 *
 * @param n 控制扫描次数, 正常情况下为 log2(n) , 
 * 如果找到了无效 entry, 会将 n 重置为 table 的长度 len, 进行段清除. 
 *
 * map.set() 点用的时候传入的是元素个数, replaceStaleEntry() 调用的时候传入的是 table 的长度 len
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
            // 重置 n 为 len
            n = len;
            removed = true;
            // 依然调用 expungeStaleEntry 来进行无效 entry 的清除
            i = this.expungeStaleEntry(i);
        }
    } while ((n >>>= 1) != 0); // 无符号的右移动, 可以用于控制扫描次数在 log2(n)
    return removed;
}


private void rehash() {
    // 全清理
    this.expungeStaleEntries();

    /**
     * threshold = 2/3 * len
     * 所以threshold - threshold / 4 = 1en/2
     * 这里主要是因为上面做了一次全清理所以size减小, 需要进行判断. 
     * 判断的时候把阈值调低了. 
     */
    if (size >= threshold - threshold / 4) {
        this.resize();
    }
}

/**
 * 扩容, 扩大为原来的2倍 (这样保证了长度为2的冥) 
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
            // 虽然做过一次清理, 但在扩容的时候可能会又存在 key == null 的情况. 
            if (k == null) {
                // 这里试试将e.value设置为null
                e.value = null; // Help the GC
            } else {
                // 同样适用线性探测来设置值. 
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null) {
                    h = nextIndex(h, newLen);
                }
                newTab[h] = e;
                count++;
            }
        }
    }

    // 设置新的阈值
    setThreshold(newLen);
    size = count;
    table = newTab;
}

/**
 * 全清理, 清理所有无效 entry
 */
private void expungeStaleEntries() {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        ThreadLocal.ThreadLocalMap.Entry e = tab[j];
        if (e != null && e.get() == null)
        // 使用连续段清理
        {
            this.expungeStaleEntry(j);
        }
    }
}
```

### ThreadLocal 中的 get()

```java
public T get() {
    // 同 set 方法类似获取对应线程中的 ThreadLocalMap 实例
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
 * 初始化设值的方法, 可以被子类覆盖. 
 */
protected T initialValue() {
    return null;
}

private T setInitialValue() {
    // 获取初始化值, 默认为 null(如果没有子类进行覆盖)
    T value = this.initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    // 不为空不用再初始化, 直接调用set操作设值
    if (map != null) {
        map.set(this, value);
    } else {
        // 第一次初始化, createMap在上面介绍 set() 的时候有介绍过. 
        createMap(t, value);
    }
    return value;
}
```

### ThreadLocalMap 中的 getEntry()

```java
private ThreadLocal.ThreadLocalMap.Entry getEntry(ThreadLocal<?> key) {
    // 根据 key 计算索引, 获取 entry
    int i = key.threadLocalHashCode & (table.length - 1);
    ThreadLocal.ThreadLocalMap.Entry e = table[i];
    if (e != null && e.get() == key) {
        return e;
    } else {
        return this.getEntryAfterMiss(key, i, e);
    }
}

/**
 * 通过直接计算出来的 key 找不到对于的 value 的时候适用这个方法.
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
            // 清除无效的 entry
            expungeStaleEntry(i);
        } else {
            // 基于线性探测法向后扫描
            i = nextIndex(i, len);
        }
        e = tab[i];
    }
    return null;
}
```

### ThreadLocalMap 中的 remove()

```java
private void remove(ThreadLocal<?> key) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    // 计算索引
    int i = key.threadLocalHashCode & (len - 1);
    // 进行线性探测, 查找正确的 key
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            // 调用 weakrefrence 的 clear() 清除引用
            e.clear();
            // 连续段清除
            expungeStaleEntry(i);
            return;
        }
    }
}
```

remove() 在有上面了解后可以说极为简单了, 就是找到对应的 table[], 调用 weakrefrence 的 clear() 清除引用, 然后再调用expungeStaleEntry() 进行清除. 

## ThreadLocal 对象的回收

Entry 是一个弱引用, 是因为它不会影响到 ThreadLocal 的 GC 行为, 如果是强引用的话, 在线程运行过程中, 我们不再使用ThreadLocal 了, 将 ThreadLocal 置为 null, 但 ThreadLocal 在线程的 ThreadLocalMap 里还有引用, 导致其无法被 GC 回收. 而Entry 声明为 WeakReference, ThreadLocal 置为 null 后, 线程的 ThreadLocalMap 就不算强引用了, ThreadLocal 就可以被 GC 回收了. 



## ThreadLocal 六连问

### 内存泄漏原因探索

ThreadLocalMap 的生命周期是与线程一样的, 但是 ThreadLocal 却不一定, 可能 ThreadLocal 使用完了就想要被回收, 但是此时线程可能不会立即终止, 还会继续运行 (比如线程池中线程重复利用) , 如果 ThreadLocal 对象只存在弱引用, 那么在下一次垃圾回收的时候必然会被清理掉. 

如果 ThreadLocal 没有被外部强引用的情况下, 在垃圾回收的时候会被清理掉的, 这样一来 ThreadLocalMap 中使用这个ThreadLocal 的 key 也会被清理掉. 但是, value 是强引用, 不会被清理, 这样一来就会出现 key 为 null 的 value

key 为空的话 value 是无效数据, 久而久之, value 累加就会导致内存泄漏. 

### 怎么解决这个内存泄漏问题

在 ThreadLocalMap 中, 调用 set(), get(), remove() 方法的时候, 会清理掉 key 为 null 的记录. 在 ThreadLocal 设置为 null 之后, ThreadLocalMap 中存在 key 为 null 的值, 那么就可能发生内存泄漏, 只有手动调用 `remove()` 方法来避免, **所以我们在使用完ThreadLocal 变量时, 尽量用 threadLocal.remove() 来清除, 避免 threadLocal = null 的操作**. remove 方法是彻底地回收该对象, 而通过 `threadLoca l= null` 只是释放掉了 ThreadLocal 的引用, 但是在 ThreadLocalMap 中却还存在其 Entry, 后续还需处理. 

### JDK开发者是如何避免内存泄漏的

ThreadLocal 的设计者也意识到了这一点(内存泄漏), 他们在一些方法中埋了对 key = null 的 value 擦除操作. 

这里拿 ThreadLocal 提供的 get() 方法举例, 它调用了 ThreadLocalMap#getEntry() 方法, 对 key 进行了校验和对 null key 进行擦除. 

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

如果 key 为 null, 则会调用 getEntryAfterMiss() 方法, 在这个方法中, 如果 k == null , 则调用 expungeStaleEntry(i) 方法. 

expungeStaleEntry(i) 方法完成了对 key=null 的 key 所对应的 value 进行赋空, 释放了空间避免内存泄漏. 

同时它遍历下一个 key 为空的 entry, 并将 value 赋值为 null, 等待下次 GC 释放掉其空间. 

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

同理, set()方法最终也是调用该方法(expungeStaleEntry), 调用路径: 

```java
set 方法
set(T value) -> map.set(this, value) -> rehash() -> expungeStaleEntries()
remove方法
remove() -> ThreadLocalMap.remove(this) -> expungeStaleentries()
```

这样做, 也只能说尽可能避免内存泄漏, 但并不会完全解决内存泄漏这个问题. 比如极端情况下我们只创建 ThreadLocal 但不调用set, get, remove 方法等. 所以最能解决问题的办法就是用完 ThreadLocal 后手动调用 remove().



### 手动释放 ThreadLocal 遗留存储?你怎么去设计/实现? 

这里主要是强化一下手动 remove 的思想和必要性, 设计思想与连接池类似. 

包装其父类 remove 方法为静态方法, 如果是 spring 项目, 可以借助于 bean 的声明周期, 在拦截器的 afterCompletion 阶段进行调用. 

**弱引用导致内存泄漏, 那为什么 key 不设置为强引用**

这个问题就比较有深度了, 是你谈薪的小小资本. 

如果 key 设置为强引用, 当 threadLocal 实例释放后, threadLocal = null, 但是 threadLocal 会有强引用指向 threadLocalMap, threadLocalMap.Entry 又强引用 threadLocal, 这样会导致 threadLocal 不能正常被GC回收. 

弱引用虽然会引起内存泄漏, 但是也有 set, get, remove 方法操作对 null key 进行擦除的补救措施, 方案上略胜一筹. 

线程执行结束后会不会自动清空 Entry 的 value

一并考察了你的 gc 基础. 

事实上, 当 currentThread 执行结束后, threadLocalMap 变得不可达从而被回收, Entry 等也就都被回收了, 但这个环境就要求不对 Thread 进行复用, 但是我们项目中经常会复用线程来提高性能, 所以 currentThread 一般不会处于终止状态. 

在 web 项目中, 我们使用 tomcat 容器, 一般会开启一个线程池, 线程的释放会归还到线程池中并不会真正释放, 如果在 web 项目中调用 ThreadLocal 的 set 方法 将一个对象放入 Thread 中的成员变量 threadLocals(ThreadLocalMap) 中, 那么这个对象(ThreadLocal)是永远不会被回收的, 因为这个对象永远都被Thread 中的成员变量 threadLocals 引用着. 

如果想让垃圾收集器回收它, 有两种方法

1.  将该线程从 tomcat 线程池中去除, 当一个线程被回收的时候何况它的成员变量呢, 但是 tomcat 启动一般都会配置一个线程池进行优化, 所有该方法不太现实. 

2.  调用 ThreadLocal 的 remove 方法 将对象从 Thread 中的成员变量 threadLocals 中删除掉. 



### **Thread 和 ThreadLocal 有什么联系呢**

Thread 和 ThreadLocal 是绑定的, ThreadLocal 依赖于 Thread 去执行, Thread 将需要隔离的数据存放到 ThreadLocal (准确的讲是 ThreadLocalMap)中, 来实现多线程处理. 



### Spring 如何处理 Bean 多线程下的并发问题

ThreadLocal 天生为解决相同变量的访问冲突问题, 所以这个对于 spring 的默认单例 bean 的多线程访问是一个完美的解决方案. spring 也确实是用了 ThreadLocal 来处理多线程下相同变量并发的线程安全问题. 

spring 如何保证数据库事务在同一个连接下执行的? 

要想实现 jdbc 事务, 就必须是在同一个连接对象中操作, 多个连接下事务就会不可控, 需要借助分布式事务完成. 那 spring 如何保证数据库事务在同一个连接下执行的呢? 

DataSourceTransactionManager 是 spring 的数据源事务管理器, 它会在你调用 getConnection() 的时候从数据库连接池中获取一个 connection, 然后将其与 ThreadLocal 绑定, 事务完成后解除绑定. 这样就保证了事务在同一连接下完成. 



## ThreadLocal 应用场景

处理较为复杂的业务时, 使用 ThreadLocal 代替参数的显示传递. 

ThreadLocal 可以用来做数据库连接池保存 Connection 对象, 这样就可以让线程内多次获取到的连接是同一个了 (Spring 中的 DataSource 就是使用的 ThreadLocal) . 

管理 Session 会话, 将 Session 保存在 ThreadLocal 中, 使线程处理多次处理会话时始终是一个 Session. 



## Threadlocal 跨线程传递解决方案

**ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用, 也即变量在线程间隔离而在方法或类间共享的场景. 简单易懂的就是. 每个线程有自己的数据副本, 当线程结束后可以独立回收. **

**通常使用场景**

**比如 spring mvc 收到请求. 拦截器将用户 id 等数据放到 threadlocal 当中. 在当前线程就可以直接拿到数据**

**具体例子:**

**有些同学会说出开源项目中有些地方会用到:**

**比如这种, 比如 mysql 读写分离. 主写读从过程中, 类似于 shardingsphere 的中间件. 就通过 threadlocal 来实现某一次读请求路由到主库. 通过 threadlocal 设置标识即可**

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_2.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_2.png" style="zoom:67%;" />

如果我启动另外一个线程. 那么在主线程设置的 threadlocal 值能被子线程拿到吗?

如果拿不到, 怎么去解决这个问题. 有的同学一定会想这种场景比较少. 顶多就是面试官找茬罢了

其实不然. 这种场景非常普遍. 举一个例子: 

现在微服务基本上属于各个大厂小厂的标配. 有一个问题. 全链路如何来做监控. 如果用过 spring cloud 的同学一定会说. zipkin, 有的可能也会说 cat, pinpoint, skywalking 等等, 

暂且不讨论这些全链路组件的优劣. 在全链路组件落地的过程中, threadlocal 是一个相当关键的步骤. 拿 zipkin 的实现来说: 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_3.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_3.png" style="zoom:67%;" />

核心的数据传递都通过 threadlocal 来实现

那么回到我刚才的问题. threadlocal 跨线程是否能够传递呢? 那么我们做一个实验: 

相关代码. 我们在父线程设置了一个threadlocal. 另外启动一个线程去获取

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_4.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_4.png" style="zoom:67%;" />

运行结果:

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_5.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_5.png" style="zoom:67%;" />

结论: 拿不到数据. 获取的是个 null

那我们应该怎么去解决这个问题: 

在 java8 中提供了一个这个类(InheritableThreadLocal): 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_6.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_6.png" style="zoom:67%;" />

从字面意思来看: 可以实现父线程到子线程的共享, 那么我们实验一下

将实现换成了 InheritableThreadLocal

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_7.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_7.png" style="zoom:67%;" />

看一下效果 (果然解决了问题): 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_8.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_8.png" style="zoom:67%;" />

### InheritableThreadLocal 原理

看一下目录结构: 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_9.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_9.png" style="zoom:67%;" />

复写了父类 3 个方法

当主线程中对该变量进行 set 操作的时候, 和 ThreadLocal 一样会初始化一个 ThreadLocalMap 对实际的变量值进行存储, 以 ThreadLocal 为 key, 值为 value, 如果有多个 ThreadLocal 变量也都是存储在这个 Map 中. 该 Map 使用的是 HashMap 的原理进行数据的存储, 但是和 ThreadLocal 有一点差别, 因为其覆写了 createMap 的方法. 

在 Thread 类当中, 可以看出 Thread 类维护了两个成员变量, ThreadLocal 以及 InheritableThreadLocal, 数据类型都是 ThreadLocalMap. 这也就解释了为什么这个变量是线程私有的. 但是如果要知道为什么父子线程的变量传递, 那就继续看一下源码. 当我们在主线程中开一个新的子线程的时候, 开始会 new 一个新的 Thread

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_10.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_10.png" style="zoom:67%;" />

在 thread 构造函数中, new 一个 thread 方法

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_11.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_11.png" style="zoom:67%;" />

可以看到, 最后会调用 ThreadLocal 的 createInheritedMap 方法, 而该方法会新建一个 ThreadLocalMap, 看一下构造函数的内容: 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_12.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_12.png" style="zoom:67%;" />

parentMap 就是父线程的 ThreadLocalMap, 这个构造函数的意思大概就是将父线程的 ThreadLocalMap 复制到自己的 ThreadLocalMap 里面来, 这样我们就可以使用 InheritableThreadLocal 访问到父线程中的变量了

但是这个就能解决问题了吗. 并不是~

下个小结我会继续讲解 InheritableThreadLocal 在多线程调用过程中还存在哪些坑. 如何解决



### InheritableThreadLocal 的局限性

**看一个例子**

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_13.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_13.png" style="zoom:67%;" />

new 了一个线程池大小为1的线程池. 在第一次执行前, 设置了一个 wzAAA, 在子线程执行获取, 然后 sleep(2000) 以后, 设置了一个另外的值. 再次获取结果

看一下运行结果:

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_14.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_14.png" style="zoom:67%;" />

两次执行结果居然一样. . 那么显然不是我们想要的结果. 问题出在哪里? 

原因是我们使用了固定为 1 的线程池, 线程池中是缓存使用过的线程, 当线程被重复调用的时候并没有再重新初始化 init() 线程, 而是直接使用已经创建过的线程, 所以这里的值并不会被再次操作. 

那么如何解决这个问题: 

阿里巴巴有个开源项目: 

[https://github.com/alibaba/transmittable-thread-local](https://github.com/alibaba/transmittable-thread-local)

让我们看一下简介: 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_15.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_15.png" style="zoom:67%;" />

我们按照相关 wiki 进行改造

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_16.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_16.png" style="zoom:67%;" />

相关结果正确: 

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_17.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_17.png" style="zoom:67%;" />

相关原理简介: 

[https://github.com/alibaba/transmittable-thread-local/blob/master/docs/developer-guide.md](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Ftransmittable-thread-local%2Fblob%2Fmaster%2Fdocs%2Fdeveloper-guide.md)

其实在很多开源项目中, 尤其是全链路传递实现, 也有类似的实现: 

比如 sleuth:

<img src="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_18.png" alt="https://www.miaomiaoqi.github.io/images/java/threadlocal/th_18.png" style="zoom:67%;" />

在跨线程调用中. 也是通过覆写 runnable 来实现来防止链路丢失



## 参考

[https://www.jianshu.com/p/7a7a4b05a03c](https://www.jianshu.com/p/7a7a4b05a03c)

[https://www.jianshu.com/p/25857f3bf960](https://www.jianshu.com/p/25857f3bf960)