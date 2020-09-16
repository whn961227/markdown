## HBase

### 存储结构

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914202955.png)

1. Client

2. Zookeeper

   Hbase 通过 ZK 来做 master 的高可用、RegionServer 的监控、元数据的入口以及集群配置的维护等工作

3. Hmaster

4. Hregionserver

5. HDFS

6. Region

   Hbase 表的分片，HBase 表会**根据 RowKey 值被切分**成不同的 region 存储在 RegionServer 中，在一个RegionServer 中可以有多个不同的 region

7. Store

   HFile 存储在 Store 中，**一个 Store 对应 HBase 表中的一个列族**

8. MemStore

   内存存储，位于内存中，用来保存当前的数据操作，所以当数据保存在 WAL 中之后，RegsionServer 会在内存中存储键值对

9. HFile

   这是在磁盘上保存原始数据的实际的物理文件，是实际的存储文件。**StoreFile 是以 Hfile 的形式存储在 HDFS**的

10. HLog（*Write-Ahead Logs*）

    HBase 的修改记录，当对 HBase 读写数据的时候，数据不是直接写进磁盘，它会**在内存中保留一段时间**（时间以及数据量阈值可以设定）。但把数据保存在内存中可能有更高的概率引起数据丢失，为了解决这个问题，数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入内存中。所以在系统出现故障的时候，数据可以通过这个日志文件重建

### RowKey 设计

#### RowKey 作用

##### RowKey 在查询中的作用

HBase 中 RowKey 可以唯一标识一行记录，在 HBase 中检索数据有以下三种方式：

1. 通过 **get** 方式，指定 **RowKey** 获取唯一一条记录
2. 通过 **scan** 方式，设置 **startRow** 和 **stopRow** 参数进行范围匹配
3. **全表扫描**，即直接扫描整张表中所有行记录

当大量请求访问HBase集群的一个或少数几个节点，造成少数RegionServer的读写请求过多、负载过大，而其他RegionServer负载却很小，这样就造成**热点现象**。大量访问会使热点Region所在的主机负载过大，引起性能下降，甚至导致Region不可用。所以我们在向HBase中插入数据的时候，应**尽量均衡地把记录分散到不同的 Region**里去，平衡每个Region的压力。

在进行查询的时候，根据 RowKey 从前向后匹配，所以我们在设计 RowKey 的时候选择好字段之后，还应该结合我们的实际的高频的查询场景来组合选择的字段，**越高频的查询字段排列越靠左（*类似最左前缀匹配原则*）**。

##### RowKey 在 Region 中的作用

在 HBase 中，Region 相当于一个数据的分片，每个 Region 都有`StartRowKey`和`StopRowKey`，这是表示 Region 存储的 RowKey 的范围，HBase 表的数据时按照 RowKey 来分散到不同的 Region，要想将数据记录均衡的分散到不同的 Region 中去，因此需要 RowKey 满足这种散列的特点。此外，在数据读写过程中也是与 RowKey 密切相关，RowKey 在读写过程中的作用：

1. 读写数据时通过 RowKey 找到对应的 Region；
2. MemStore 中的数据是按照 RowKey 的字典序排序；
3. HFile 中的数据是按照 RowKey 的字典序排序。

#### Rowkey 设计原则

**长度原则**

RowKey 是一个二进制码流，可以是任意字符串，最大长度为 64kb，实际应用中一般为 10-100byte，以 byte[] 形式保存，一般设计成定长。建议越短越好，不要超过16个字节，原因如下：

1. 数据的持久化文件 HFile 中时按照 Key-Value 存储的，如果 RowKey 过长，例如超过 100byte，那么 1000w 行的记录，仅 RowKey 就需占用近1GB的空间。这样会极大**影响 HFile 的存储效率**

2. MemStore 会缓存部分数据到内存中，若 RowKey 字段过长，**内存的有效利用率就会降低**，就不能缓存更多的数据，从而降低检索效率

3. 目前操作系统都是 64 位系统，内存 8 字节对齐，控制在 16 字节，8字节的整数倍利用了操作系统的最佳特性

   *内存对齐在某些情况下可以减少读取内存的次数以及一些运算，性能更高；另外，由于内存对齐保证了读取b变量是单次操作，在多核环境下，原子性更容易保证。但是内存对齐提升性能的同时，也需要付出相应的代价。由于变量与变量之间增加了填充，并没有存储真实有效的数据，所以占用的内存会更大。这也是一个典型的空间换时间的应用场景。*

**唯一原则**

必须在设计上保证 RowKey 的唯一性。由于在 HBase 中数据存储是 Key-Value 形式，若向 HBase 中同一张表插入相同 RowKey 的数据，则原先存在的数据会被新的数据覆盖。

**排序原则**

HBase 的 RowKey 是按照 ASCII 有序排序的，因此我们在设计 RowKey 的时候要充分利用这点。

**散列原则**

设计的 RowKey 应均匀的分布在各个 HBase 节点上。

#### Rowkey 字段选择

RowKey 字段的选择，遵循的**最基本原则是唯一性**，RowKey 必须能够唯一的识别一行数据。无论应用的负载特点是什么样，RowKey 字段都应该**参考最高频的查询场景**。数据库通常都是以如何高效的读取和消费数据为目的，而不是数据存储本身。然后，结合具体的负载特点，再对选取的 RowKey 字段值进行改造，组合字段场景下需要重点考虑字段的顺序。

#### 避免数据热点的方法

在对HBase的读写过程中，如何避免热点现象呢？主要有以下几种方法：

##### Reversing

如果经初步设计出的 RowKey 在数据分布上不均匀，但 **RowKey 尾部的数据却呈现出了良好的随机性**，此时，可以考虑将 **RowKey 的信息翻转**，或者直接将尾部的 bytes 提前到 RowKey 的开头。**Reversing 可以有效的使 RowKey 随机分布，但是牺牲了 RowKey 的有序性**

##### Salting

Salting（加盐）的原理是在原 RowKey 的前面**添加固定长度的随机数**，也就是给 RowKey 分配一个随机前缀使它和之间的 RowKey 的开头不同。**随机数能保障数据在所有 Regions 间的负载均衡**。

##### Hashing

基于 RowKey 的完整或部分数据进行 Hash，而后将 Hashing 后的值完整替换或部分替换原 RowKey 的前缀部分。这里说的 hash 包含 MD5、sha1、sha256 或 sha512 等算法

