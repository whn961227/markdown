## Redis

### 简介

Redis 是一个高性能**非关系型** *(NoSQL)* 的 **键值对数据库**。

与传统数据库不同的是 **Redis** 的数据是 **存在内存** 中的，所以 **读写速度** 非常 **快**，因此 Redis 被广泛应用于 **缓存** 方向，每秒可以处理超过 `10` 万次读写操作，是已知性能最快的 Key-Value 数据库。另外，Redis 也经常用来做 **分布式锁**。

除此之外，Redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。

### 优势

* **读写性能极高** – Redis 读的速度是 110000 次 /s, 写的速度是 81000 次 /s 。
* **支持数据持久化**：支持 AOF 和 RDB 两种持久化方式。
* **支持事务**：**Redis 的所有操作都是原子性的**，同时 **Redis 还支持对几个操作合并后的原子性执行**
* **丰富的数据类型** - Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
* **支持主从复制**，主机会自动将数据同步到从机，可以进行读写分离。

### 缺点

* 数据库 **容量受到物理内存的限制**，不能用作海量数据的高性能读写，因此 Redis 适合的场景主要局限在较小数据量的高性能操作和运算上。
* Redis **不具备自动容错和恢复功能**，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复。
* 主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了 **系统的可用性**。
* **Redis 较难支持在线扩容**，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

### 为什么要用缓存？为什么使用 Redis

> **提一下现在 Web 应用的现状**

在日常的 Web 应用对数据库的访问中，**读操作的次数远超写操作**，比例大概在 **1:9** 到 **3:7**，所以需要读的可能性是比写的可能大得多的。当我们使用 SQL 语句去数据库进行读写操作时，数据库就会 **去磁盘把对应的数据索引取回来**，这是一个相对较慢的过程。

> **使用 Redis or 使用缓存带来的优势**

如果我们把数据放在 Redis 中，也就是直接放在内存之中，让服务端直接去读取内存中的数据，那么这样 **速度** 明显就会快上不少 *(高性能)*，并且会 **极大减小数据库的压力** *(特别是在高并发情况下)*。

> **也要提一下使用缓存的考虑**

但是使用内存进行数据存储开销也是比较大的，**限于成本** 的原因，一般我们只是使用 Redis 存储一些 **常用和主要的数据**，比如用户登录的信息等。

一般而言在使用 Redis 进行存储的时候，我们需要从以下几个方面来考虑：

* **业务数据常用吗？命中率如何？** 如果命中率很低，就没有必要写入缓存；
* **该业务数据是读操作多，还是写操作多？** 如果写操作多，频繁需要写入数据库，也没有必要使用缓存；
* **业务数据大小如何？** 如果要存储几百兆字节的文件，会给缓存带来很大的压力，这样也没有必要；

### 数据类型

Redis 支持 5 中基础数据类型：string（字符串），hash（哈希），list（列表），set（集合），zset（sorted set：有序集合）。

> **String**

string 是 redis 最基本的数据类型。**一个 key 对应一个 value**。

> **Hash**

Redis 中的字典相当于 Java 中的 **HashMap**，内部实现也差不多类似，都是通过 **"数组 + 链表"** 的链地址法来解决部分 **哈希冲突**，同时这样的结构也吸收了两种不同数据结构的优点。hash 是一个键值对（key - value）集合。适合用于存储对象。

> **List**

Redis 的列表相当于 Java 语言中的 **LinkedList**，Redis 列表是简单的字符串列表，按照插入顺序排序。我们可以往列表的左边或者右边添加元素。**list 内的元素是可重复的。**

> **Set**

Redis 的集合相当于 Java 语言中的 **HashSet**，它内部的键值对是**无序**、**唯一**的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。

> **Zset**

它类似于 Java 中 **SortedSet** 和 **HashMap** 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以为每个 value 赋予一个 score 值，用来代表**排序的权重**。

它的内部实现就依赖了一种叫做 **「跳跃列表」** 的数据结构。

> **HyperLogLog**

用于计数的 HyperLogLog、用于支持存储地理位置信息的 Geo

#### 跳跃表

##### 跳跃表简介

> **为什么使用跳跃表**

因为 zset 要支持随机的插入和删除，所以它 **不宜使用数组来实现**，关于排序问题，我们也很容易就想到 **红黑树/ 平衡树** 这样的树形结构，为什么 Redis 不使用这样一些结构呢？

1. **性能考虑：**在高并发的情况下，树形结构需要执行一些类似于 rebalance 这样的可能涉及整棵树的操作，相对来说跳跃表的变化只涉及局部
2. **实现考虑：**在复杂度与红黑树相同的情况下，跳跃表实现起来更简单，看起来也更加直观；

> **本质是解决查找问题**

我们先来看一个普通的链表结构：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQJ2WzK4Dj4ibLKst3qkVLjMQcy5FkqLu7pmHhvoH15H5JUmIsBjicdhvw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

我们需要这个链表按照 score 值进行排序，这也就意味着，当我们需要添加新的元素时，我们需要定位到插入点，这样才可以继续保证链表是有序的，通常我们会使用 **二分查找法**，但二分查找是有序数组的，链表没办法进行位置定位，我们除了遍历整个找到第一个比给定数据大的节点为止 *（时间复杂度 O(n))* 似乎没有更好的办法。

但假如我们每相邻两个节点之间就增加一个指针，让指针指向下一个节点，如下图：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQZYuqmprbyribHqEdWvMKjdeiaaUR5swWG3iciaJ2ghlwp6ubdfxLahIQVg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

