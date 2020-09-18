## Spark 基础解析

### 概述

#### Spark 内置模块

**Spark Core：**实现了 Spark 的**基本功能**，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。Spark Core 中还包含了对**弹性分布式数据集（RDD）的 API 定义**

**Spark SQL：**Spark 用来**操作结构化数据**的程序包。通过 Spark SQL，我们可以使用 SQL 或者 Apache Hive 版本的 SQL（HQL）来查询数据。Spark SQL 支持多种数据源，比如 Hive 表、Parquet 以及 JSON 等

**Spark Streaming：**Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应

**Spark MLlib：**提供常见的机器学习功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能

**Spark GraphX：**图计算

**集群管理器：**Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器（Cluster Manager）上运行，包括 Hadoop Yarn、Apache Mesos，以及 Spark 自带的一个简易调度器，叫做独立调度器

#### Spark 特点

1. **快：**与 MR 相比，Spark 基于内存的运算要快 100 倍以上，基于硬盘的运算也要快 10 倍以上。Spark 实现了高效的 **DAG** 执行引擎，可以通过基于内存来高效处理数据流。计算的中间结果是存在内存上的
2. **易用**
3. **通用**
4. **兼容性**

### 运行模式

#### 重要角色

##### Driver（驱动器）

Spark 的驱动器是执行开发程序中的 main 方法的进程。它负责开发人员编写的用来创建 SparkContext、创建 RDD，以及进行 RDD 的转化操作和行动操作代码的执行。

如果用 spark shell，当启动 spark shell 的时候，系统后台自启了一个 Spark Driver，就是在 Spark shell 中预加载一个叫做 sc 的 SparkContext 对象。如果 Driver 终止，那么 Spark 应用也就结束了

主要负责：

1. **把用户程序转为作业（job）**
2. 跟踪 Executor 的运行状况
3. 为执行器节点调度任务
4. UI 展示应用运行状况

##### Executor（执行器）

Spark Executor 是一个工作进程，负责在 Spark 作业中运行任务，任务间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点上继续运行

主要负责：

1. 负责运行组成 Spark 应用的任务，并将结果返回给 Driver
2. 通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算

#### Local 模式

local 模式就是运行在一台计算机上的模式，通常就是用于在本机上练手和测试，可以通过以下集中方式设置 Master

**local：**所有计算都运行在一个线程当中

**local[K]：**指定使用几个线程来运行计算

**local[*]：**这种模式直接帮你按照 Cpu 最多 Cores 来设置线程数

```shell
# 提交 Spark 程序
bin/spark-submit \
--class <main-class> \ # 设置应用的启动类
--master <master-url> \ # 指定 Master 的地址，默认为 local
--executor-memory 1G \ # 指定每个 executor 可用内存为 1G
--total-executor-cores 2 \ # 指定每个 executor 使用的 cpu 核数为 2 个
--deploy-mode <deploy-mode> \ # 是否发布你的驱动到 worder 节点（cluster）或者作为一个本地客户端（Client）（default：client）
--conf <key>=<value> \ # 任意的 Spark 配置属性，格式为 key = value
... # other options
<application-jar> \ # 打包好的应用 jar，包含依赖
[application-arguments] # 传给 main() 方法的参数
```

#### Standalone 模式

构建一个由 Master+Slave 构成的 Spark 集群，Spark 运行在集群中

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200803213840165.png" alt="image-20200803213840165"  />

#### Yarn 模式

Spark 客户端直接连接 Yarn，不需要额外构建 Spark 集群，有 yarn-client 和 yarn-cluster 两种模式，主要区别在于：Driver 程序的运行节点

yarn-client：Driver 程序运行在客户端，适用于交互、调试，希望立即看到 app 的输出

yarn-cluster：Driver 程序运行在由 RM 启动的 AppMaster，适用于生产环境

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200803213742.png"  />

### WordCount 程序

