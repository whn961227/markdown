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



### 其他重要问题

---

#### 快速失败（fail-fast）

Java集合的错误检测机制，在使用迭代器对集合进行遍历的时候，在多线程下操作非安全失败（safe-fail）的集合类可能会触发fast-fail机制，则抛出ConcurrentModificationException异常。另外，在单线程下，如果在遍历过程中对集合对象的内容进行了修改也会触发fast-fail机制

##### 原因

每当迭代器使用hashNext()/next()遍历下一个元素之前，会判断modCount变量是否为exceptedModCount值，是就返回遍历；否则抛出异常，终止遍历

如果在集合的遍历期间对其进行修改，就会改变modCount的值，进而导致modCount!=exceptedModCount，进而抛出ConcurrentModificationException异常

>注：通过Iterator的方法修改集合的话会修改exceptedModCount的值，所以不会抛出异常

#### 安全失败（fail-safe）

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。所以，在遍历过程中对原集合修改并不能被迭代器检测到，不会抛出ConcurrentModificationException异常