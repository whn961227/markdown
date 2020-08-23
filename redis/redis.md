## Redis

### 简介

Redis 是一个高性能的 key - value 数据库

Redis 与 其他 key - value 缓存产品有以下三个特点：

* Redis 支持数据持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
* Redis 不仅仅支持简单的 key - value 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储
* Redis 支持数据的备份，即 master - slave 模式的数据备份

### 优势

* 性能极高 – Redis 读的速度是 110000 次 /s, 写的速度是 81000 次 /s 。
* 丰富的数据类型 - Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* 原子性 - Redis 的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过 MULTI 和 EXEC 指令包起来。
* 其他特性 - Redis 还支持 publish/subscribe 通知，key 过期等特性。

### 数据类型

Redis 支持 5 中数据类型：string（字符串），hash（哈希），list（列表），set（集合），zset（sorted set：有序集合）。

> **string**

string 是 redis 最基本的数据类型。**一个 key 对应一个 value**。

> **hash**

Redis 中的字典相当于 Java 中的 **HashMap**，内部实现也差不多类似，都是通过 **"数组 + 链表"** 的链地址法来解决部分 **哈希冲突**，同时这样的结构也吸收了两种不同数据结构的优点。hash 是一个键值对（key - value）集合。适合用于存储对象。

> **list**

Redis 的列表相当于 Java 语言中的 **LinkedList**，Redis 列表是简单的字符串列表，按照插入顺序排序。我们可以往列表的左边或者右边添加元素。**list 内的元素是可重复的。**

> **set**

Redis 的集合相当于 Java 语言中的 **HashSet**，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。

> **zset**

它类似于 Java 中 **SortedSet** 和 **HashMap** 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以为每个 value 赋予一个 score 值，用来代表排序的权重。