现在假设我们想要查找数据时，可以根据这条新的链表查找，如果碰到比待查找数据大的节点时，再回到原来的链表中进行查找，比如，我们想要查找 7，查找的路径则是沿着下图中标注出的红色指针所指向的方向进行的：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQn3GcsgglqK0DaME5KXiciaQLCkkSVKMia9gmv5icavhQhOwuHRavTLIMdg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

这是一个略微极端的例子，但我们仍然可以看到，通过新增加的指针查找，我们不再需要与链表上的每一个节点逐一进行比较，这样改进之后需要比较的节点数大概只有原来的一半。

利用同样的方式，我们可以在新产生的链表上，继续为每两个相邻的节点增加一个指针，从而产生第三层链表：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQZxrtjNvic9S9GPVcQiaWS4dhxvCJPxdHSxSCUdP81SU5o6JjS0E9sy5A/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

在这个新的三层链表结构中，我们试着 **查找 13**，那么沿着最上层链表首先比较的是 11，发现 11 比 13 小，于是我们就知道只需要到 11 后面继续查找，**从而一下子跳过了 11 前面的所有节点。**

可以想象，当链表足够长，这样的多层链表结构可以帮助我们跳过很多下层节点，从而加快查找的效率。

> **更进一步的跳跃表**

**跳跃表 skiplist** 就是受到这种多层链表结构的启发而设计出来的。按照上面生成链表的方式，上面每一层链表的节点个数，是下面一层的节点个数的一半，这样查找过程就非常类似于一个二分查找，使得查找的时间复杂度可以降低到 *O(logn)*。

但是，这种方法在插入数据的时候有很大的问题。新插入一个节点之后，就会打乱上下相邻两层链表上节点个数严格的 2:1 的对应关系。如果要维持这种对应关系，就必须把新插入的节点后面的所有节点 *（也包括新插入的节点）* 重新进行调整，这会让时间复杂度重新蜕化成 *O(n)*。删除数据也有同样的问题。

**skiplist** 为了避免这一问题，它不要求上下相邻两层链表之间的节点个数有严格的对应关系，而是 **为每个节点随机出一个层数(level)**。比如，一个节点随机出的层数是 3，那么就把它链入到第 1 层到第 3 层这三层链表中。为了表达清楚，下图展示了如何通过一步步的插入操作从而形成一个 skiplist 的过程：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQ5qUqf8c0vC3bfbc710Tz6iadcOlDYb39pApOUP9pCaUDQtuicUn9Jibvg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

从上面的创建和插入的过程中可以看出，每一个节点的层数（level）是随机出来的，而且新插入一个节点并不会影响到其他节点的层数，因此，**插入操作只需要修改节点前后的指针，而不需要对多个节点都进行调整**，这就降低了插入操作的复杂度。

现在我们假设从我们刚才创建的这个结构中查找 23 这个不存在的数，那么查找路径会如下图：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQclc3x7f7KuQtUMpfn1rP3I7mGVuOyydQjyhujAgTzo9z8XiacD0oJmA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

##### 跳跃表的实现

Redis 中的跳跃表由 `server.h/zskiplistNode` 和 `server.h/zskiplist` 两个结构定义，前者为**跳跃表节点**，后者则**保存了跳跃节点的相关信息**，同之前的 `集合 list` 结构类似，其实只有 `zskiplistNode` 就可以实现了，但是引入后者是为了更加方便的操作：

```cpp
/* ZSETs use a specialized version of Skiplists */
typedef struct zskiplistNode {
    // value
    sds ele;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned long span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    // 跳跃表头指针
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```

> **随机层数**

对于每一个新插入的节点，都需要调用一个随机算法给它分配一个合理的层数，源码在 `t_zset.c/zslRandomLevel(void)` 中被定义：

```java
int zslRandomLevel(void) {
    int level = 1;
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}
```

直观上期望的目标是 50% 的概率被分配到 `Level 1`，25% 的概率被分配到 `Level 2`，12.5% 的概率被分配到 `Level 3`，以此类推...有 2-63 的概率被分配到最顶层，因为这里每一层的晋升率都是 50%。

**Redis 跳跃表默认允许最大的层数是 32**，被源码中 `ZSKIPLIST_MAXLEVEL` 定义，当 `Level[0]` 有 264 个元素时，才能达到 32 层，所以定义 32 完全够用了。

> **创建跳跃表**

这个过程比较简单，在源码中的 `t_zset.c/zslCreate` 中被定义：

```cpp
zskiplist *zslCreate(void) {
    int j;
    zskiplist *zsl;

    // 申请内存空间
    zsl = zmalloc(sizeof(*zsl));
    // 初始化层数为 1
    zsl->level = 1;
    // 初始化长度为 0
    zsl->length = 0;
    // 创建一个层数为 32，分数为 0，没有 value 值的跳跃表头节点
    zsl->header = zslCreateNode(ZSKIPLIST_MAXLEVEL,0,NULL);
    
    // 跳跃表头节点初始化
    for (j = 0; j < ZSKIPLIST_MAXLEVEL; j++) {
        // 将跳跃表头节点的所有前进指针 forward 设置为 NULL
        zsl->header->level[j].forward = NULL;
        // 将跳跃表头节点的所有跨度 span 设置为 0
        zsl->header->level[j].span = 0;
    }
    // 跳跃表头节点的后退指针 backward 置为 NULL
    zsl->header->backward = NULL;
    // 表头指向跳跃表尾节点的指针置为 NULL
    zsl->tail = NULL;
    return zsl;
}
```

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQ5x6LbD2SKn18F5LcpW85gtiaEFd9Oic34lDMg9bicy5vmpajoZLU5wEpw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

> **插入节点的实现**

这几乎是最重要的一段代码了，但总体思路也比较清晰简单，如果理解了上面所说的跳跃表的原理，那么很容易理清楚插入节点时发生的几个动作 *（几乎跟链表类似）*：

