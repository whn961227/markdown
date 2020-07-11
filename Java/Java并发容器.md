## 并发容器

### ConcurrentHashMap

---

无论读操作还是写操作都能保证很高的性能：在进行读操作时几乎不需要加锁，而进行写操作时通过锁分段技术只对操作的段加锁而不影响对其他段的访问

#### ConcurrentHashMap和Hashtable的区别

* **底层数据结构**

  jdk1.7的ConcurrentHashMap底层采用 **Segment数组+HaSshEntry数组+链表**

  jdk1.8采用的数据结构跟HashMap的结构一样，**Node数组+链表/红黑树**

* **实现线程安全的方式**

  jdk1.7，ConcurrentHashMap采用分段锁，对整个桶数组分段（segment），每把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率

  jdk1.8，摒弃了segment的概念，直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和CAS来操作

  Hashtable采用全表锁，使用synchronized来保证线程安全，只要有一个线程访问或操作该对象，其他线程只能阻塞，相当于将所有的操作串行化，效率低下

#### ConcurrentHashMap线程安全的具体实现方式/底层具体实现

##### JDK1.7

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200708150849.png" style="zoom: 25%;" />

将数据分为段存储，然后给每一段数据加一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问

ConcurrentHashMap由 **Segment数组+HashEntry数组+链表** 组成

Segment实现了ReetrantLock，所以Segment是一种可重入锁；HashEntry用于存储键值对数据

##### JDK1.8

![image-20200708151203675](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200708151203675.png)

ConcurrentHashMap取消了Segment分段锁，才用CAS和synchronized来保证并发安全。数据结构采用 **数组+链表/红黑树**，synchronized只会锁定当前链表或红黑树的首节点，只要hash不冲突，就不会产生并发，效率提升



### CopyOnWriteArrayList

---

读取完全不用加锁，写入也不会阻塞读取操作，只有写入和写入之间需要进行同步等待

#### 实现方式

所有可变操作（add，set等）都是通过创建底层数组的新副本来实现

#### 适用场景

在写操作的同时允许读操作，适用于读多写少的场景

##### 缺陷

* **内存占用**：在写操作时需要复制一个新数组，使得内存占用为原来的两倍
* **数据不一致**：读操作不能读取实时性数据，因为部分写操作的数据还未同步到读数组中

不适合内存敏感以及对实时性要求很高的场景



### ConcurrentLinkedQueue

---

非阻塞队列的典型，采用CAS操作实现

底层数据结构是链表

适用对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景



### BlockingQueue

---

