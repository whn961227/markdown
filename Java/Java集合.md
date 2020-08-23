## 集合

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200707220958.png)

### 概述

---

#### List，Set，Map三者区别

List：有序可重复

Set：无序不可重复

Map：存储key-value键值对，key无序不可重复，value无序可重复

#### 集合底层数据结构总结

##### List

* ArrayList：Object[]数组
* Vector：Object[]数组
* LinkedList：双向链表

##### Set

* HashSet：基于HashMap实现的，底层采用HashMap保存元素

#### Iterator迭代器

##### 迭代器Iterator是什么

##### 迭代器Iterator有啥用

遍历集合，特点是更加安全，可以确保在当前遍历的集合元素被更改的时候，就会抛出ConcurrentModificationException异常



### Collection子接口之List

---

#### ArrayList和Vector的区别

1. Vector的方法都是同步的（Synchronized），是线程安全的，而ArrayList不是线程安全的。由于线程的同步会影响性能，因此，ArrayList的性能比Vector好
2. ArrayList按1.5倍扩容，Vector按2倍扩容

#### ArrayList和LinkedList的区别

1. **是否线程安全**

   都是不同步的，都不是线程安全的

2. **底层数据结构**

   ArrayList：Object[]数组；LinkedList：双向链表

3. **插入和删除是否受元素位置的影响**

4. **是否支持快速随机访问**

5. **内存空间占用**



### Collection子接口之Set

---



### Map接口

---

#### HashMap 和Hashtable 的区别

1. **是否线程安全**

   HashMap 不是线程安全的，Hashtable 是线程安全的，内部方法基本都经过 Synchronized 修饰

2. **效率**

   因为线程安全问题，HashMap 要比 Hashtable 效率高。Hashtable 基本被淘汰

3. **对 Null Key 和 Null Value 的支持**

   HashMap 可以存储 null 的 key 和 value，但 null 作为 key 只有一个，null 作为值可以有多个；Hashtable 不允许有 null 的 key 和 value，否则会抛出 NullPointerException

4. **初始容量大小和每次扩充容量大小的不同**

   创建时如果不指定初始容量，Hashtable 默认的初始大小为11，每次按 2n+1 倍扩容；HashMap 默认的初始大小为 16，每次按 2 倍扩容

   创建时如果指定初始容量，Hashtable 会直接创建给定的容量；HashMap 会将其扩充为 2 的幂次方大小，HashMap 总是使用 2 的幂作为 HashMap 的大小

5. **底层数据结构**

   HashMap：数组+链表+红黑树

#### HashMap 和 HashSet 区别

HashSet 底层是基于 HashMap 实现的

#### HashMap 和 TreeMap 区别

相比于 HashMap 来说 TreeMap 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力

#### HashMap 的底层实现

 数组+链表+红黑树

#### HashMap 的长度为什么是 2 的幂次方

数组下标的计算方法是（n-1）& hash（也就是说 hash % length == hash &（length-1）的前提是 length 是 2 的幂次方），采用二进制位操作 &，相当于 %，能够提高运算效率

#### HashMap 多线程操作导致死循环问题

jdk1.7 采用头插法，在 rehash 时会形成循环链表

jdk1.8 多线程会出现数据覆盖

> **HashMap在多线程环境下存在线程安全问题，怎么处理这种情况？**

* 使用Collections.synchronizedMap(Map)创建线程安全的map集合；
* Hashtable
* ConcurrentHashMap

不过出于线程并发度的原因，我都会舍弃前两者使用最后的ConcurrentHashMap，他的性能和效率明显高于前两者

> Collections.synchronizedMap是怎么实现线程安全的你有了解过么？

在**SynchronizedMap**内部维护了一个**普通对象Map**，还有**排斥锁mutex**，如图

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpyhVLAW08sszrgEKUamuEKRYfOvAN7JoEXvgu88icnh5fso9DWDTbJZW21GvVyZpDaCw24EuaaxWZw/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 50%;" />

```java
Collections.synchronizedMap(new HashMap<>(16));
```

我们在调用这个方法的时候就需要传入一个Map，可以看到有两个构造器，如果你传入了mutex参数，则将对象排斥锁赋值为传入的对象。

如果没有，则将对象排斥锁赋值为this，即调用synchronizedMap的对象，就是上面的Map。

创建出synchronizedMap之后，再操作map的时候，就会对方法上锁，如图全是🔐

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpyhVLAW08sszrgEKUamuEKRgG8CTU8Uj4k0djWqQiaiayXO7H3WTTUN0v0jegVsj8fxBcCcIl4XAmqg/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

> 回答得不错，能跟我聊一下Hashtable么？

跟HashMap相比Hashtable是线程安全的，适合在多线程的情况下使用，但是效率可不太乐观。

他在**对数据操作的时候都会上锁**，所以效率比较低下。

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpyhVLAW08sszrgEKUamuEKR5TBac6DgWoozh2ovicuHVgiaiaRWaulhvC5Tw2ssAbzXsG6Nk5vwYGqcQ/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

> 除了这个你还能说出一些Hashtable 跟HashMap不一样点么？

Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null。

> 为啥 Hashtable 是不允许 KEY 和 VALUE 为 null, 而 HashMap 则可以呢？

因为Hashtable在我们put 空值的时候会直接**抛空指针异常**，但是HashMap却做了特殊处理。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> 但是你还是没说为啥Hashtable 是不允许键或值为 null 的，HashMap 的键值则都可以为 null？