1. 找到当前我需要插入的位置 *（其中包括相同 score 时的处理）*；
2. 创建新节点，调整前后的指针指向，完成插入；

为了方便阅读，我把源码 `t_zset.c/zslInsert` 定义的插入函数拆成了几个部分

> 第一部分：声明需要存储的变量

```cpp
// 存储搜索路径
zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
// 存储经过的节点跨度
unsigned int rank[ZSKIPLIST_MAXLEVEL];
int i, level;
```

> 第二部分：搜索当前节点插入位置

```cpp
serverAssert(!isnan(score));
x = zsl->header;
// 逐步降级寻找目标节点，得到 "搜索路径"
for (i = zsl->level-1; i >= 0; i--) {
    /* store rank that is crossed to reach the insert position */
    rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
    // 如果 score 相等，还需要比较 value 值
    while (x->level[i].forward &&
            (x->level[i].forward->score < score ||
                (x->level[i].forward->score == score &&
                sdscmp(x->level[i].forward->ele,ele) < 0)))
    {
        rank[i] += x->level[i].span;
        x = x->level[i].forward;
    }
    // 记录 "搜索路径"
    update[i] = x;
}
```



> 第三部分：生成插入节点

```cpp
/* we assume the element is not already inside, since we allow duplicated
 * scores, reinserting the same element should never happen since the
 * caller of zslInsert() should test in the hash table if the element is
 * already inside or not. */
level = zslRandomLevel();
// 如果随机生成的 level 超过了当前最大 level 需要更新跳跃表的信息
if (level > zsl->level) {
    for (i = zsl->level; i < level; i++) {
        rank[i] = 0;
        update[i] = zsl->header;
        update[i]->level[i].span = zsl->length;
    }
    zsl->level = level;
}
// 创建新节点
x = zslCreateNode(level,score,ele);
```

> 第四部分：重排前向指针

```cpp
for (i = 0; i < level; i++) {
    x->level[i].forward = update[i]->level[i].forward;
    update[i]->level[i].forward = x;

    /* update span covered by update[i] as x is inserted here */
    x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
    update[i]->level[i].span = (rank[0] - rank[i]) + 1;
}

/* increment span for untouched levels */
for (i = level; i < zsl->level; i++) {
    update[i]->level[i].span++;
}
```

> 第五部分：重排后向指针并返回

```cpp
x->backward = (update[0] == zsl->header) ? NULL : update[0];
if (x->level[0].forward)
    x->level[0].forward->backward = x;
else
    zsl->tail = x;
zsl->length++;
return x;
```

> **节点删除实现**

删除过程由源码中的 `t_zset.c/zslDeleteNode` 定义，和插入过程类似，都需要先把这个 **"搜索路径"** 找出来，然后对于每个层的相关节点重排一下前向后向指针，同时还要注意更新一下最高层数 `maxLevel`，直接放源码：

```cpp
/* Internal function used by zslDelete, zslDeleteByScore and zslDeleteByRank */
void zslDeleteNode(zskiplist *zsl, zskiplistNode *x, zskiplistNode **update) {
    int i;
    for (i = 0; i < zsl->level; i++) {
        if (update[i]->level[i].forward == x) {
            update[i]->level[i].span += x->level[i].span - 1;
            update[i]->level[i].forward = x->level[i].forward;
        } else {
            update[i]->level[i].span -= 1;
        }
    }
    if (x->level[0].forward) {
        x->level[0].forward->backward = x->backward;
    } else {
        zsl->tail = x->backward;
    }
    while(zsl->level > 1 && zsl->header->level[zsl->level-1].forward == NULL)
        zsl->level--;
    zsl->length--;
}

/* Delete an element with matching score/element from the skiplist.
 * The function returns 1 if the node was found and deleted, otherwise
 * 0 is returned.
 *
 * If 'node' is NULL the deleted node is freed by zslFreeNode(), otherwise
 * it is not freed (but just unlinked) and *node is set to the node pointer,
 * so that it is possible for the caller to reuse the node (including the
 * referenced SDS string at node->ele). */
int zslDelete(zskiplist *zsl, double score, sds ele, zskiplistNode **node) {
    zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
    int i;

    x = zsl->header;
    for (i = zsl->level-1; i >= 0; i--) {
        while (x->level[i].forward &&
                (x->level[i].forward->score < score ||
                    (x->level[i].forward->score == score &&
                     sdscmp(x->level[i].forward->ele,ele) < 0)))
        {
            x = x->level[i].forward;
        }
        update[i] = x;
    }
    /* We may have multiple elements with the same score, what we need
     * is to find the element with both the right score and object. */
    x = x->level[0].forward;
    if (x && score == x->score && sdscmp(x->ele,ele) == 0) {
        zslDeleteNode(zsl, x, update);
        if (!node)
            zslFreeNode(x);
        else
            *node = x;
        return 1;
    }
    return 0; /* not found */
}
```

> **节点更新实现**

当我们调用 `ZADD` 方法时，如果对应的 value 不存在，那就是插入过程，如果这个 value 已经存在，只是调整一下 score 的值，那就需要走一个更新流程。

假设这个新的 score 值并不会带来排序上的变化，那么就不需要调整位置，直接修改元素的 score 值就可以了，但是如果排序位置改变了，那就需要调整位置，该如何调整呢？

从源码 `t_zset.c/zsetAdd` 函数 `1350` 行左右可以看到，Redis 采用了一个非常简单的策略：

```cpp
/* Remove and re-insert when score changed. */
if (score != curscore) {
    zobj->ptr = zzlDelete(zobj->ptr,eptr);
    zobj->ptr = zzlInsert(zobj->ptr,ele,score);
    *flags |= ZADD_UPDATED;
}
```