```scala
object WordCount {
    def main(args: Array[String]):Unit={
        // 创建 SparkConf 并设置 APP 名称
        val conf = new SparkConf().setAppName("WordCount");
        // 创建 SparkContext，该对象是提交 Spark APP 的入口
        val sc= new SparkContext(conf);
        // 使用 sc 创建 RDD 并执行相应的 transformation 和 action
        val res: RDD[(String, Int)] = sc.textFile("input/words").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_+_)
        // 打印
        res.collect().foreach(print)
        // 关闭连接
        sc.stop()
    }
}
```



## Spark Core

### RDD 概述

#### 定义

RDD 是弹性分布式数据集，是 Spark 中最基本的数据抽象，代码中是一个抽象类，它代表一个**不可变、可分区、里面的元素可并行计算**的集合

#### 特点

**分区**

RDD 逻辑上是分区的，由一个或多个分区组成。对于 RDD 来说，每个分区会被一个计算任务所处理。

**只读**

RDD 是只读的，要想改变 RDD 中的数据，只能在现有的 RDD 基础上创建新的RDD

由一个 RDD 转化到另一个 RDD，可以通过丰富的操作算子实现

RDD 的操作算子包括两类，一类叫做 transformations，它是用来将 RDD 进行转化，构建 RDD 的血缘关系；另一类叫做 actions，它是用来触发 RDD 的计算，得到 RDD 的相关计算结果或者将 RDD 保存到文件系统中。

**依赖**

RDD 通过操作算子进行转换，转换得到的新 RDD 包含了从其他 RDD 衍生所必需的信息，RDD 之间维护着这种血缘关系，也称之为**依赖**。部分分区数据丢失后，可以通过依赖关系重新计算丢失的分区数据，而不是对 RDD 的所有分区进行重新计算。

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200804093915.png" style="zoom:33%;" />

依赖包括两种：一种是**窄依赖**，上游 RDD 和下游 RDD 分区之间的关系是**一对一**的，或者上游 RDD 一个分区只对应一个下游 RDD 的分区的情况下的上游 RDD 和下游 RDD 的分区关系是**多对一**的，**不会有 shuffle 产生**，另一种是**宽依赖**，上游 RDD 与 子 RDD 分区之间的关系是**一对多**，**会有 shuffle 的产生**，上游 RDD 的一个分区的数据去到了下游 RDD 的不同分区里面

**缓存**

如果在应用程序中多次使用同一个 RDD，可以将 RDD 缓存起来，该 RDD 只有在第一次计算的时候会根据血缘关系得到分区的数据，在后续其他地方用到该 RDD 的时候，会直接从缓存处取而不用根据血缘关系计算，这样就加速后期的重用。

**CheckPoint**

虽然 RDD 的血缘关系天然地可以实现容错，当 RDD 的某个分区数据失败或丢失，可以通过血缘关系重建。但是对于长时间迭代型应用来说，随着迭代的进行，RDD 之间的血缘关系会越来越长，一旦在后续迭代过程中出错，则需要通过非常长的血缘关系去重建，势必影响性能。为此，RDD 支持 checkpoint 将数据保存到持久化的存储中，这样就可以**切断之前的血缘关系**，因为 checkpoint 后的 RDD 不需要知道它的上游 RDDs 了，它可以从 checkpoint 处拿到数据



### RDD 编程

#### RDD 的创建

##### **从集合创建**

```scala
sc.parallelize()
sc.makeRDD()
```

**从外部存储系统的数据集创建**

```scala
sc.textFile();
```

#### RDD 的转换

##### Value 类型