这是因为Hashtable使用的是**安全失败机制（fail-safe）**，这种机制会使你此次读到的数据不一定是最新的数据。

如果你使用null值，就会使得其无法判断对应的key是不存在还是为空，因为你无法再调用一次contain(key）来对key是否存在进行判断，ConcurrentHashMap同理。

> ConcurrentHashMap，的数据结构吧，以及为啥他并发度这么高？

HashMap 底层是基于 `数组 + 链表` 组成的，不过在 jdk1.7 和 1.8 中具体实现稍有不同。

我先说一下他在1.7中的数据结构吧：

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpyhVLAW08sszrgEKUamuEKR92tLGjq5XU8SCBVmGAgiaSp95mnibgngXjFjycLTSkDMpOEfKvZaFBzQ/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 80%;" />

如图所示，是由 Segment 数组、HashEntry 组成，和 HashMap 一样，仍然是**数组加链表**。

Segment 是 ConcurrentHashMap 的一个内部类，主要的组成如下：

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    private static final long serialVersionUID = 2249069246763182397L;

    // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
    transient volatile HashEntry<K,V>[] table;

    transient int count;
        // 记得快速失败（fail—fast）么？
    transient int modCount;
        // 大小
    transient int threshold;
        // 负载因子
    final float loadFactor;

}
```

HashEntry跟HashMap差不多的，但是不同点是，他使用volatile去修饰了他的数据Value还有下一个节点next。

> 那你能说说他并发度高的原因么？

原理上来说，ConcurrentHashMap 采用了**分段锁**技术，其中 Segment 继承于 ReentrantLock。

不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 **CurrencyLevel (Segment 数组数量)**的线程并发。

每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

就是说如果容量大小是16他的并发度就是16，可以同时允许16个线程操作16个Segment而且还是线程安全的。

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();//这就是为啥他不可以put null值的原因
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          
         (segments, (j << SSHIFT) + SBASE)) == null) 
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```

他先定位到Segment，然后再进行put操作。

我们看看他的put源代码，你就知道他是怎么做到线程安全的了，关键句子我注释了。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
          // 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry
            HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
            V oldValue;
            try {
                HashEntry<K,V>[] tab = table;
                int index = (tab.length - 1) & hash;
                HashEntry<K,V> first = entryAt(tab, index);
                for (HashEntry<K,V> e = first;;) {
                    if (e != null) {
                        K k;
 // 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        e = e.next;
                    }
                    else {
                 // 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        int c = count + 1;
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
               //释放锁
                unlock();
            }
            return oldValue;
        }
```

首先第一步的时候会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 `scanAndLockForPut()` 自旋获取锁。

1. 尝试自旋获取锁。
2. 如果重试的次数达到了 `MAX_SCAN_RETRIES` 则改为阻塞锁获取，保证能获取成功。

> 那他get的逻辑呢？

get 逻辑比较简单，只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。

由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。

ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**。

> 你有没有发现1.7虽然可以支持每个Segment并发访问，但是还是存在一些问题？

是的，因为基本上还是数组加链表的方式，我们去查询的时候，还得遍历链表，会导致效率很低，这个跟jdk1.7的HashMap是存在的一样问题，所以他在jdk1.8完全优化了。

> 那你再跟我聊聊jdk1.8他的数据结构是怎么样子的呢？

其中抛弃了原有的 Segment 分段锁，而采用了 `CAS + synchronized` 来保证并发安全性。

跟HashMap很像，也把之前的HashEntry改成了Node，但是作用不变，把值和next采用了volatile去修饰，保证了可见性，并且也引入了红黑树，在链表大于一定值的时候会转换（默认是8）。

> 同样的，你能跟我聊一下他值的存取操作么？以及是怎么保证线程安全的？

ConcurrentHashMap在进行put操作的还是比较复杂的，大致可以分为以下步骤：

1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
5. 如果都不满足，则利用 synchronized 锁写入数据。
6. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpyhVLAW08sszrgEKUamuEKRAQbAll8f70WVfhklkktibkNs4xtfCDP6gH4MvG6kAkaSgxEcGzJtu1g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> ConcurrentHashMap的get操作又是怎么样子的呢？

* 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
* 如果是红黑树那就按照树的方式获取值。
* 就不满足那就按照链表的方式遍历获取值。

### 其他重要问题

---

#### 快速失败（fail-fast）

Java集合的错误检测机制，在使用迭代器对集合进行遍历的时候，在多线程下操作非安全失败（safe-fail）的集合类可能会触发fast-fail机制，则**抛出ConcurrentModificationException异常**。另外，在单线程下，如果**在遍历过程中对集合对象的内容进行了修改也会触发fast-fail机制**

##### 原因

每当迭代器使用**hashNext()/next()遍历下一个元素之前**，会**判断modCount变量是否为exceptedModCount值**，是就返回遍历；否则抛出异常，终止遍历

如果**在集合的遍历期间对其进行修改**，就会**改变modCount**的值，进而导致**modCount!=exceptedModCount**，进而**抛出ConcurrentModificationException异常**

>注：通过Iterator的方法修改集合的话会修改exceptedModCount的值，所以不会抛出异常

#### 安全失败（fail-safe）

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是**先复制原有集合内容**，**在拷贝的集合上进行遍历**。所以，在遍历过程中对原集合修改并不能被迭代器检测到，不会抛出ConcurrentModificationException异常