**把这个元素删除再插入这个**，需要经过两次路径搜索，从这一点上来看，Redis 的 `ZADD` 代码似乎还有进一步优化的空间。

> **元素排名的实现**

跳跃表本身是有序的，Redis 在 skiplist 的 forward 指针上进行了优化，给每一个 forward 指针都增加了 `span` 属性，用来 **表示从前一个节点沿着当前层的 forward 指针跳到当前这个节点中间会跳过多少个节点**。在上面的源码中我们也可以看到 Redis 在插入、删除操作时都会小心翼翼地更新 `span` 值的大小。

```cpp
/* Find the rank for an element by both score and key.
 * Returns 0 when the element cannot be found, rank otherwise.
 * Note that the rank is 1-based due to the span of zsl->header to the
 * first element. */
unsigned long zslGetRank(zskiplist *zsl, double score, sds ele) {
    zskiplistNode *x;
    unsigned long rank = 0;
    int i;

    x = zsl->header;
    for (i = zsl->level-1; i >= 0; i--) {
        while (x->level[i].forward &&
            (x->level[i].forward->score < score ||
                (x->level[i].forward->score == score &&
                sdscmp(x->level[i].forward->ele,ele) <= 0))) {
            // span 累加
            rank += x->level[i].span;
            x = x->level[i].forward;
        }

        /* x might be equal to zsl->header, so test if obj is non-NULL */
        if (x->ele && sdscmp(x->ele,ele) == 0) {
            return rank;
        }
    }
    return 0;
}
```

### 持久化

#### 简介

**Redis** 的数据 **全部存储** 在 **内存** 中，如果 **突然宕机**，数据就会全部丢失，因此必须有一套机制来保证 Redis 的数据不会因为故障而丢失，这种机制就是 Redis 的 **持久化机制**，它会将内存中的数据库状态 **保存到磁盘** 中。

> **持久化发生了什么 | 从内存到磁盘**

我们来稍微考虑一下 **Redis** 作为一个 **"内存数据库"** 要做的关于持久化的事情。通常来说，从客户端发起请求开始，到服务器真实地写入磁盘，需要发生如下几件事情：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7PMcYtBZdH78LrPP2OrMV8iae8skGYoH6nlF88SxhhKEGxMt0TQKFFyL8X6epic3McpkxV8sibaj4zA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img"  />

**详细版** 的文字描述大概就是下面这样：

1. 客户端向数据库 **发送写命令** *(数据在客户端的内存中)*
2. 数据库 **接收** 到客户端的 **写请求** *(数据在服务器的内存中)*
3. 数据库 **调用系统 API** 将数据写入磁盘 *(数据在内核缓冲区中)*
4. 操作系统将 **写缓冲区** 传输到 **磁盘控制器** *(数据在磁盘缓存中)*
5. 操作系统的磁盘控制器将数据 **写入实际的物理媒介** 中 *(数据在磁盘中)*

**注意:** 上面的过程其实是 **极度精简** 的，在实际的操作系统中，**缓存** 和 **缓冲区** 会比这 **多得多**...

#### 两种持久化方式

##### 方式一：快照（RDB）

**Redis 快照** 是最简单的 Redis 持久性模式。当满足特定条件时，它将**生成数据集的时间点快照**，例如，如果先前的快照是在2分钟前创建的，并且现在已经至少有 *100* 次新写入，则将创建一个新的快照。此条件可以由用户配置 Redis 实例来控制，也可以在运行时修改而无需重新启动服务器。快照作为包含整个数据集的单个 `.rdb` 文件生成。

但我们知道，Redis 是一个 **单线程** 的程序，这意味着，我们不仅仅要响应用户的请求，还需要进行内存快照。而后者要求 Redis 必须进行 IO 操作，这会严重拖累服务器的性能。

还有一个重要的问题是，我们在 **持久化的同时**，**内存数据结构** 还可能在 **变化**，比如一个大型的 hash 字典正在持久化，结果一个请求过来把它删除了，可是这才刚持久化结束，咋办？

> **使用系统多进程 COW（Copy On Write）机制 | fork 函数**

操作系统多进程 **COW(Copy On Write) 机制** 拯救了我们。**Redis** 在持久化时会调用 `glibc` 的函数 `fork` 产生一个子进程，简单理解也就是基于当前进程 **复制** 了一个进程，主进程和子进程会共享内存里面的代码块和数据段：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7PMcYtBZdH78LrPP2OrMV8UK0nXA56QicaQ3gz516q8GiceGYZdDX6sbNOrPfk5J8TrBjMjW5RRSDg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img"  />

这里多说一点，**为什么 fork 成功调用后会有两个返回值呢？** 因为子进程在复制时复制了父进程的堆栈段，所以两个进程都停留在了 `fork` 函数中 *(都在同一个地方往下继续"同时"执行)*，等待返回，所以 **一次在父进程中返回子进程的 pid，另一次在子进程中返回零，系统资源不够时返回负数**。*(伪代码如下)*

```cpp
pid = os.fork()
if pid > 0:
  handle_client_request()  # 父进程继续处理客户端请求
if pid == 0:
  handle_snapshot_write()  # 子进程处理快照写磁盘
if pid < 0:
  # fork error
```

所以 **快照持久化** 可以完全交给 **子进程** 来处理，**父进程** 则继续 **处理客户端请求**。**子进程** 做数据持久化，它 **不会修改现有的内存数据结构**，它只是对数据结构进行遍历读取，然后序列化写到磁盘中。但是 **父进程** 不一样，它必须持续服务客户端请求，然后对 **内存数据结构进行不间断的修改**。