```scala
map(func)
mapPartitions(func)
/* 
map() 和 mapPartitions() 的区别：
map()：一次处理一条数据
mapPartitions()：每次处理一个分区的数据，这个分区的数据处理完后，原 RDD 中分区的数据才能释放，可能导致 OOM
*/
mapPartitionsWithIndex(func)
flatMap(func) // flatMap() 比 map() 多一步扁平化操作
glom() // 将每一个分区形成一个数组，形成新的 RDD 类型是 RDD[Array[T]]
groupBy(func)
filter(func)
sample(withReplacement, fraction, seed) // 以指定的随机种子随机抽样出数量为 fraction 的数据，withReplacement 表示是抽出的数据是否放回，true 为有放回的抽样，false 为无放回的抽样，seed 用于指定随机数生成器种子
distinct([numTasks])
coalesce(numPartitions)
repartition(numPartitions)
/*
coalesce 和 repartition 的区别
coalesce 重新分区，可以选择是否进行 shuffle 过程
repartition 实际上就是调用 coalesce，默认是进行 shuffle 的
*/
sortBy(func, [ascending], [numTasks]) // 使用 func 先对数据进行处理，按照处理后的数据比较结果排序
pipe(command,[envVars]) // 管道，针对每个分区，都执行一个 shell 脚本，返回输出的 RDD，注意：脚本需要放到 Worker 节点可以访问到的位置

```

##### 双 Value 类型交互

```scala
union(otherDataSet) // 并集
subtract(otherDataset) // 差集
intersection(otherDataset) // 交集
cartesian(otherDataset) // 笛卡尔积
zip(otherDataset) // 将两个 RDD 组合成 key/value 形式的 RDD，这里默认两个 RDD 的 partition 数量以及元素数量都相同，否则会抛出异常
```

##### Key-Value 类型

```scala
partitionBy // 对 pairRDD 进行分区操作，产生 shuffle 过程
groupByKey
reduceByKey
/*
reduceByKey 和 groupByKey 的区别
reduceByKey：按照 key 进行聚合，在 shuffle 之前有 combine(预聚合)操作，返回结果是 RDD[k,v]
groupByKey：按照 key 进行分组，直接进行 shuffle
*/
aggregateByKey(zeroValue:U,[partitioner: Partitioner]) (seqOp: (U, V) => U,combOp: (U, U) => U)
// zeroValue：给每一个分区中的每一个 key 一个初始值
// seqOp：函数用于在每一个分区中用初始值逐步迭代 value
// combOp：函数用于合并每个分区中的结果
foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]
// aggregateByKey 的简化操作，seqOp 和 combOp 相同
combineByKey[C](createCombiner: V => C,  mergeValue: (C, V) => C,  mergeCombiners: (C, C) => C) // 对相同 K，把 V 合并成一个集合
// createCombiner：combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就和之前的某个元素的键相同。如果这是一个新的元素，combineByKey()会使用一个叫做 createCombiner() 的函数来创建那个键对应的累加器的初始值
// mergeValue：如果这是一个在处理当前分区之前已经遇到的键，它会使用 mergeValue() 方法将该键的累加器对应的当前值与这个新的值进行合并
// mergeCombiners：由于每个分区都是独立处理的，因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器，就需要使用用户提供的 mergeCombiners() 方法将各个分区的结果进行合并
sortByKey
mapValues // 只对 V 进行操作
join(otherDataset,[numTasks])
cogroup(otherDataset, [numTasks])
```

**aggregateByKey 案例分析**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200804112517.png" style="zoom: 33%;" />

**combineByKey 案例分析**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200804113456.png" style="zoom: 33%;" />

#### Action

```scala
reduce(func)
collect()
count()
first()
take(n)
takeOrdered(n)
aggregate(zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)
fold(num)(func)
saveAsTextFile(path)
saveAsSequenceFile(path)
saveAsObjectFile(path)
countByKey()
foreach(func)
```

#### RDD 中的函数传递

**传递一个方法**

使类继承 scala.Serializable

```scala
class Search() extends Serializable {...}
```

**传递一个属性**

两种解决方案：

```scala
// 1. 使类继承 scala.Serializable
class Search() extends Serializable {...}
// 2. 将类变量 query 赋值给局部变量
def getMatche2(rdd:RDD[String]):RDD[String] = {
    val query_:String = this.query
    rdd.filter(x => x.contains(query_))
}
```

