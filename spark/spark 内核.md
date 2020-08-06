## Spark 内核解析

### 核心组件

**Driver**

**Executor**

### 通用运行流程概述

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806115039.png" style="zoom: 33%;" />

任务提交后，都会先**启动 Driver 进程**，随后 Driver 进程**向集群管理器注册应用程序**，之后集群管理器根据此任务的配置文件**分配 Executor 并启动**，当 Driver 所需的资源全部满足时，Driver 开始**执行 main 函数**，Spark 查询为懒执行，当**执行到 action 算子**时开始反向推算，根据宽依赖进行 **stage 的划分**，随后**每一个 stage 对应一个 taskset**，taskset 中有多个 task，根据本地化原则，**task 会被分发到指定的 Executor 去执行**，在任务执行的过程中，Executor 也会不断与 Driver 进行通信，报告任务运行情况

### 部署模式

* Standalone

* Apache Mesos

* Hadoop Yarn

  根据 Driver 在集群中的位置不同，分为 yarn client 和 yarn cluster

#### Standalone 模式运行机制

Standalone 集群有四个重要组成部分：

1. Driver
2. Master：是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责
3. Worker：是一个进程，一个 Worker 运行在集群中的一台服务器上，主要负责两个职责，一个是用自己的内存存储 RDD 的某个或某些 partition；另一个是启动其他进程和线程（Executor），对 RDD 上的 partition 进行并行的处理和计算
4. Executor：是一个进程，一个 Worker 上可以运行多个 Executor，Executor 通过启动多个线程（task）来执行对 RDD 的 partition 进行并行计算，也就是执行我们对 RDD 定义的例如 map、flatMap、reduce 等算子操作

##### Standalone Client 模式

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806141610.png" style="zoom: 33%;" />

在 Standalone Client 模式下，**Driver 在任务提交的本地机器上运行**，Driver 启动后向 Master 注册应用程序，Master 根据 submit 脚本的资源需求找到内部资源至少可以启动一个 Executor 的所有 Worker，然后在这些 Worker 之间分配 Executor，Worker 上的 Executor 启动后会向 Driver 反向注册，所有的 Executor 注册完成后，Driver 开始执行 main 函数，之后执行到 Action 算子时，开始划分 Stage，每个 Stage 生成对应的 taskSet，之后将 task 分发到各个 Executor 上执行

##### Standalone Cluster 模式

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806142240.png" style="zoom:33%;" />

在 Standalone Cluster 模式下，任务提交后，**Master 会找到一个 Worker 启动 Driver 进程**，Driver 启动后向 Master 注册应用程序，Master 根据 submit 脚本的资源需求找到内部资源至少可以启动一个 Executor 的所有 Worker，然后在这些 Worker 之间分配 Executor，Worker 上的 Executor 启动后会向 Driver 反向注册，所有的 Executor 注册完成后，Driver 开始执行 main 函数，之后执行到 Action 算子时，开始划分 Stage，每个 Stage 生成对应的 taskSet，之后将 task 分发到各个 Executor 上执行

> Standalone  的两种模式下（Client/Cluster），Master  在接到 Driver 注册 Spark 应用程序的请求后，会获取其所管理的剩余资源能够启动一个 Executor 的所有 Worker，然后在这些 Worker 之间分发 Executor，此时的分发只考虑 Worker 上的资源是否足够使用，直到当前应用程序所需的所有 Executor 都分配完毕，Executor 反向注册完毕后，Driver 开始执行 main 程序

#### Yarn 模式运行机制

##### Yarn Client 模式

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806143443.png" style="zoom: 50%;" />

在 Yarn Client 模式下，**Driver 在任务提交的本地机器上运行**，Driver 启动后会**和 RM 通讯申请启动 AppMaster**，随后 RM 分配 Container，在合适的 **NM 上启动 AppMaster**，此时 AppMaster 的功能相等于一个 **ExecutorLauncher**，只负责**向 RM 申请 Executor 内存**

RM 接到 AppMaster 的资源申请后会分配 Container，然后 AppMaster 在资源分配指定的 **NM 上启动 Executor 进程**，Executor 进程启动后会向 Driver **反向注册**，Executor 全部注册完成后 **Driver 开始执行 main 函数**，之后执行到 **Action** 算子时，触发一个 **job**，并根据**宽依赖开始划分 stage**，每个 stage 生成对应的 **taskSet**，之后**将 task 分发到各个 Executor 上执行**