这个时候就会使用操作系统的 **COW 机制**来进行 **数据段页面** 的分离。数据段是由很多操作系统的页面组合而成，当父进程对其中一个页面的数据进行修改时，会将被共享的页面复制一份分离出来，然后 **对这个复制的页面进行修改**。这时 **子进程** 相应的页面是 **没有变化的**，还是进程产生时那一瞬间的数据。

子进程因为数据没有变化，它能看到的内存里的数据在进程产生的一瞬间就凝固了，再也不会改变，这也是为什么 **Redis** 的持久化 **叫「快照」的原因**。接下来子进程就可以非常安心的遍历数据了进行序列化写磁盘了。

> **优点**

他会生成多个数据文件，每个数据文件分别都代表了某一时刻**Redis**里面的数据，这种方式，有没有觉得很适合做**冷备**，完整的数据运维设置定时任务，定时同步到远端的服务器，比如阿里的云服务，这样一旦线上挂了，你想恢复多少分钟之前的数据，就去远端拷贝一份之前的数据就好了。

**RDB**对**Redis**的性能影响非常小，是因为在同步数据的时候他只是**fork**了一个子进程去做持久化的，而且他在数据恢复的时候速度比**AOF**来的快。

> **缺点**

* 内存数据全量同步，数据量大的状况下，会由于 I/O 而严重影响性能。
* 可能会因为 Redis 宕机而丢失从当前至最近一次快照期间的数据。



##### 方式二：AOF

**快照不是很持久**。如果运行 Redis 的计算机停止运行，电源线出现故障或者您 `kill -9` 的实例意外发生，则写入 Redis 的最新数据将丢失。尽管这对于某些应用程序可能不是什么大问题，但有些使用案例具有充分的耐用性，在这些情况下，快照并不是可行的选择。

**AOF(Append Only File - 仅追加文件)** 它的工作方式非常简单：每次执行 **修改内存** 中数据集的写操作时，都会 **记录** 该操作。假设 AOF 日志记录了自 Redis 实例创建以来 **所有的修改性指令序列**，那么就可以通过对一个空的 Redis 实例 **顺序执行所有的指令**，也就是 **「重放」**，来恢复 Redis 当前实例的内存数据结构的状态。

当 Redis 收到客户端修改指令后，会先进行参数校验、逻辑处理，如果没问题，就 **立即** 将该指令文本 **存储** 到 AOF 日志中，也就是说，**先执行指令再将日志存盘**。

> **AOF 重写**

**Redis** 在长期运行的过程中，AOF 的日志会越变越长。如果实例宕机重启，重放整个 AOF 日志会非常耗时，导致长时间 Redis 无法对外提供服务。所以需要对 **AOF 日志 "瘦身"**。

**Redis** 提供了 `bgrewriteaof` 指令用于对 AOF 日志进行瘦身。其 **原理** 就是 **开辟一个子进程** 对内存进行 **遍历** 转换成一系列 Redis 的操作指令，**序列化到一个新的 AOF 日志文件** 中。序列化完毕后再将操作期间发生的 **增量 AOF 日志** 追加到这个新的 AOF 日志文件中，追加完毕后就立即替代旧的 AOF 日志文件了，瘦身工作就完成了。

> **优点**

上面提到了，**RDB**五分钟一次生成快照，但是**AOF**是一秒一次去通过一个后台的线程`fsync`操作，那最多丢这一秒的数据。

**AOF**在对日志文件进行操作的时候是以`append-only`的方式去写的，他只是追加的方式写数据，自然就少了很多磁盘寻址的开销了，写入性能惊人，文件也不容易破损。

**AOF**的日志是通过一个叫**非常可读**的方式记录的，这样的特性就适合做**灾难性数据误删除**的紧急恢复了，比如公司的实习生通过**flushall**清空了所有的数据，只要这个时候后台重写还没发生，你马上拷贝一份**AOF**日志文件，把最后一条**flushall**命令删了就完事了。

> **缺点**

一样的数据，**AOF**文件比**RDB**还要大，恢复时间长。

**AOF**开启后，**Redis**支持写的**QPS**会比**RDB**支持写的要低，他不是每秒都要去异步刷新一次日志嘛**fsync**，当然即使这样性能还是很高

##### 混合持久化

重启 Redis 时，我们很少使用 `rdb` 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 `rdb` 来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。

**Redis 4.0** 为了解决这个问题，带来了一个新的持久化选项——**混合持久化**。将 `rdb` 文件的内容和增量的 AOF 日志文件存在一起。这里的 AOF 日志不再是全量的日志，而是 **自持久化开始到持久化结束** 的这段时间发生的增量 AOF 日志，通常这部分 AOF 日志很小：

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7PMcYtBZdH78LrPP2OrMV8sicEbY3SLIW8PerMMmjy3A7abvbYXPnRxuWsVLecdjkB36To07BoC5g/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

于是在 Redis 重启的时候，可以先加载 `rdb` 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，重启效率因此大幅得到提升。

#### Redis 数据的恢复

**RDB 和 AOF 文件共存情况下的恢复流程如下图：**

<img src="https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwYKLxiaKZ8VC0Gyht0PlvxdfkHHrp6Alickok5dByWShGJwKNB9eiaNpQw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 80%;" />

从图可知，Redis 启动时会**先检查 AOF** 是否存在，如果 AOF 存在则直接加载 AOF，如果不存在 AOF，则直接加载 RDB 文件。

### 缓存可能出现的问题

#### 缓存雪崩

缓存雪崩是指在我们**设置缓存时采用了相同的过期时间**，**导致缓存在某一时刻同时失效**，请求全部转发到DB，DB瞬时压力过重雪崩。由于原有缓存失效，新缓存未到期间所有原本应该访问缓存的请求都去查询数据库了，而对数据库CPU和内存造成巨大压力，严重的会造成数据库宕机。

