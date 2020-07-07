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

* HashSet：基于HashMap实现的，

### Collection子接口之List

#### ArrayList和Vector的区别

1. Vector的方法都是同步的（Synchronized），是线程安全的，而ArrayList不是线程安全的。由于线程的同步会影响性能，因此，ArrayList的性能比Vector好
2. ArrayList按1.5倍扩容，Vector按两倍扩容

#### ArrayList和LinkedList的区别

1. 是否线程安全

   都是不同步的，都不是线程安全的

2. 底层数据结构

   ArrayList：Object[]数组；LinkedList：双向链表

3. 插入和删除是否受元素位置的影响

4. 是否支持快速随机访问

5. 内存空间占用