##### Yarn Cluster 模式

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806144026.png" style="zoom: 50%;" />

在 Yarn Cluster 模式下，任务提交后会和 RM 通讯申请启动 AppMaster，随后 RM 分配 Container，在合适的  NM 上启动 AppMaster，此时的 **AppMaster 就是 Driver**

Driver 启动后向 RM 申请 Executor 内存，RM 接到 AppMaster 的资源申请后会分配 Container，然后 AppMaster 在资源分配指定的 **NM 上启动 Executor 进程**，Executor 进程启动后会向 Driver **反向注册**，Executor 全部注册完成后 **Driver 开始执行 main 函数**，之后执行到 **Action** 算子时，触发一个 **job**，并根据**宽依赖开始划分 stage**，每个 stage 生成对应的 **taskSet**，之后**将 task 分发到各个 Executor 上执行**

### 通讯架构

### SparkContext 解析

### 任务调度机制

#### 任务提交流程

提交一个 Spark 应用程序，首先通过 Client 向 RM 请求启动一个 Application，同时检查是否有足够的资源满足 Application 的需求，如果资源条件满足，则准备 AppMaster 的启动上下文，交给 RM，并循环监控 Application 的状态

当提交的资源队列中有资源时，**RM 会在某个 NM 上启动 AppMaster 进程**，**AppMaster 会单独启动 Driver 后台线程**，当 Driver 启动后，AppMaster 会通过本地  RPC 连接 Driver，并**开始向 RM 申请 Container 资源运行 Executor 进程**（**一个 Executor 对应一个 Container**），**当 RM 返回 Container 资源，AppMaster 则在对应的 Container 上启动 Executor**

**Driver 线程主要是初始化 SparkContext 对象**，准备运行所需的上下文，然后一方面保持与 AppMaster 的 RPC 连接，通过 AppMaster 申请资源，另一方面根据用户业务逻辑开始**调度任务**，将任务下发到已有的空闲 Executor 上

当 RM 向 AppMaster 返回 Container 资源时，AppMaster 就尝试在对应的 Container 上启动 Executor 进程，Executor 进程起来后，会向 Driver 反向注册，注册成功后保持与 Driver 的心跳，同时等待 Driver 分发任务，当分发的任务执行完毕后，将任务状态上报给 Driver

**Client 只负责提交 Application 并监控 Application 的状态**。对于 Spark 的任务调度主要是集中在两个方面：**资源申请和任务分发**，其主要是通过 AppMaster，Driver 以及 Executor 之间来完成

#### 任务调度概述

Spark 应用程序包括 Job、Stage 以及 Task 三个概念：

* Job 是以 Action 方法为界，遇到一个 Action 方法则触发一个 Job
* Stage 是 Job 的子集，以 RDD 宽依赖（Shuffle）为界，遇到 Shuffle 做一次划分
* Task 是 Stage 的子集，以并行度（分区数）来衡量，分区数是多少，则有多少个 task

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806150336.png" style="zoom: 25%;" />

RDD 通过其 Transactions 操作，形成了 RDD 血缘关系图，即 DAG，最后通过 Action 的调用，触发 Job 并调度执行。

**DAGScheduler 负责 Stage 级的调度**：主要是将 job 切分成若干 Stages，并将 Stage 打包成 taskSet 交给 TaskScheduler 调度

**TaskScheduler 负责 Task 级的调度：**将 DAGScheduler 给过来的 TaskSet 按照指定的调度策略分发到 Executor 上执行，调度过程中 SchedulerBackend 负责提供可用资源，其中 SchedulerBackend 有多种实现，分别对接不同的资源管理系统

#### Stage 级调度

Spark 的任务调度是从 DAG 切割开始，主要是由 DAGScheduler 来完成。当遇到一个 Action 操作后就会触发一个 Job 的计算，并交给 DAGScheduler 来提交

Job 由最终的 RDD 和 Action 方法封装而成，SparkContext 将 Job 交给 DAGScheduler 提交，它会根据 RDD 的血缘关系构成的 DAG 进行切分，将一个 Job 划分为若干 Stages，具体划分策略是，**由最终的 RDD 不断通过依赖回溯判断父依赖是否是宽依赖，即以 Shuffle 为界，划分 Stage，窄依赖的 RDD 之间被划分到同一个 Stage 中，可以进行 pipeline 式的计算。**划分的 Stages 分两类，一类叫做 **ResultStage**，为 DAG 最下游的 Stage，由  Action 方法决定，另一类叫做 **ShuffleMapStage**，为下游 Stage 准备数据