#### RDD 依赖关系

##### Lineage

RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage（血统）记录下来，以便恢复丢失的分区。RDD 的 Lineage 会记录 RDD 的元数据信息和转换行为，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区

```scala
// 查看 RDD 的 Lineage
toDebugString
// 查看依赖类型
dependencies
```

##### 宽依赖

##### 窄依赖

##### DAG

DAG 叫做有向无环图，原始的 RDD 通过一系列的转换就形成了 DAG，根据 RDD 之间的依赖关系的不同将 DAG 划分成不同的 stage，对于窄依赖，partition 的转换处理在 stage 中完成计算。对于宽依赖，由于有 shuffle 的存在，只能在 parent RDD 处理完成后，才能开始接下来的计算，因此**宽依赖是划分 stage 的依据**

##### 任务划分

RDD 任务可以切分为：Application、Job、Stage 和 Task

1. Application：初始化一个 SparkContext 即生成一个 Application
2. Job：一个 Action 算子就会生成一个 Job
3. Stage：根据 RDD 之间的依赖关系的不同将 Job 划分成不同的 Stage
4. Task：Stage 是一个 TaskSet，task 数目由该 Stage 最后一个 RDD 中的 partition 个数决定，将 Stage 划分的结果发送到不同的 Executor 执行即为一个 Task

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200804142449.png" style="zoom:33%;" />

#### RDD 缓存

Spark 非常快速的一个原因是 RDD 支持缓存。成功缓存后，如果之后的操作使用到了该数据集，直接从缓存中获取。虽然缓存也有丢失的风险，但是由于 RDD 之间的依赖关系，如果某个分区的缓存数据丢失，只需要重新计算该分区的数据即可。

##### 缓存级别

| Storage Level                          | Meaning                                                      |
| -------------------------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY                            | 默认的缓存级别，将 RDD 以反序列化的 Java 对象的形式存储在 JVM 中，如果内存空间不够，则部分分区数据将不再缓存 |
| MEMORY_ONLY_DISK                       | 将 RDD 以反序列化的 Java 对象的形式存储在 JVM 中，如果内存空间不够，将未缓存的数据存在磁盘上，在需要这些分区时从磁盘读取 |
| MEMORY_ONLY_SER                        | 将 RDD 以序列化的 Java 对象的形式进行存储（每一个分区为一个 byte 数组）。这种方式比反序列化对象节省存储空间，但在读取时会增加 CPU 的计算负担。仅支持 Java 和 Scala |
| MEMORY_ONLY_DISK_SER                   | 类似于 MEMORY_ONLY_SER，但是溢出的数据会存储到磁盘，而不是在用到它们时重新计算。仅支持 Java 和 Scala |
| DISK_ONLY                              | 只在磁盘上缓存 RDD                                           |
| MEMORY_ONLY_2，MEMORY_ONLY_DISK_2，etc | 与上面的对应级别功能相同，但是会为每个分区在集群中的两个节点上建立副本 |
| OFF_HEAP                               | 与 MEMORY_ONLY_SER 类似，但将数据存储在堆外内存。这需要启动堆外内存 |



##### 移除缓存

Spark 会自动监视每个节点上的缓存使用情况，并按照 LRU 的规则删除旧分区数据。

也可以使用 `RDD.unpersist()`方法进行手动删除。



#### RDD CheckPoint

为当前 RDD 设置检查点，该函数将会创建一个二进制文件，并存储到 checkpoint 目录中，该目录是用 SparkContext.setCheckpointDir() 设置的。在 checkpoint 的过程中，该 RDD 的所有依赖于父 RDD 的信息将全部被移除。对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发



### 键值对 RDD 数据分区器

Spark 目前支持 **Hash 分区**和 **Range 分区**，用户也可以**自定义分区**，Hash 分区为当前的默认分区，Spark 中分区器直接决定了 RDD 中的分区的个数，RDD中每条数据经过 shuffle 过程属于哪个分区和 Reduce 的个数