<img src="https://mmbiz.qpic.cn/mmbiz_png/uChmeeX1FpytRQo5EwsGvK5H8uEWAVRJd1o4XNuwCu5LqAD1aaxc76Q3foXHuuH9al5vnicJ4wM1s436icEZ9Apg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

> **问题排查**

1. 在一个较短的时间内，缓存中较多的 key 集中过期
2. 此周期内请求访问过期的数据，redis 未命中，redis 向数据库获取数据
3. 数据库同时接收到大量的请求无法及时处理
4. Redis 大量请求被积压，开始出现超时现象
5. 数据库流量激增，数据库崩溃
6. 重启后仍然面对缓存中无数据可用
7. Redis 服务器资源被严重占用，Redis 服务器崩溃
8. Redis 集群呈现崩塌，集群瓦解
9. 应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃
10. 应用服务器，redis，数据库全部重启，效果不理想

> **解决方案**

处理缓存雪崩简单，在批量往 **Redis** 存数据的时候，把**每个 Key 的失效时间都加个随机值**就好了，这样可以保证数据不会在同一时间大面积失效

```java
setRedis（Key，value，time + Math.random() * 10000）;
```

或者设置**热点数据永远不过期**，有更新操作就更新缓存就好了（比如运维更新了首页商品，那你刷下缓存就完事了，不要设置过期时间），电商首页的数据也可以用这个操作，保险。

#### 缓存穿透

缓存穿透是指**缓存和数据库中都没有的数据**，而用户不断发起请求，我们数据库的 id 都是1开始自增上去的，如发起为id值为 -1 的数据或 id 为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大，严重会击垮数据库。

<img src="https://mmbiz.qpic.cn/mmbiz_png/uChmeeX1FpytRQo5EwsGvK5H8uEWAVRJEm0y5oDpHIDdCAMSicSt27MicjJlapL7ns97rljuTsa8xy2TGoCs1I5w/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

**像这种你如果不对参数做校验，数据库 id 都是大于 0 的，我一直用小于 0 的参数去请求你，每次都能绕开 Redis 直接打到数据库，数据库也查不到，每次都这样，并发高点就容易崩掉了。**

> **解决方案**

1. **增加校验**

   在接口层增加校验，不合法的参数直接 return

2. **缓存空值**

   如果一个查询返回的数据为空（不管是数据不存在，还是系统故障）我们仍然把这个**空结果进行缓存**，但它的过期时间会很短，最长不超过5分钟。通过这个设置的默认值存放到缓存，这样第二次到缓存中获取就有值了，而不会继续访问数据库

3. **采用布隆过滤器 BloomFilter**

   优势：占用内存空间小，**位存储**；性能特别高，**使用 key 的 hash 判断 key 存不存在**

   将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

   在缓存之前在加一层 BloomFilter，在查询的时候先去 BloomFilter 去查询 key 是否存在，如果不存在就直接返回，存在再去查询缓存，缓存中没有再去查询数据库

#### 缓存击穿

在平常高并发的系统中，**大量的请求同时查询一个 key** 时，此时这个 **key 正好失效**了，就会导致大量的请求都打到数据库上面去。这种现象我们称为缓存击穿

> **问题排查**

1. Redis 中某个 key 过期，该 key 访问量巨大
2. 多个数据请求从服务器直接压到 Redis 后，均未命中
3. Redis 在短时间内发起了大量对数据库中同一数据的访问

> **解决方案**

1. **使用互斥锁**

   这种解决方案思路比较简单，就是**只让一个线程构建缓存**，**其他线程等待构建缓存的线程执行完**，**重新从缓存获取数据**就可以了。如果是单机，可以用 synchronized 或者 lock 来处理，如果是分布式环境可以用分布式锁就可以了（分布式锁，可以用memcache的add, redis的setnx, zookeeper的添加节点操作）。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/OKUamcRpPbMM5WYs6KT3deLPfg7ickffUtrkrcKpf7HyQQ0ydzlQdG7yGEnDqwuc7966pLbLyofodEvQfUsh6xg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2. **热点数据永远不过期**
   * 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期
   * 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在 key 对应的 value 里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

#### 缓存预热

缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题。**用户直接查询事先被预热的缓存数据。如图所示：**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/OKUamcRpPbMM5WYs6KT3deLPfg7ickffUD7kBVwW1CAERBsSf9rEyia2uuUIyqicYpsV8sOBBWtibGlDUoewpr7rng/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 80%;" />

如果不进行预热， 那么 Redis 初识状态数据为空，系统上线初期，对于高并发的流量，都会访问到数据库中， 对数据库造成流量的压力。



> **关系型数据库跟 Redis 本质上的区别**

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpw3kedn8KYhTFdutS1fDAiaq3kA1SIY46vib8WGiaFE7PeKxqTJiaq8wI2mMGXKOG9XPWJKbQaBtphhTA/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

**Redis **采用的是**基于内存**的采用的是**单进程单线程模型**的 **KV 数据库**，由 C 语言编写，官方提供的数据是可以达到 100000+ 的 **QPS（每秒内查询次数）**。

* **完全基于内存**，绝大部分请求是纯粹的内存操作，非常快速；

* 数据结构简单，对数据操作也简单，Redis 不使用表，不会强制用户对各个关系进行关联，不会有复杂的关系限制，其存储结构就是键值对，类似于**HashMap**，**HashMap**的优势就是查找和操作的时间复杂度都是O(1)；