**一个 Stage 是否被提交，需要判断它的父 Stage 是否执行，只有在父 Stage 执行完毕才能提交当前 Stage，如果一个 Stage 没有父 Stage，那么从该 Stage 开始提交。**Stage 提交时会将 Task 信息（分区信息以及方法等）序列化并被打包成 TaskSet 交给 TaskScheduler，一个 Partition 对应一个 Task，另一方面 TaskScheduler 会监控 Stage 的运行状态，只有 Executor 丢失或者 Task 由于 Fetch 失败才需要重新提交失败的 Stage 以调度运行失败的任务，其他类型的 Task 失败会在 TaskScheduler 的调度过程中重试

#### Task 级调度

Spark Task 的调度是由 TaskScheduler 来完成，DAGScheduler 将 Stage 打包到 TaskSet 交给 TaskScheduler，TaskScheduler 会将 TaskSet 封装为 TaskSetManager 加入到调度队列中，TaskSetManager 结构如下图所示：

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200806152805.png" style="zoom: 33%;" />

**TaskSetManager 负责监控管理同一个 Stage 中的 Tasks，TaskScheduler 就是以 TaskSetManager 为单元来调度任务**

##### 调度策略

##### 本地化调度

##### 失败重试与黑名单机制

### Spark Shuffle

#### Shuffle 的核心要点

##### ShuffleMapStage 和 FinalStage

划分的 Stages 时，最后一个 Stage 称为 FinalStage，它本质上是一个 ResultStage 对象，前面的所有 Stage 被称为 ShuffleMapStage

**ShuffleMapStage 的结束伴随着 Shuffle 文件的写磁盘**

**ResultStage 基本上对应代码中的 action 算子，即将一个函数应用在 RDD 的各个 partition 的数据集上，意味着一个 job 的运行结束**

##### Shuffle 中的任务个数

1. map 端 task 个数的确定

   shuffle 过程中的 task 个数由 RDD 分区数决定，而 RDD 的分区个数与参数 spark.default.parallelism 有密切关系

   在 Yarn Cluster 模式下，如果没有手动设置 spark.default.parallelism，则有

   ```scala
   spark.default.parallelism = max(所有 executor 使用的 core 总数, 2)
   ```

   如果进行了手动配置，则：

   ```scala
   spark.default.parallelism = 配置值
   ```

   还有一个重要的配置：

   ```scala
   spark.files.maxPartitionBytes = 128M(默认)
   // 代表着 RDD 的一个分区能存放的最大字节数
   ```

   当一个 Spark 应用程序执行时，生成 sparkContext，同时会生成两个参数，由上面得到的 spark.default.parallelism 推导出这两个参数的值：

   ```scala
   sc.defaultParallelism = spark.default.parallelism
   sc.defaultMinPartitions = min(spark.default.parallelism, 2)
   ```

   ① 通过 Scala 集合方式 parallelize 生成的 RDD

   ```scala
   val rdd = sc.parallelize(1 to 10)
   // 如果没有指定分区数，则有：
   // rdd 的分区数 = sc.defaultParallelism
   ```

   ② 在本地文件系统通过 textFile 方式生成的 RDD

   ```scala
   val rdd = sc.textFile("path/file")
   // rdd 的分区数 = max(本地 file 的分片数, sc.defaultMinPartitions)
   ```

   ③ 在 HDFS 文件系统生成的 RDD

   rdd 的分区数 = max (HDFS 文件的 Block 数目，sc.defaultMinPartitions)

   ④ 从 HBase 数据表获取数据并转换为 RDD

   rdd 的分区数 = Table 的 region 个数

   ⑤ 通过获取 json（或者 parquet 等等）文件转换成的 DataFrame

   rdd 的分区数 = 该文件在文件系统中存放的 Block 数目

   ⑥ Spark Streaming 获取 Kafka 消息对应的分区数

   * 基于 Receiver：

     在 Receiver 的方式中，Spark 中的 partition 和 kafka 中的 partition 并不是相关的，所以如果加大每个 topic 的 partition 数量，仅仅是增加线程来处理由单一 Recevier 消费的主题，但是并没有增加 Spark 在处理数据上的并行度

   * 基于 DirectDStream

     Spark 会创建跟 Kafka partition 一样多的 RDD partition，并且会并行从 Kafka 中读取数据，所以在 Kafka partition 和 RDD partition 之间，有一个一对一的映射关系