> 注意：
>
> 1. 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区器的值是 None
> 2. 每个 RDD 的分区 ID 范围：0 ~ numPartitions-1，决定这个值是属于哪个分区

#### 获取 RDD 分区

可以通过使用 RDD 的 partitioner 属性来获取 RDD 的分区方式

#### Hash 分区

HashPartitioner 分区的原理：对于给定的 key，计算其 hashCode，除以子 RDD 分区的个数取余

#### Range 分区

**HashPartitioner 分区弊端：**可能导致每个分区中数据量的不均匀，极端情况下会导致某些分区拥有 RDD 的全部数据

**RangePartitioner 作用：**将一定范围内的数映射到某一个分区内，尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，一个分区中的元素肯定都是比另一个分区内的元素小或者大，但是分区内的元素是不能保证顺序的。

**实现过程：**

1. 先从整个 RDD 中抽取样本数据，将样本数据排序，计算出每个分区的最大 key 值，形成一个 Array[KEY] 类型的数组变量 rangeBounds
2. 判断 key 在 rangeBounds 中所处的范围，给出该 key 值在下一个 RDD 中的分区 id 下标；该分区器要求 RDD 中的 KEY 类型必须是可以排序的

#### 自定义分区

```scala
class MyPartitioner extends Partitioner {
  // 返回创建出来的分区数
  override def numPartitions: Int = ???
  // 返回给定键的分区编号(0 到 numPartitions-1)
  override def getPartition(key: Any): Int = ???
  // 判断相等性的标准方法，Spark 需要用这个方法来检查分区器对象是否和其他分区器实例相同，这样 Spark 才可以判断两个 RDD 的分区方式是否相同
  override def equals(obj: Any): Boolean = super.equals(obj)
}
```



### 数据读取与保存

Spark 的数据读取及数据保存可以从两个维度来作区分：**文件格式**以及**文件系统**

文件格式分为：Text 文件、Json 文件、Csv 文件、Sequence 文件以及 Object 文件

文件系统分为：本地文件系统、HDFS、HBASE 以及 数据库

#### 文件类数据读取与保存

**Text 文件**

```scala
// 数据读取
sc.textFile(String)
// 数据保存
rdd.saveAsTextFile(String)
```

**Json 文件**

**Sequence 文件**

SequenceFile 文件是 Hadoop 用来存储二进制形式的 key-value 对而设计的一种平面文件（Flat File）。Spark 有专门用来读取 SequenceFile 的接口。在 SparkContext 中，可以调用 `SequenceFile[ keyClass, valueClass ](path)`

> 注意：SequenceFile 文件只针对 PairRDD

```scala
// 将 RDD 保存为 Sequence 文件
rdd.saveAsSequenceFile()
// 读取 Sequence 文件
sc.sequenceFile[Int,Int]()
```

**对象文件**

对象文件是将对象序列化后保存的文件，采用 Java 的序列化机制

```scala
// 将 RDD 保存为 Object 文件
rdd.saveAsObjectFile()
// 读取 Object 文件
sc.obectFile[Int]()
```

#### 文件系统类数据读取与保存

**HDFS**

**MySql 数据库连接**

**HBase 数据库**



### RDD 编程进阶

#### 累加器

累加器用来对信息进行聚合，通常在向 Spark 传递函数时，比如使用 map() 或者用 filter() 传条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本，更新这些副本的值也不会影响驱动器中的对应变量。如果我们想实现所有分片处理时更新共享变量的功能，那么累加器可以实现我们想要的效果

**系统累加器**

```scala
val notice = sc.textFile("")
// 通过在驱动器中调用 SparkContext.accumulator(initialValue) 方法，创建出存有初始值的累加器
val blanklines = sc.accumulator(0)
// Spark 闭包里的执行器代码可以使用累加器的 += 方法增加累加器的值
val tmp = notice.flatMap(line => {
    if (line == "")
    	blanklines += 1
    line.split(" ")
})
// 驱动器程序可以调用累加器的 value 属性来访问累加器的值
blanklines.value
```