* Redis 使用**单进程单线程**模型的（K，V）数据库，将数据存储在内存中，存取均不会受到硬盘 IO 的限制，因此其执行速度极快。

  另外单线程也能处理高并发请求，还可以**避免频繁上下文切换和锁的竞争**，如果想要**多核运行也可以启动多个实例**。

* 使用多路I/O复用模型，非阻塞IO；

*注：Redis 采用的 I/O 多路复用函数：epoll/kqueue/evport/select*

选用策略：

* 因地制宜，优先选择时间复杂度为 O(1) 的 I/O 多路复用函数作为底层实现。
* 由于 Select 要遍历每一个 IO，所以其时间复杂度为 O(n)，通常被作为保底方案。
* 基于 React 设计模式监听 I/O 事件。

> **既然提到了单机会有瓶颈，那你们是怎么解决这个瓶颈的**

采用集群的部署方式也就是 **Redis cluster**，并且是**主从同步读写分离**，类似 **Mysql** 的主从同步，**Redis cluster** 支撑 N 个 **Redis master node**，每个**master node**都可以挂载多个 **slave node**。

这样整个 **Redis** 就可以横向扩容了。如果你要支撑更大数据量的缓存，那就横向扩容更多的 **master** 节点，每个 **master** 节点就能存放更多的数据了。

### Redis 实现分布式锁

#### 分布式锁

分布式锁是**控制分布式系统**之间共同**访问共享资源**的一种**锁**的实现。如果一个系统，或者不同系统的不同主机之间共享某个资源时，往往需要**互斥**，来排除干扰，满足数据一致性。

**分布式锁需要解决的问题如下：**

* **互斥性：**任意时刻只有一个客户端获取到锁，不能有两个客户端同时获取到锁。
* **安全性：**锁只能被持有该锁的客户端删除，不能由其他客户端删除。
* **死锁：**获取锁的客户端因为某些原因而宕机继而无法释放锁，其他客户端再也无法获取锁而导致死锁，此时**需要有特殊机制来避免死锁**。
* **容错：**当各个节点，如某个 Redis 节点宕机的时候，客户端仍然能够获取锁或释放锁。

#### 如何使用 Redis 实现分布式锁

**使用 SETNX(*SET if Not eXists*) 实现**，SETNX key value：如果 Key 不存在，则创建并赋值。

该命令时间复杂度为 O(1)，如果设置成功，则返回 1，否则返回 0。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwpTGsvibEroO3YP2gO8fS8ibwXl28XqGUr5dziaSHapJ1jELy7ym2CYAicA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由于 SETNX 指令操作简单，且是**原子性**的，所以初期的时候经常被人们作为分布式锁，我们在应用的时候，可以在某个共享资源区之前先使用 SETNX 指令，查看是否设置成功。

如果**设置成功**则说明前方**没有客户端正在访问该资源**，如果**设置失败**则说明**有客户端正在访问该资源**，那么**当前客户端就需要等待**。

但是如果真的这么做，就会存在一个问题，因为 **SETNX 是长久存在**的，所以假设一个**客户端正在访问资源**，并且**上锁**，那么当这个客户端**结束访问**时，**该锁依旧存在**，后来者也无法成功获取锁，这个该如何解决呢？

由于 SETNX 并不支持传入 EXPIRE 参数，所以我们可以直接**使用 EXPIRE 指令来对特定的 Key 来设置过期时间**。

```
EXPIRE key seconds
```