2. reduce 端 task 个数的确定

   Reduce 端进行数据的聚合，一部分聚合算子可以手动指定 reducetask 的并行度，如果没有指定，**则以 map 端的最后一个 RDD 的分区数作为其分区数，那么分区数就决定了 reduce 端的 task 的个数**

##### reduce 端数据的读取

#### HashShuffle 解析

##### 未优化的 HashShuffleManager

shuffle write 阶段，将每个 task 处理的数据对**相同的 key 执行 hash 算法，从而将相同 key 都写入同一个磁盘文件中，而每一个磁盘文件都只属于下游 Stage 的一个 task。**在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去

**下一个 Stage 的 task 有多少个，当前 stage 的每个 task 就要创建多少份磁盘文件**

shuffle read 阶段，**该 stage 的每一个 task 就需要将上一个 stage 的计算结果中的所有相同 key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行 key 的聚合或连接操作。**由于 shuffle write 的过程中，map task 给下游 stage 的每个 reduce task 都创建了一个磁盘文件，因此 shuffle read 的过程中，每个 reduce task 只要从上游 stage 的所有 map task 所在节点上，拉取属于自己的那一个磁盘文件即可。

##### 优化后的 HashShuffleManager

spark.shuffle.consolidateFiles，该参数默认值为 false，将其设置为 true 即可开启优化机制

shuffle write 阶段，task 就不是为下游 stage 的 每个 task 创建一个磁盘文件了，此时会出现 **shuffleFileGroup** 的概念，**每个 shuffleFileGroup 会对应一批磁盘文件，磁盘文件的数量与下游 stage 的 task 数量是相同的。**一个 Executor 上有多少个 CPU core，就可以并行执行多少个 task。**而第一批并行执行的每个 task 都会创建一个 shuffleFileGroup，并将数据写入对应的磁盘文件内**

当 Executor 的 CPU core 执行完一批 task，接着执行下一批 task 时，**下一批 task 就会复用之前已有的 shuffleFileGroup，包括其中的磁盘文件**，也就是说，**此时 task 会将数据写入已有的磁盘文件中，而不会写入新的磁盘文件中。**因此，**consolidate 机制允许不同的 task 复用同一批磁盘文件，这样就可以有效将多个 task 的磁盘文件进行一定程度上的合并，从而大幅减少磁盘文件的数量，进而提升 shuffle write 的性能**



#### SortShuffle 解析

当 shuffle read task 的数量小于等于 spark.shuffle.sort. bypassMergeThreshold 参数的值时（默认为 200），就会启动 bypass 机制

##### 普通运行机制

在该模式下，**数据会先写入一个内存数据结构中，**此时根据不同的 shuffle 算子，可能选用不同的数据结构。如果是 reduceByKey 这种聚合类的 shuffle 算子，那么会选用 Map 数据结构，一边通过 Map 进行聚合，一边写入内存；如果是 join 这种普通的 shuffle 算子，那么会选用 Array 数据结构，直接写入内存。**每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构的数据溢写到磁盘，然后清空内存数据结构**

**在溢写到磁盘文件之前，会先根据 key 对内存数据结构中已有的数据进行排序。排序过后，会分批将数据写入磁盘文件**

**一个 task 将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作， 也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是 merge 过程，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中**。此外，**由于一个 task 就只对应一个磁盘文件，也就意味着该 task 为下游 stage 的 task 准备的数据都在这一个文件中，因此还会单独写一份索引文件，其中标识了下游各个 task 的数据在文件中的 start offset 与 end offset。**

##### bypass 运行机制

触发条件如下：

* shuffle read task 数量小于 spark.shuffle.sort.bypassMergeThreshold 参数的值
* 不是聚合类的 shuffle 算子

此时，**每个 task 会为每个下游 task 都创建一个临时磁盘文件，并将数据按 key 进行 hash 然后根据 key 的 hash 值，将 key 写入对应的磁盘文件之中。**当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

**该过程的磁盘写机制其实跟未经优化的 HashShuffleManager 是一模一样的，因 为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。**

该机制与普通的 SortShuffleManager 运行机制的不同在于：

第一，**磁盘写机制不同**；第二，**不会进行排序**

**启用该机制的最大好处在于，shuffle write 过程中，不需要进行数据的排序操作，也就节省掉了这部分的性能开销。**



### 内存管理

#### 堆内和堆外内存规划

