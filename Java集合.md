## 集合

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200707220958.png)

### List，Set，Map三者区别

List：有序可重复

Set：无序不可重复

Map：存储key-value键值对，key无序不可重复，value无序可重复

### 集合底层数据结构总结

#### List

* ArrayList：Object[]数组
* Vector：Object[]数组
* LinkedList：双向链表

#### Set

* HashSet：基于HashMap实现的，底层采用HashMap保存元素

### Collection子接口之List

#### ArrayList和Vector的区别

1. Vector的方法都是同步的（Synchronized），是线程安全的，而ArrayList不是线程安全的。由于线程的同步会影响性能，因此，ArrayList的性能比Vector好
2. ArrayList按1.5倍扩容，Vector按2倍扩容

#### ArrayList和LinkedList的区别

1. 是否线程安全

   都是不同步的，都不是线程安全的

2. 底层数据结构

   ArrayList：Object[]数组；LinkedList：双向链表

3. 插入和删除是否受元素位置的影响

4. 是否支持快速随机访问

5. 内存空间占用

### Collection子接口之Set

### Map接口

#### HashMap和Hashtable的区别

1. 是否线程安全

   HashMap不是线程安全的，Hashtable是线程安全的，内部方法基本都经过Synchronized修饰

2. 效率

   因为线程安全问题，HashMap要比Hashtable效率高。Hashtable基本被淘汰

3. 对Null Key和Null Value的支持

   HashMap可以存储null的key和value，但null作为key只有一个，null作为值可以有多个；Hashtable不允许有null的key和value，否则会抛出NullPointerException

4. 初始容量大小和每次扩充容量大小的不同

   创建时如果不指定初始容量，Hashtable默认的初始大小为11，每次按2n+1倍扩容；HashMap默认的初始大小为16，每次按2倍扩容

   创建时如果指定初始容量，Hashtable会直接创建给定的容量；HashMap会将其扩充为2的幂次方大小，HashMap总是使用2的幂作为HashMap的大小

5. 底层数据结构

   HashMap：数组+链表+红黑树