> 注意：工作节点上的任务不能访问累加器的值。从这些任务的角度来看，累加器是一个只写的变量

**自定义累加器**

实现自定义累加器需要继承 AccumulatorV2 并重写方法

#### 广播变量（调优策略）

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。

```scala
// 通过一个类型 T 的对象调用 SparkContext.broadcast 创建出一个 Broadcast[T] 对象。任何可序列化的类型都可以这么实现
val broadcastVar = sc.broadcast(Array(1,2,3))
// 通过 value 属性访问该对象的值
broadcastVar.value
// 变量只会发到各个节点一次，应作为只读值处理（修改这个值不会影响到别的节点）
```



### 扩展

#### RDD 相关概念关系

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200804173812.png" style="zoom:33%;" />

输入可能以多个文件的形式存储在 HDFS 上，每个 File 都包含了很多块（block），当 Spark 读取这些文件作为输入时，会根据具体数据格式对应的 InputFormat 进行解析，一般是将若干个 block 合并成一个输入分片，成为 InputSplit，注意 InputSplit 不能跨越文件。随后将为这些输入分片生成具体的 Task。InputSplit 与 Task 是一一对应的关系。随后这些具体的 Task 每个都会被分配到集群上的某个节点的某个 Executor 去执行

1. 每个节点可以起一个或多个 Executor
2. 每个 Executor 由若干 Core 组成，每个 Executor 的每个 Core 一次只能执行一个 Task
3. 每个 Task 执行的结果就是生成了目标 RDD 的一个 Partition

> 注意：这里的 core 是虚拟的 core 而不是机器的物理 CPU 核，可以理解为就是 Executor 的一个工作线程。而 Task 被执行的并发度 = Executor 数目 * 每个 Executor 核数

至于 Partition 的数目：

1. 对于数据读入阶段，例如 sc.textFile，输入文件被划分为多少 InputSplit 就会需要多少初始 Task
2. 在 Map 阶段 Partition 数目保持不变
3. 在 Reduce 阶段，RDD 的聚合会触发 shuffle 操作，聚合后的 RDD 的 Partition 数目跟具体操作有关，例如 repartition 操作会聚合成指定分区数，还有一些算子是可配置的

RDD 在计算的时候，每个分区都会起一个 task，所以 RDD 的分区数目决定了总的 Task 数目。申请的计算节点（Executor）数目和每个计算节点核数，决定了你同一时刻可以并行执行的 task

比如的RDD有100个分区，那么计算的时候就会生成100个task，你的资源配置为10个计算节点，每个两2个核，同一时刻可以并行的task数目为20，计算这个RDD就需要5个轮次。如果计算资源不变，你有101个task的话，就需要6个轮次，在最后一轮中，只有一个task在执行，其余核都在空转。如果资源不变，你的RDD只有2个分区，那么同一时刻只有2个task运行，其余18个核空转，造成资源浪费。这就是在spark调优中，增大RDD分区数目，增大任务并行度的做法。



## Spark SQL

### 概述

#### 定义

Spark SQL 是 Spark 用来处理结构化数据的一个模块，它提供了 2 个编程抽象：**DataFrame** 和 **DataSet**，并且作为分布式 SQL 查询引擎的作用

Spark SQL 转换成 RDD，然后提交到集群执行，执行效率非常快

#### 特点

**易整合**

**统一的数据访问方式**

**兼容 Hive**

**标准的数据连接**

#### DataFrame 定义

与 RDD 类似，DataFrame 也是一个分布式数据容器。然而 DataFrame 更像传统数据库的二维表格，除了**数据**以外，还记录数据的结构信息，即 **Schema**。同时，与 Hive 类似，DataFrame 也支持**嵌套数据类型**（struct，array 和 map）

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200805091655.png" style="zoom: 25%;" />