作为一个 JVM 进程，**Executor 的内存管理建立在 JVM 的内存管理上，Spark 对 JVM 的堆内（On-heap）空间进行了更为详细的分配，以充分利用内存**。同时，**Spark 引入了堆外（Off-heap）内存，使之可以直接在工作节点的系统内存中开辟空间，进一步优化了内存的使用**

**堆内内存受到 JVM 统一管理，堆外内存是直接向操作系统进行内存的申请和释放**

##### 堆内内存

堆内内存的大小，由 Spark 应用程序启动时的 – executor-memory 或 spark.executor.memory 参数配置。Executor 内运行的并发任务共享 JVM 堆内内存，**这些任务在缓存 RDD 数据和广播（Broadcast）数据时占用的内存被规划为存储（Storage）内存**，而这**些任务在执行 Shuffle 时占用的内存被规划为执行（Execution）内存，剩余的部分不做特殊规划，那些 Spark 内部的对象实例，或者用户定义的 Spark 应用程序中的对象实例，均占用剩余的空间**。不同的管理模式下，这三部分占用的空间大小各不相同

Spark 对堆内内存的管理是一种逻辑上的**”规划式”**的管理，因为**对象实例占用内存的申请和释放都由 JVM 完成**，**Spark 只能在申请后和释放前记录这些内存**，
我们来看其具体流程：

1. Spark 在代码中 new 一个对象实例
2. JVM 从堆内内存分配空间，创建对象并返回对象引用
3. Spark 保存该对象的引用，记录该对象占用的内存

释放内存流程如下：

1. Spark 记录该对象释放的内存，删除该对象的引用
2. 等待 JVM 的垃圾回收机制释放该对象占用的堆内内存

我们知道，JVM 的对象可以以**序列化的方式存储**，序列化的过程是将对象转换为二进制字节流，本质上可以理解为将非连续空间的链式存储转化为连续空间或块 存储，在**访问时则需要进行序列化的逆过程——反序列化**，将字节流转化为对象，
序列化的方式可以节省存储空间，但增加了存储和读取时候的计算开销。

对于 Spark 中**序列化的对象**，由于是字节流的形式，**其占用的内存大小可直接计算**，而对于**非序列化的对象**，其占用的内存是**通过周期性采样近似估算**而得，即并不是每次新增的数据项都会计算一次占用的内存大小，**这种方法降低了时间开销但是有可能误差较大，导致某一时刻的实际内存有可能远远超出预期。**此外，**在被 Spark 标记为释放的对象实例，很有可能在实际上并没有被 JVM 回收，导致实际可用的内存小于 Spark 记录的可用内存。**所以 **Spark 不能准确记录实际可用的堆内内存，从而也就无法完全避免内存溢出（OOM）的异常**

虽然不能精准控制堆内内存的申请和释放，但 Spark 通过对**存储内存**和**执行内存**各自独立的**规划**管理，可以决定是否要在存储内存里缓存新的 RDD，以及是否为新的任务分配执行内存，在一定程度上可以提升内存的利用率，减少异常的出现

##### 堆外内存

为了**进一步优化内存**的使用以及**提高 Shuffle 时排序的效率**，**Spark 引入了堆外（Off-heap）内存，使之可以直接在工作节点的系统内存中开辟空间，存储经过序列化的二进制数据**

**堆外内存意味着把内存对象分配在 Java 虚拟机的堆以外的内存，这些内存直接受操作系统管理（而不是虚拟机）。这样做的结果就是能保持一个较小的堆，以减少垃圾收集对应用的影响**

利用 JDK Unsafe API（从 Spark 2.0 开始，在管理堆外的存储内存时不再基于 Tachyon，而是与堆外的执行内存一样，基于 JDK Unsafe API 实现），**Spark 可以直接操作系统堆外内存，减少了不必要的内存开销，以及频繁的 GC 扫描和回收，提升了处理性能**。堆外内存可以被精确地申请和释放（堆外内存之所以能够被精确的申请和释放，是由于内存的申请和释放不再通过 JVM 机制，而是直接向操作系统申请，**JVM 对于内存的清理是无法准确指定时间点的，因此无法实现精确的释放**），而且**序列化的数据占用的空间可以被精确计算，所以相比堆内内存来说降低了管理的难度，也降低了误差**

在默认情况下堆外内存并不启用，可通过设置 spark.memory.offHeap.enabled 参数启用，并由 spark.memory.offHeap.size 参数设定堆外空间的大小。**除了没有 other 内存，堆外内存与堆内内存的划分方式相同，所有运行中的并发任务共享存储内存和执行内存**