![img](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQp8on3IsH47HfiadUUbqSCL0Wt1bicrmicjzH5SGJQmcalzsKcfeHXHcq1GaDGzBicgAicgotjxg7t18qg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```java
RedisService redisService = SpringUtils.getBean(RedisService.class);
long status = redisService.setnx(key,"1");
if(status == 1){
  redisService.expire(key,expire);
  doOcuppiedWork();
}
```

这段程序存在的问题：假设程序运行到第二行出现异常，那么程序来不及设置过期时间就结束了，则 **Key 会一直存在**，等同于锁一直被持有无法释放。

出现此问题的根本原因为：**原子性得不到满足**。

**解决：**从 Redis 2.6.12 版本开始，我们就可以使用 Set 操作，将 SETNX 和 EXPIRE 融合在一起执行，具体做法如下：

* **EX second：**设置键的过期时间为 Second 秒。
* **PX millisecond：**设置键的过期时间为 MilliSecond 毫秒。
* **NX：**只在键不存在时，才对键进行设置操作。
* **XX：**只在键已经存在时，才对键进行设置操作。

```
SET KEY value [EX seconds] [PX milliseconds]  [NX|XX]
```

注：SET 操作成功完成时才会返回 OK，否则返回 nil。

有了 SET 我们就可以在程序中使用类似下面的代码实现分布式锁了：

```java
RedisService redisService = SpringUtils.getBean(RedisService.class);
String result = redisService.set(lockKey,requestId,SET_IF_NOT_EXIST,SET_WITH_EXPIRE_TIME,expireTime);
if("OK.equals(result)"){
  doOcuppiredWork();
}
```

### Redis 实现异步队列

**①使用 Redis 中的 List 作为队列**

使用上文所说的 Redis 的数据结构中的 **List 作为队列 Rpush 生产消息**，**LPOP 消费消息**。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwTW3XlkIBpIyEibriaosAXPd3QjESn09MiaeicZYsBxNLic8zkrjedOsq90g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时我们可以看到，该队列是使用 Rpush 生产队列，使用 LPOP 消费队列。

在这个生产者-消费者队列里，当 LPOP 没有消息时，证明该队列中没有元素，并且生产者还没有来得及生产新的数据。

**缺点：**LPOP 不会等待队列中有值之后再消费，而是直接进行消费。

**弥补：**可以通过在应用层引入 Sleep 机制去调用 LPOP 重试。

**②使用 BLPOP key [key…] timeout**

BLPOP key [key …] timeout：阻塞直到队列有消息或者超时。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwy6s6rVVoXGZEXVGsdJf35Paw1ib1NdZ6j4Wtsa3S4cYia5F2AoLmG2ow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

<img src="https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwBZUM5iaXpm5ouvYPr9rjYUUicCPWZUCrjaLQYZiaWkGY74OKdOjiaT2eEw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjw7bRmic3pTMSjsOZjVgzr1GuWh5IibGNgBXBmD0gNhz3lRziawY8SGAOsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**缺点：**按照此种方法，我们**生产后的数据只能提供给各个单一消费者消费**。能否实现生产一次就能让多个消费者消费呢？

**③Pub/Sub：主题订阅者模式**

发送者（Pub）发送消息，订阅者（Sub）接收消息。订阅者可以订阅任意数量的频道。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwDzmQqKkoBvSeqcgazlWYy8BAj149DrE3jcicMvp9MqTg3FURIAYwjug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Pub/Sub模式的缺点：**消息的发布是无状态的，无法保证可达。对于发布者来说，消息是“即发即失”的。

此时如果某个消费者在生产者发布消息时下线，重新上线之后，是无法接收该消息的

### Redis 的同步机制

#### 主从同步原理

Redis 一般是使用**一个 Master 节点来进行写操作**，而**若干个 Slave 节点进行读操作**，Master 和 Slave 分别代表了一个个不同的 Redis Server 实例。

另外定期的数据备份操作也是单独选择一个 Slave 去完成，这样可以最大程度发挥 Redis 的性能，为的是保证数据的弱一致性和最终一致性。

另外，Master 和 Slave 的数据不是一定要即时同步的，但是在一段时间后 Master 和 Slave 的数据是趋于同步的，这就是最终一致性。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwQzh9pIdAPYlnPYkzrgsL5cJOakpFkW73F2ZXQfk1zicQ7UdZWP9NoKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**全同步过程如下：**

* Slave 发送 Sync 命令到 Master。
* Master 启动一个后台进程，将 Redis 中的数据快照保存到 RDB 快照中。
* Master 将保存**数据快照期间接收到的写命令缓存**起来。
* Master 完成写文件操作后，将 RDB 快照发送给 Slave。
* 使用新的 AOF 文件替换掉旧的 AOF 文件。
* Master 将这期间收集的增量写命令发送给 Slave 端。

**增量同步过程如下：**

* Master 接收到用户的操作指令，判断是否需要传播到 Slave。
* 将操作记录追加到 AOF 文件。
* 将操作传播到其他 Slave：对齐主从库；往响应缓存写入指令。
* 将缓存中的数据发送给 Slave。

#### Redis Sentinel（哨兵）

**主从模式弊端：当 Master 宕机后，Redis 集群将不能对外提供写入操作**。Redis Sentinel 可解决这一问题。

**解决主从同步 Master 宕机后的主从切换问题：**

* **监控：**检查主从服务器是否运行正常。
* **提醒：**通过 API 向管理员或者其它应用程序发送故障通知。
* **自动故障迁移：**主从切换（在 Master 宕机后，将其中一个 Slave 转为 Master，其他的 Slave 从该节点同步数据）。

### Redis 集群

> 如何从海量数据里快速找到所需

**①分片**

按照**某种规则去划分数据**，**分散存储**在多个节点上。通过将数据分到多个 Redis 服务器上，来减轻单个 Redis 服务器的压力。

**②一致性 Hash 算法**

既然要将数据进行分片，那么**通常的做法就是获取节点的 Hash 值**，然后**根据节点数求模**。

但这样的方法有明显的弊端，**当 Redis 节点数需要动态增加或减少的时候**，**会造成大量的 Key 无法被命中**。所以 Redis 中引入了一致性 Hash 算法。

该算法**对 2^32 取模**，**将 Hash 值空间组成虚拟的圆环**，**整个圆环按顺时针方向组织**，每个节点依次为 0、1、2…2^32-1。

之后**将每个服务器进行 Hash 运算**，确定服务器在这个 Hash 环上的地址，**确定了服务器地址后**，**对数据使用同样的 Hash 算法**，**将数据定位到特定的 Redis 服务器上**。

**如果定位到的地方没有 Redis 服务器实例**，则继续**顺时针寻找**，**找到的第一台服务器**即该数据最终的服务器位置

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwsQ9iaddqzNWPWPuQXnibeTHW579EiaQotiaQE26fibjSelmgHd3CKqWcFgw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**③Hash 环的数据倾斜问题**

Hash 环在**服务器节点很少**的时候，**容易遇到服务器节点不均匀**的问题，这会造成**数据倾斜**，数据倾斜指的是**被缓存的对象大部分集中在 Redis 集群的其中一台或几台服务器上**。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjwQgoEqozHkj8OiawOuzn1ERlxIoHDvDOSl22Das0NyhRlN1kkjZleibWQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，一致性 Hash 算法运算后的数据大部分被存放在 A 节点上，而 B 节点只存放了少量的数据，久而久之 A 节点将被撑爆。

针对这一问题，可以引入**虚拟节点**解决。简单地说，就是**为每一个服务器节点计算多个 Hash**，**每个计算结果位置都放置一个此服务器节点**，称为虚拟节点，可以在**服务器 IP 或者主机名后放置一个编号实现**。

![img](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhoI8AZ5iaIZIhbVs0wSkASjw9Z3xyxRjNI2HYqKSrzUwTgoIy6jHhEXkwsdJvibLkCszHxQTnFuUibLA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

例如上图：将 NodeA 和 NodeB 两个节点分为 Node A#1-A#3，NodeB#1-B#3。