上图体现了 DataFrame 和 RDD 的区别。左侧的 RDD[Person] 虽然以 Person 为类型参数，但 Spark 框架本身不了解 Person 类的内部结构。而右侧的 DataFrame 却提供了详细的结构信息，使得 Spark SQL 可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。

DataFrame 为数据提供了 Schema 的视图，可以把它当作数据库中的一张表来对待，DataFrame 也是懒执行的，性能比 RDD 要高，主要原因：

优化的执行计划：查询计划通过 Spark catalyst optimiser 进行优化

#### DataSet 定义

1. 是 DataFrame API 的一个扩展，是 Spark 最新的数据抽象
2. 用户友好的 API 风格，既具有类型安全检查也具有 DataFrame 的查询优化特性
3. DataSet 支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率
4. 样例类被用来在 DataSet 中定义数据的结构信息，样例类中每个属性的名称直接映射到 DataSet 中的字段名称
5. DataFrame 是 DataSet 的特例，DataFrame = DataSet[Row]，所以可以通过 as 方法将 DataFrame 转换为 DataSet。Row 是一个类型
6. DataSet 是强类型的
7. DataFrame 只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，而 DataSet 不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查



### Spark SQL 编程

#### SparkSession

SparkSession 内部封装了 SparkContext，所以计算实际上是由 SparkContext 完成的

#### DataFrame

##### 创建

```scala
// 从 Spark 数据源创建
val df = spark.read.json("")
// 从 RDD 进行转换
// 从 Hive Table 进行查询返回
```

##### SQL 风格语法（主要）

```scala
// 创建 DataFrame
val df = spark.read.json("")
// 对 df 创建临时表
df.createOrReplaceTempView("people")
// 通过 SQL 语句实现全表查询
val sqlDF = spark.sql("select * from people")
// 结果展示
sqlDF.show
```

> 注意：临时表是 Session 范围内的，Session 退出后，表就失效了。如果想应用范围内有效，可以使用全局表。注意使用全局表时需要全路径访问

```scala
// 对于 df 创建全局表
df.createGlobalTempView("people")
// 通过 SQL 语句实现全表查询
spark.sql("select * from global_temp.people").show()
```

##### DSL 风格语法（次要）

```scala
// 查看 DataFrame 的 Schema 信息
df.printSchema
```

#####  RDD 转换为 DataFrame

> 注意：如果需要 RDD 与 DF 或者 DS 之间操作，需要引入 import spark.implicits._ 【spark 不是包名，而是 SparkSession 对象的名称】

```scala
import spark.implicits._
val peopleRDD = sc.textFile("")
// 通过手动确定转换
peopleRDD.map{x => val para = x.split(",");(para(0), para(1).trim.toInt)}.toDF("name", "age")
```

```scala
// 通过反射确定（需要用到样例类）
case class People(name:String, age:Int)
// 根据样例类将 RDD 转换为 DataFrame
peopleRDD.map{x => val para = x.split(",");People(para(0), para(1).trim.toInt)}.toDF
```

##### DataFrame 转换为 RDD

```scala
// 创建一个 DataFrame
val df = spark.read.json("")
// 将 DataFrame 转换为 RDD
val dfToRDD = df.rdd
```

#### DataSet

##### 创建

```scala
// 创建样例类
case class Person(name:String, age:Long)
// 创建 DataSet
val caseClassDS = Seq(Person("Andy", 32)).toDS()
```

##### RDD 转换为 DataSet

```scala
// 创建 RDD
val peopleRDD = sc.textFile("")
// 创建样例类
case class Person(name:String, age:Long)
// 将 RDD 转换为 DataSet
peopleRDD.map(x => val para = x.split(",");People(para(0), para(1).trim.toInt)}).toDS()
```

##### DataSet 转换为 RDD

```scala
// 创建一个 DataSet
val DS = Seq(Person("Andy", 32)).toDS()
// 将 DataSet 转换为 RDD
DS.rdd
```

#### DataFrame 与 DataSet 的互操作

#### RDD、DataFrame、DataSet

**三者的共性**

1. RDD、DataFrame、DataSet 都是 Spark 平台下的分布式弹性数据集，为处理超大型数据提供遍历
2. 三者都有惰性机制，在进行创建、转换，不会立即执行，只有在遇到 Action 时，才会执行
3. 三者都会根据 Spark 的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出
4. 三者都有 partition 的概念
5. 三者有许多共同的函数，如 filter、排序等
6. 在对 DataFrame 和 DataSet 进行操作需要这个包支持 import spark.implicits._
7. DataFrame 和 DataSet 均可使用模式匹配获取各个字段的值和类型

**三者的区别**

#### 用户自定义函数



## Spark Streaming

### 概述

#### 定义

Spark Streaming 用于**流式数据**的处理。Spark Streaming 支持的数据输入源很多，例如 Kafka、Flume 和简单的 TCP 套接字等。数据输入后可以用 Spark 的高度抽象源语，如 map、reduce 等进行计算。结果也能保存在很多地方，如 HDFS，数据库等

Spark Streaming 使用**离散化流 DStream** 作为抽象表示。DStream 是随时间推移而收到的数据的序列。在内部，每个**时间区间**收到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列

#### 架构

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200805165701.png" style="zoom:25%;" />

### DStream 转换

DStream 上的原语于 RDD 类似，分为 Transformations（转换）和 Output Operations（输出）两种，此外转换操作中还有一些比较特殊的原语，如 updateStateByKey()、transform() 以及各种 Window 相关的原语

#### 无状态

无状态转化操作就是把简单的 RDD 转化操作应用到每个批次上，也就是转化 DStream 中的每一个 RDD。部分无状态转化操作列在了下表中

> 注意，针对键值对的 DStream 转化操作（比如 reduceByKey()）要添加 import StreamingContext._ 才能在 Scala 中使用

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806095305.png" style="zoom: 33%;" />

尽管这些函数看起来像作用在整个流上一样，但事实上每个 DStream 在内部是由许多 RDD（批次）组成，且无状态转化操作时分别应用到每个 RDD 上的

#### 有状态转化操作（重点）

##### UpdateStateByKey

```scala
object WorldCount {
  def main(args: Array[String]) {
      val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
      val ssc = new StreamingContext(conf, Seconds(3))
      ssc.sparkContext.setCheckpointDir("cp")
      
      val lines = ssc.socketTextStream("localhost", 9999)
      lines.flatMap(_.split(" ")).map(_,1L)
           // updateStateByKey 是有状态计算方法
      	   // 第一个参数表示 相同 key 的 value 的集合
      	   // 第二个参数表示 相同 key 的缓冲区的数据
      	   .updateStateByKey[Long](
           	(seq:Seq[Long], buffer:Option[Long]) => {
                val newBufferValue = buffer.getOrElse(0) + seq.sum
                Option(newBufferValue)
            }
           ).print()
      ssc.start()
      ssc.awaitTermination()
  }
}
```

##### Window Operations

Window Operations 可以设置窗口的大小和滑动窗口的间隔来动态的获取当前 Streaming 的允许状态。基于窗口的操作会在一个比 StreamingContext 的批次间隔更长的时间范围内，通过整合多个批次的结果，计算出整个窗口的结果

> 注意：所有基于窗口的操作都需要两个参数，分别为窗口时长以及滑动步长，两者都必须是 StreamingContext 的批次间隔的整数倍

窗口时长控制每次计算最近的多少个批次的数据，其实就是最近的 windowDuration/batchInterval 个批次。滑动步长的默认值与批次间隔相等，用来控制对新的 DStream 进行计算的间隔

## 数据倾斜解决方案

#### 过滤少数导致倾斜的 key

**使用场景**

导致倾斜的 key 就少数几个，而且对计算结果的影响不大的话，直接过滤掉这些 key

#### 提高 Shuffle 操作的并行度



