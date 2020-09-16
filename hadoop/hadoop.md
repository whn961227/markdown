## Hadoop

### Hadoop 组成

**Hadoop 1.x 和 Hadoop 2.x 区别**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200725212255.png"  />

Hadoop 1.x 中 MapReduce 同时处理 **业务逻辑运算** 和 **资源** 的调度，耦合性较大

Hadoop 2.x 中增加了 Yarn。Yarn 负责 **资源调度**，MapReduce 只负责 **运算**

#### HDFS 架构

* **NameNode：**存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），每个文件的块列表和块所在 DataNode 等
* **DataNode：**在本地文件系统存储文件块数据，以及块数据的校验和
* **SecondaryNameNode：**用来监控 HDFS 状态的辅助后台程序，每隔一段时间获取 HDFS 元数据的快照

#### Yarn 架构

* **ResourceManager：**
  * 处理客户端请求
  * 监控 NodeManager
  * 启动并监控 ApplicationMaster
  * 资源的分配与调度
* **NodeManager：**
  * 管理单个节点上的资源
  * 处理来自 ResourceManager 的命令
  * 处理来自 ApplicationMaster 的命令
* **ApplicationMaster：**
  * 负责数据的切分
  * 为应用程序申请资源并分配给内部的任务
  * 任务的监控与容错
* **Container：**
  * Container 是 Yarn 上的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等

#### MapReduce 架构

MapReduce 将计算过程分为两个阶段：Map 和 Reduce

1. Map 阶段并行处理输入数据
2. Reduce 阶段对 Map 结果进行汇总



## HDFS

### 概述

#### 背景

数据量越来越大，在一个操作系统中存不下所有的数据，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切需要一种系统来管理多台机器上的文件，这就是 **分布式文件管理系统**

**HDFS 只是分布式文件管理系统中的一种**

#### 使用场景

HDFS 的使用场景：适合一次写入，多次读出的场景，且不支持文件的修改

#### 优缺点

**优点：**

1. **高容错性**
   * 数据自动保存多个副本，通过增加副本的形式，提高容错性
   * 某一个副本丢失以后，它可以自动恢复

2. **适合处理大数据**
   * 数据规模：数据量大
   * 文件规模：文件数量大

3. **可构建在廉价机器上，通过多副本机制，提高可靠性**

**缺点：**

1. 不适合**低延时**数据访问，比如毫秒级的存储数据
2. **无法高效的对大量小文件进行存储**
   * 存储大量小文件的话，它会**占用 NameNode 大量的内存**来存储文件目录和块信息
   * 小文件存储的**寻址时间**会超过读取时间，它违反了 HDFS 的设计目标
3. **不支持并发写入**、**文件随机修改**
   * 一个文件只能有一个写，不允许多个线程同时写
   * 仅支持数据 append（追加），不支持文件的随机修改

#### 组成架构

* **Client：**客户端
  * 文件切分。文件上传 HDFS 时，Client 将文件切分成一个一个的 block，然后进行上传
  * 与 NameNode 交互，获取文件的位置信息
  * 与 DataNode 交互，读取或者写入数据
  * Client 通过一些命令来管理 HDFS，比如 NameNode 格式化
  * Client 通过一些命令来访问 HDFS，比如对 HDFS 增删查改操作

* **NameNode：**就是 Master，它是一个主管、管理者
  * 管理 HDFS 的名称空间
  * 配置副本策略
  * 管理数据块（block）映射信息
  * 处理客户端的读写请求
* **DataNode：**就是 Slave，NamNode 下达命令，DataNode 执行实际的操作
  * 存储实际的数据块
  * 执行数据块的读写操作
* **SecondaryNameNode：**并非 NameNode 的热备，当 NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务
  * 辅助 NameNode，分担其工作量，比如定期合并 Fsimage 和 Edits，并推送给 NameNode
  * 在紧急情况下，可辅助恢复 NameNode

#### 文件块大小

HDFS 中的文件在物理上是分块存储（block），默认大小在 Hadoop 2.x 版本中是 128M，老版本中是 64M

**block 不能设置过大，也不能设置过小**

1. 如果块设置**过大**，一方面**从磁盘传输数据的时间会明显大于寻址时间**，导致程序在处理这块数据时，变得非常慢；另一方面，**MapReduce 中的 map 任务通常一次只处理一个块中的数据**，如果块过大**运行速度会变慢**，**并行度降低**
2. 如果设置**过小**，一方面存放大量小文件会**占用 NameNode 大量内存**来存储元数据，而 NameNode 的内存是有限的；另一方面，块过小，**寻址时间增长**，导致程序一直在找 block 的开始位置。

因此块适当设置大一些，减少寻址时间，那么传输一个由多个块组成的文件的时间主要取决于**磁盘的传输速度**

**设置多大合适呢？**

1. HDFS 中平均寻址时间是 10ms
2. 经过大量测试发现，寻址时间为传输时间的 1%时，为最佳状态，所以最佳传输时间为 1s
3. 目前磁盘的传输速度普遍为 100MB/s，最佳 block 大小为 100MB，所以设置为 128M



### packet & chunk

**packet：** **client 向 DataNode**，或 **DataNode 的 PipeLine 之间传数据的基本单位**，默认 64K

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200727212412960.png" alt="image-20200727212412960" style="zoom: 80%;" />

header 中包含了一些元信息，包括这个 Packet 是不是所属 block 的最后一个 packet，数据长度，编码，packet 的数据部分的第一个字节在 block 中的 offset，DataNode 接到这个 Packet 是否必须 sync 磁盘

**chunk：** chunk 是 **client 向 DataNode**，或 **DataNode 的 PipeLine 之间进行数据校验的基本单位**，默认 512 Byte，因为用作校验，所以每个 chunk 需要带有 4 Byte 的校验位，每个 chunk 实际写入 packet 的大小为 516 Byte

* 首先，当数据流进入 DFSOutputStream 时，DFSOutputStream 内会有一个 chunk 大小的 buf，当数据写满这个 buf（或遇到强制 flush），会计算 checkSum 值，然后填塞进 packet
* 当一个 chunk 填塞进 Packet 后，仍然不会立即发送，而是累积到一个 packet 填满后，将这个 packet 放入 DataQueue 队列
* 进入 DataQueue 队列的 packet 会被 DataSteamer 线程取出发送到 DataNode

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200726165636702.png" alt="image-20200726165636702" style="zoom: 80%;" />



### 数据块的复制策略

1. 数据的可靠性
2. 数据的写入效率
3. DN 的负载均衡

#### 机架感知（副本存储节点选择）

1. 第一个选择与 Client 最接近的机架上的 DN
2. 第二个选择与第一个 DN 不同机架的 DN
3. 第三个选择与第二个同一个机架的 DN



### 数据流

#### 写数据流程

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200725225922.png)

1. 客户端通过 **Distributed FileSystem** 模块向 NM 请求上传文件，NN 检查目标文件是否存在，父目录是否存在
2. NN 返回是否可以上传
3. 客户端请求第一个 block 上传到哪几个 DN 上
4. NN 返回 DN 节点，分别为 DN1，DN2，DN3
5. 客户端通过 **FSDataOutputStream** 模块请求 DN1 建立传输通道，DN1 收到请求后会继续调用 DN2，DN2 调用DN3，将这个通信管道建立完成
6. DN1，DN2，DN3 逐级应答客户端
7. 客户端开始往 DN1 上传第一个 block（先从磁盘读取数据放到一个本地内存缓存），以 packet 为单位，DN1 收到一个 packet 就会传给 DN2，DN2 传给 DN3；DN1 每传一个 packet 会放入一个应答队列等待应答
8. 当一个 block 传输完成后，客户端再次请求 NN 上传第二个 block 的 DN

#### 读数据流程

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200726171348.png)

1. 客户端通过 **Distributed FileSystem** 向 NN 请求下载文件，NN 通过查询元数据，找到文件块所在的 DN 地址
2. 选择一台 DN （就近原则，然后随机）服务器，请求读取数据
3. DN 开始传输数据给客户端（从磁盘里面读取数据输入流，以 packet 为单位来做校验）
4. 客户端以 packet 为单位接收，先在本地缓存，然后写入目标文件



### NN 内存全景

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200730145420.png" style="zoom: 25%;" />

**NameSpace：**维护整个文件系统的目录结构及目录树上的状态变化。除在内存常驻外，这部分数据会定期 flush 到持久化设备上，生成一个新的 FsImage 文件，方便 NameNode 发生重启时，从 FsImage 及时恢复整个 NameSpace

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200730151053.png" style="zoom:25%;" />

整个 NameSpace 目录树中存在两种不同类型的 INode 数据结构：INodeDirectory 和 INodeFile。其中 INodeDirectory 标识的是目录树中的目录，INodeFile 标识的是目录树中的文件

**BlockManager：**维护整个文件系统中与数据块相关的信息及数据块的状态变化，BlocksMap 在 NameNode 内存空间占据很大比例，由 BlockManager 统一管理。NameSpace 与 BlockManager 之间通过 INodeFile 有序 Blocks 数组关联到一起



### NN 和 2NN

NameNode 元数据存储在内存中，为了解决易丢失，因此产生在磁盘中备份元数据的 **FsImage**；更新元数据的同时更新 FsImage，为了解决效率过低，因此，引入了 Edits 文件（只进行追加操作，效率很高），每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到 Edits 中，可以通过 FsImage 和 Edits 的合并，合成元数据

如果长时间添加数据到 Edits，会导致该文件过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，引入 2NN，定期进行 FsImage 和 Edits 的合并

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200727211447323.png" alt="image-20200727211447323"  />



### DataNode 工作机制

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200727212820184.png" alt="image-20200727212820184" style="zoom: 80%;" />



### 小文件存档

**HDFS 存储小文件弊端**

每个文件均按块存储，每个块的元数据存储在 NameNode 的内存中，因此 HDFS 存储小文件会非常低效。因为大量的小文件会耗尽 NameNode 中的大部分内存。

**解决存储小文件办法之一**

HDFS 存档文件或 HAR 文件，是一个更高效的文件存档工具，它将文件存入 HDFS 块，在减少 NameNode 内存使用的同时，允许对文件进行透明的访问。具体来说，HDFS 存档文件对内还是一个个独立文件，对 NameNode 而言却是一个整体，减少了 NameNode 的内存

```shell
# 通过 archive 工具存档
hadoop archive -archiveName files.har /my/files /my
# 第一个选项是存档文件的名称，这里是第一个参数 file.har
# 第二个参数是需要归档的文件
# 第三个参数是 Har 文件的输出目录
```



### 如何扩展 HDFS 的存储容量

* 增加 DN 节点数
* 单节点 DN 挂载新的磁盘



### Shell 操作

```shell
# 检查 HDFS 上文件和目录的健康状态、获取文件的 block 块信息和位置信息等
# -move: 移动损坏的文件到/lost+found目录下
# -delete: 删除损坏的文件
# -openforwrite: 输出检测中的正在被写的文件
# -list-corruptfileblocks: 输出损坏的块及其所属的文件
# -files: 输出正在被检测的文件
# -blocks: 输出block的详细报告 （需要和-files参数一起使用）
# -locations: 输出block的位置信息 （需要和-files参数一起使用）
# -racks: 输出文件块位置所在的机架信息（需要和-files参数一起使用）
hdfs fsck /

# 手动修复缺失或损坏的 block 副本
hdfs debug  recoverLease  -path /blockrecover/genome-scores.csv -retries 10

# 自动修复
# 当数据块损坏后，DN 节点执行 directoryscan 操作之前，都不会发现损坏
# directoryscan 间隔 6h
dfs.datanode.directoryscan.interval : 21600
# 在 DN 向 NN 进行 blockreport 之前，都不会恢复数据块
# blockreport 间隔 6h
dfs.blockreport.intervalMsec : 21600000
# 当 NN 收到 blockreport 才会进行数据恢复操作

# block 默认大小 128M
dfs.block.size : 134217728
```



### Hadoop 数据压缩

#### 概述

压缩技术能够有效减少 HDFS 读写字节数。压缩提高了网络带宽和磁盘空间的效率。在运行 MR 程序时，I/O 操作，网络数据传输，Shuffle 和 Merge 要花大量的时间，尤其是**数据规模很大和工作负载密集**的情况下，因此，**使用数据压缩显得非常重要**

鉴于磁盘 I/O 和网络带宽是 Hadoop 的宝贵资源，**数据压缩对于节省资源，最小化磁盘 I/O 和网络传输非常有帮助**，**可以在任意 MR 阶段启用压缩**

压缩是提高 Hadoop 运行效率的一种**优化策略**

**通过对 Mapper、Reducer 运行过程的数据进行压缩，以减少磁盘 IO**，提高 MR 程序运行速度

*注意：采用压缩技术减少了磁盘 IO，但同时增加了 CPU 运算负担。所以，压缩特性运用得当能提高性能，但运用不当也可能降低性能*

**压缩基本原则：**

1. **运算密集型的 job，少用压缩**
2. **IO 密集型的 job，多用压缩**

#### 压缩方式的选择

> **Gzip 压缩**

优点：压缩率比较高，速度较快；Hadoop 本身支持

缺点：不支持 split

应用场景：**当每个文件压缩后在 1 个块大小内，都可以考虑用 Gzip 压缩格式**

> **Bzip2 压缩**

优点：支持 split；具有很高的压缩率；Hadoop 本身自带

缺点：速度慢

应用场景：**适合对速度要求不高，但需要较高的压缩率的时候**；或者**输出之后的数据较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用的比较少的情况**；

> **Lzo 压缩**

优点：速度比较快，合理的压缩率；支持 split，是 Hadoop 中最流行的压缩格式

缺点：压缩率比 Gzip 要低一些；Hadoop 本身不支持，需要安装；为了支持 split 需要建索引，还需要指定 inputformat 为 Lzo 格式

> **Snappy 压缩**

优点：高速压缩速度和合理的压缩率

缺点：不支持 Split；压缩率比 Gzip 要低；Hadoop 本身不支持，需要安装

### HDFS HA

HDFS HA 功能通过配置 **Active/Standby 两个 NameNodes** 实现在集群中对 NameNode 的热备来解决单点故障问题，其中**只有 Active NameNode 对外提供读写服务**，Standby NameNode 会根据 Active NameNode 的状态变化，在必要时切换成 Active 状态

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200728115232.png" style="zoom: 33%;" />

**ZKFC：**ZKFC 即 ZKFailoverController，作为独立进程存在，负责**控制 NameNode 的主备切换**，ZKFC 会监测 NameNode 的健康状况，当发现 Active NameNode 出现异常时会**通过 Zookeeper 集群进行一次主备选举**，完成 Active 和 Standby 状态的切换

**JournalNode 集群：** **共享存储系统**，负责**存储 HDFS 的元数据*Edits文件***，**Active NameNode（写入）和 Standby NameNode（读取）通过共享存储系统实现元数据同步**，在主备切换过程中，新的 Active NameNode 必须确保元数据同步完成才能对外提供服务

**Zookeeper 集群：**为 ZKFC 提供**主备选举**支持

**DataNode 节点：**除了通过 JournalNode 共享 HDFS 的元数据信息之外，主 NameNode 和备 NameNode 还需要共享 HDFS 的数据块和 DataNode 之间的映射关系。**DataNode 会同时向主 NameNode 和备 NameNode 上报数据块的位置信息**

####  主备切换实现

[主备切换实现]: https://developer.ibm.com/zh/articles/os-cn-hadoop-name-node/

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200728105751.png" style="zoom: 25%;" />

**ZKFailoverController **在启动的时候会创建 HealthMonitor 和 ActiveStandbyElector ，在创建的同时也会向它们注册相应的回调方法

**HealthMonitor：**监控 NameNode 的健康状态，如果检测到 NameNode 状态发生变化，会回调 ZKFC 的相应方法进行自动的主备选举

**ActiveStandbyElector：**接收 ZKFC 的选举请求，通过 Zookeeper 自动完成主备选举，选举完成后回调 ZKFC 的主备切换方法对 NameNode 进行 Active 和 Standby 状态的切换

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200728142654.png" style="zoom:33%;" />

1. HealthMonitor 初始化完成之后会启动内部的线程来定时调用对应 NameNode 的 HAServiceProtocol RPC 接口的方法，对 NameNode 的健康状态进行检测
2. HealthMonitor 如果检测到 NameNode 的健康状态发生变化，会回调 ZKFC 注册的相应方法进行处理
3. 如果 ZKFC 判断需要进行主备切换，会首先使用 ActiveStandbyElector 来进行自动的主备选举
4. ActiveStandbyElector 与 Zookeeper 进行交互完成自动的主备选举
5. ActiveStandbyElector 在主备选举完成后，会回调 ZKFC 的相应方法来通知当前的 NameNode 成为主 NameNode 或者备 NameNode
6. ZKFC 调用对应 NameNode 的 HAServiceProtocol RPC 接口的方法将 NameNode 转换为 Active 状态或 Standby 状态



## MapReduce

### 定义

MapReduce 是一个分布式运算程序的编程框架，核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上

**优点**

* **MapReduce 易于编程**
* **良好的扩展性**
* **高容错性**
* **适合 PB 级以上海量数据的离线处理**

**缺点**

* **不擅长实时计算**
* **不擅长流式计算：**输入数据集是静态的
* **不擅长 DAG（有向图）计算：**多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，每个 MapReduce 作业的输出结果都会写入到磁盘，会造成大量的磁盘 IO，导致性能非常的低下

### WordCount 案例实操

```java
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

	Text k = new Text();
	IntWritable v = new IntWritable(1);

	@Override
	protected void map(LongWritable key, Text value, Context context) throws Exception {
		String line = value.toString();
		String[] words = line.toSplit(" ");
		for(String word : Words) {
			k.set(word);
			context.write(k, v);
		}
	}
}

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
	
	IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws Exception {
		int sum = 0;
		for(IntWritable value : values) {
			sum += value.get();
		}
		v.set(sum);
		context.write(key, v);
	}
}

public class WordCountDriver {
	public static void main(String[] args) throws Exception {
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		// 设置 jar
		job.setJarByClass(WordCountDriver.class);
		// 设置 Mapper 类和 Reducer 类
		job.setMapperClass(WordCountMapper.class);
		job.setReducerClass(WordCountReducer.class);
		// 设置 Mapper key value 输出类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		// 设置 Reducer key value 输出类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		// 设置输入输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPaths(job, new Path(args[1]));

		boolean res = job.waitForCompletion(true);
		System.exit(res ? 0 : 1);
	}
}
```



### Hadoop 序列化

* **序列化：**将内存中的对象，转换成字节序列以便于存储到磁盘（持久化）和网络传输
* **反序列化：**将收到字节序列（或其他数据传输协议）或者是磁盘的持久化数据，转换成内存中的对象

**为什么不用 Java 的序列化**

 Java 的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以，Hadoop 自己开发了一套序列化体制（Writable）

**Hadoop 序列化特点：**

1. 紧凑
2. 快速
3. 可扩展
4. 互操作

**常用序列化类型**

| Java 类型 | Hadoop Writable 类型 |
| --------- | -------------------- |
| boolean   | BooleanWritable      |
| byte      | ByteWritable         |
| int       | IntWritable          |
| float     | FloatWritable        |
| long      | LongWritable         |
| double    | DoubleWritable       |
| String    | Text                 |
| map       | MapWritable          |
| array     | ArrayWritable        |

**自定义 bean 对象实现序列化接口（Writable）**

1. 必须实现 Writable 接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
3. 重写序列化方法
4. 重写反序列化方法
5. 如果需要将自定义的 bean 放在 key 中传输，还需要实现 Comparable 接口，因为 MR 的 shuffle 过程中要求 key 必须能排序



### 框架原理

#### InputFormat 数据输入

##### 切片与 MapTask 并行度决定机制

**数据切片：**数据切片只是在**逻辑上对输入进行分片**，并不会在磁盘上将其切分成片进行存储

1. 一个 Job 的 Map 阶段并行度由客户端在提交 Job 时的切片数决定
2. 每一个 Split 切片分配一个 MapTask 并行实例处理
3. 默认情况下，切片大小 = BlockSize
4. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

##### FileInputFormat 切片机制

1. 简单地按照文件的内容长度进行切分
2. 切片大小，默认等于 Block 大小（每次切片时判断切完剩下的部分是否大于块的 1.1 倍，不大于 1.1 倍就划分一块切片）
3. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切分

**缺点：**不管文件多小，都会是一个单独的切片，都会交给一个 MapTask，如果有大量的小文件，就会产生大量的 MapTask，处理效率极其低下

##### CombineTextInputFormat 切片机制

**应用场景：**用于小文件过多的场景，可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就会交给一个 MapTask 处理

**切片机制：**生成切片过程包括：**虚拟存储过程** 和 **切片过程**

**设置虚拟存储切片最大值：**setMaxInputSplitSize

* 虚拟存储过程
  *  将输入目录下所有文件大小，依次和设置的虚拟存储切片最大值比较，如果不大于设置的最大值，逻辑上划分一个块；如果输入文件大于设置的最大值且大于两倍，那么以最大值切分一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时将文件均分为 2 个虚拟存储块（防止出现太小切片）

* 切片过程
  * 判断虚拟存储的文件大小是否大于虚拟存储切片最大值，大于等于则单独形成一个切片；如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片

##### FileInputFormat 实现类

* TextInputFormat

  默认的 FileInputFormat 实现类。按行读取每条记录。Key 是存储该行在整个文件中的起始字节偏移量，LongWritable 类型；值是这行的内容，不包括任何行终止符（换行符和回车符），Text 类型

* KeyValueTextInputFormat

  每一行均为一条记录，被分隔符分割为 key，value。

* NLineInputFormat

  每个 map 进程处理的 InputSplit 不再按 Block 块去划分，而是按 NLineInputFormat 指定的行数 N 来划分

* 自定义 InputFormat
  * 自定义一个类继承 FileInputFormat
  * 改写 RecordReader，实现一次读取一个完整文件封装 KV
  * 在输出时使用 SequenceFileOutputFormat 输出合并文件

#### shuffle 机制

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200729113051.png)

1. Map 方法之后，Reduce 方法之前这段处理过程叫做 shuffle
2. Map 方法之后，**数据首先进入分区方法，把数据标记好分区**，然后**把数据发送到环形缓冲区**；环形缓冲区默认大小为 100M，**环形缓冲区达到阈值（80%）时，进行溢写**；**溢写前对数据进行排序**，排序按照对 key 的索引进行字典顺序排序，排序的手段`快排`；溢写产生大量溢写文件，需要**对溢写文件进行归并排序**；*对溢写的文件也可以进行 Combiner 操作*，前提是汇总操作，求平均值不行。最后**将文件按照分区存储到磁盘**，等待 Reduce 端拉取
3. **每个 Reduce 拉取 Map 端对应分区的数据**。拉取数据后**先存储到内存中**，内存不够了，**再存储到磁盘**。拉取完所有数据后，**采用归并排序将内存和磁盘中的数据都进行排序**。在进入 Reduce 方法前，*可以对数据进行分组操作*



##### Partition 分区

将统计结果按照条件输出到不同文件中（分区）

**默认 Partition 分区**

```java
// 默认分区是根据 key 的 hashCode 对 ReduceTasks 个数取模得到的
public class HashPartitioner<K, V> extends Partitioner<K, V> {
  public int getPartition(K key, V value, int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
  }
}
```

**自定义 Partitioner 分区**

1. 自定义类继承 Partitioner，重写 getPartition() 方法

2. 在 Job 驱动中，设置自定义 Partitioner

   ```java
   job.setPartitionerClass();
   ```

3. 自定义 Partition 后，要根据自定义 Partitioner 的逻辑设置相应数量的 Reduce Task

   ```java
   job.setNumReduceTasks();
   ```

**分区总结：**

1. 如果 ReduceTask 的数量 > getPartition 的结果数，则会多产生几个空的输入文件 part-r-000xx
2. 如果 1 < ReduceTask 的数量 < getPartition 的结果数，则有一部分分区数据无处安放，会 Exception
3. 如果 ReduceTask 的数量 = 1，则不管 MapTask 端输出多少个分区文件，最终结果都交给一个 ReduceTask，最终也就只会产生一个结果文件 part-r-00000
4. 分区号必须从 0 开始，逐一累加

##### Combiner 合并

1. Combiner 是 MR 程序中 Mapper 和 Reducer 之外的一种组件
2. Combiner 组件的父类是 Reducer
3. Combiner 和 Reducer 的区别在运行位置：
   * Combiner 是在每一个 MapTask 所在节点运行
   * Reducer 是接收全局所有 Mapper 的输出结果
4. Combiner 的意义就是对每一个 MapTask 的输出进行局部汇总，减少网络传输量
5. Combiner 能够应用的前提是不能影响最终的业务逻辑

#### Map Task 机制

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200912184024.png)

1. Read 阶段：MapTask 通过用户编写的 RecordReader，从输入 InputSplit 中解析出一个个 key/value

2. Map 阶段：该节点主要是将解析出的 key/value 交给用户编写 map() 函数处理，并产生一系列新的 key/value

3. Collect 收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用 OutputCollector.collect() 输出结果。在该函数内部，它会**将生成的 key/value 分区（调用 Partitioner ）**，并写入一个环形内存缓冲区中。

4. Spill 阶段：即“溢写”，当环形缓冲区满后，MapReduce 会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

   溢写阶段详情：

   1. 利用**快速排序**算法对缓存区内的数据进行排序，排序方式是，**先按照分区编号 Partition 进行排序，然后按照key进行排序**。这样，经过排序后，**数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序**。
   2. 按照分区编号由小到大依次将**每个分区中的数据写入**任务工作目录下的**临时文件** output/spillN.out（ N 表示当前溢写次数）中。**如果用户设置了 Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作**。
   3. **将分区数据的元信息写到内存索引数据结构 SpillRecord 中**，其中每个分区的元信息包括在临时文件中的**偏移量**、**压缩前数据大小**和**压缩后数据大小**。如果当前内存索引大小超过 1MB，则将内存索引写到文件 output/spillN.out.index 中。

5. Combine 阶段：**当所有数据处理完成后，MapTask 对所有临时文件进行一次合并，以确保最终只会生成一个数据文件**。



#### Reduce Task 机制

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200912213510.png)

1. Copy 阶段：ReduceTask **从各个 MapTask 上远程拷贝一片数据**，并针对某一片数据，**如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中**
2. Merge 阶段：在远程拷贝数据的同时，**ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并**，以防止内存使用过多或磁盘上文件过多
3. Sort 阶段：按照 MapReduce 语义，用户编写 reduce() 函数输入数据是按 key 进行聚集的一组数据。为了将key 相同的数据聚在一起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了局部排序，因此，ReduceTask 只需对所有数据进行一次**归并排序**即可
4. Reduce 阶段：reduce() 函数将计算结果写到 HDFS 上



#### OutputFormat 数据输出

##### OutputFormat 接口实现类

* **文本输出 TextOutputFormat：**默认的输出格式是 TextOutputFormat，它把每条记录写为文本行
* **SequenceFileOutputFormat：**将 SequenceFileOutputFormat 输出作为后续 MapReduce 任务的输入，因为它的格式紧凑，很容易被压缩
* **自定义 OutputFormat**

#### Join 多种应用

##### Reduce Join

* **Map 端主要工作：**为来自不同表或文件的 key/value 对，打标签以区别不同来源的记录。然后用连接字段作为 key，其余部分和新加的标志作为 value，最后进行输出
* **Reduce 端主要工作：**在 Reduce 端以连接字段作为 key 的分组已经完成，只需要在每个分组当中将那些来源于不同文件的记录（在 Map 阶段已经打标签）分开，最后再进行合并

**缺点：**这种方式，合并的操作是在 Reduce 阶段完成，Reduce 阶段处理压力太大，Map 节点的运算负载很低，资源利用率不高，且在 Reduce 阶段极易产生数据倾斜

**解决方案：**Map 端实现数据合并

##### Map Join

**适用场景：**适用于一张表十分小，一张表很大的场景

在 **Map 端缓存多张表**，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能减少数据倾斜

**具体方法：DistributedCache**

1. 在 Mapper 的 setup 阶段，将文件读取到缓存集合中

2. 在驱动函数中加载缓存

   ```java
   job.addCacheFile(new URI(filePath));
   ```

## Yarn

### 基本架构

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200729161857.png" style="zoom: 25%;" />

* ResourceManager

  1. 处理 Client 请求
  2. 监控 NodeManager
  3. 启动或监控 ApplicationMaster
  4. 资源的分配与调度

* NodeManager

  1. 管理单个节点上的资源
  2. 处理来自 ResourceManager 的命令
  3. 处理来自 ApplicationMaster 的命令

* ApplicationMaster

  1. 负责数据的切分
  2. 为 Application 申请资源并分配给内部的任务
  3. 任务的监控与容错

* Container

  Container 是资源的抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等



### 工作机制

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200729162309.png" style="zoom: 50%;" />

* 工作提交
  1. Client 调用 job.waitForCompletion 方法，向整个集群提交 MR 作业
  2. Client 向 RM 申请一个作业 id
  3. RM 向 Client 返回该 job 资源的提交路径和作业 id
  4. Client 提交 jar 包，切片信息和配置文件到指定的资源提交路径
  5. Client 提交完资源后，向 RM 申请运行 MrAppMaster
  
* 作业初始化
  6. 当 RM 收到 Client 的请求后，将该 job 添加到容量调度器中
  7. 某一个空闲的 NM 领取到该 job
  8. 该 NM 创建 Container，并产生 MRAppMaster
  9. 下载 Client 提交的资源到本地
  
* 任务分配
  10. MrAppMaster 向 RM 申请运行多个 MapTask 任务资源
  11. RM 将运行 MapTask 任务分配给另外两个 NM，另外两个 NM 分别领取任务并创建容器
  
* 任务运行

  12. MR 向两个接收到任务的 NM 发送程序启动脚本，这两个 NM 分别启动 MapTask，MapTask 对数据分区排序
  13. MrAppMaster 等待所有 MapTask 运行完毕后，向 RM 申请容器，运行 ReduceTask
  14. ReduceTask 向 MapTask 获取相应分区的数据
  15. 程序运行完毕后，MR 会向 RM 申请注销自己

* 进度和状态更新

  Yarn 中的任务将其进度和状态（包括 counter）返回给应用管理器，客户端每秒（通过 mapreduce.client.progressmonitor.pollinerval 设置）向应用管理器请求进度更新，展示给用户

* 作业完成

  除了向应用管理器请求作业进度外，客户端每 5 秒都会通过调用 waitForCompletion() 来检查作业是否完成。时间间隔可以通过 mapreduce.client.completion.pollinterval 来设置。作业完成之后，应用管理器和 Container 会清理工作状态。作业的信息会被作为历史服务器存储以备之后用户核查



### 资源调度器

* **先进先出调度器（FIFO）**

  ![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914114212.png)

  先进先出，同一时间队列中只有一个任务在执行

* **容量调度器（Capacity Scheduler）**

  ![](https://raw.githubusercontent.com/whn961227/images/master/data/20200912182534.png)

  1. 多队列；每个队列内部先进先出，同一时间队列中只有一个任务在执行。**队列的并行度为队列的个数**
  2. 首先，计算每个队列中正在运行的任务数与其应该分得的计算资源之间的比值，选择一个该比值最小的队列——**最闲**的，其次，按照作业优先级和提交时间顺序，同时考虑用户资源量限制和内存限制**对队列内任务排序**

* **公平调度器（Fair Scheduler）**

  ![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914114727.png)

  1. 多队列；每个队列内部按照**缺额**大小分配资源启动任务，同一个时间队列中有多个任务执行。队列的并行度大于等于队列的个数
  2. **每个队列中的 job 按照优先级分配资源，优先级越高分配的资源越多**，但是每个 job 都会分配到资源以确保公平
  3. 同一队列中，**job 的资源缺额越大，越先获得资源优先执行**，作业也是按照缺额的高低来先后执行的

## Hadoop RPC 机制

### RPC 通信模型

RPC 是一种提供网络从远程计算机上请求服务，但不需要了解底层网络技术的协议

RPC 通常采用 **客户机/服务器** 模型。请求程序是一个客户机，而服务提供程序是一个服务器。一个典型的 RPC 框架。主要包括以下几个部分：

* **通信模块**

  两个相互协作的通信模块实现请求-应答协议，它们在客户和服务器之间传递请求和应答消息，一般不会对数据包进行任何处理。

* **Stub 程序**

  客户端和服务器端均包含 Stub 程序，可以将之看做代理程序。它使得远程函数调用表现的跟本地调用一样，对用户程序完全透明。

  在客户端，Stub 程序像一个本地程序，但不直接执行本地调用，而是将请求信息提供网络模块发送给服务器端，服务器端给客户端发送应答后，客户端 Stub 程序会解码对应的结果。

  在服务器端，Stub 程序依次进行**解码**请求消息中的参数、**调用**相应的服务过程和**编码**应答结果的返回值等处理

* **调度程序**

* **客户程序/服务过程**

**一个 RPC 请求从发送到获取处理结果，所经历的步骤：**

1. 客户程序以本地方式调用系统产生的 Stub 程序
2. 该 Stub 程序将函数调用信息按照网络通信模块的要求封装成消息包，并交给通信模块发送给远程服务器端
3. 远程服务器端接收此消息后，将此消息发送给相应的 Stub 程序
4. Stub 程序拆封消息，形成被调用过程要求的形式，并调用对应函数
5. 被调用函数按照所获参数执行，并将结果返回给 Stub 程序
6. Stub 程序将此结果封装成消息，通过网络通信模块逐级地传送给客户程序



### RPC 总体架构

Hadoop RPC 主要分为四部分：

* **序列化层：**序列化主要作用是将结构化对象转为字节流以便于通过网络进行传输或写入持久存储，在 RPC 框架中，它主要是用于将**用户请求中的参数**或者**应答**转换成**字节流**以便跨机器传输
* **函数调用层：**主要功能是定位要调用的函数并执行该函数，Hadoop RPC 采用了 Java 反射机制与动态代理实现了函数调用
* **网络传输层：**描述了 Client 和 Server 之间消息传输的方式，Hadoop  RPC 采用了基础 TCP/IP 的 Socket 机制
* **服务器端处理框架：**



### Hadoop RPC 的使用方法

**主要分为以下 4 个步骤**

```java
// 定义 RPC 协议
// RPC 协议是客户端和服务器端之间的通信接口，它定义了服务器端对外提供的服务接口
public interface MyBizable extends VersionedProtocol {
    public static final long versionID = 2345234L;
    public abstract String hello(String name);
}

// 实现 RPC 协议
public class MyBiz implements MyBizable {
    public String hello(String name) {
        System.out.println("我被调用了");
        return "hello " + name;
    }

    public long getProtocolVersion(String protocol, long clientVersion) throws IOException {
        return versionID;
    }

    public ProtocolSignature getProtocolSignature(String protocol, long clientVersion, int clientMethodsHash) throws IOException {
        return null;
    }
}

// 构造并启动 RPC Server
public class MyServer {
    public static final int SERVER_PORT = 12345;
    public static final String SERVER_ADDRESS = "localhost";

    public static void main(String[] args) throws IOException {
        final RPC.Server server = new RPC.Builder(new Configuration())
                .setProtocol(MyBizable.class)
                .setInstance(new MyBiz())
                .setBindAddress(SERVER_ADDRESS)
                .setPort(SERVER_PORT).build();
        server.start();
    }
}

// 构造 RPC Client 并发送 RPC 请求
// 使用静态方法 getProxy 构造客户端代理对象
public class MyClient {
    public static void main(String[] args) throws IOException {
        final MyBizable proxy = RPC.getProxy(MyBizable.class
                , MyBiz.versionID
                , new InetSocketAddress(MyServer.SERVER_ADDRESS, MyServer.SERVER_PORT)
                , new Configuration());

        final String result = proxy.hello("world");
        System.out.println(result);
        RPC.stopProxy(proxy);
    }
}
```



## Hadoop 企业优化

### HDFS 优化

* HDFS 小文件影响
  1. **影响 NN 的寿命，因为文件元数据存储在 NN 的内存中**
     * 当 NN 重新启动时，必须**从本地磁盘上的缓存中读取每个文件的元数据**，这意味着要从磁盘中读取大量的数据，会导致**启动时间延迟**
     * NN 必须不断**跟踪并检查集群中每个数据块的存储位置**。这是通过**监听数据节点**来报告其所有数据块来完成的。数据节点必须报告的块越多，它将**消耗的网络带宽**就越多。
  2. **影响计算引擎的任务数量，比如每个小的文件都会生成一个 Map 任务**
     * 大量的小文件意味**大量的随机磁盘 IO**，一次大的顺序读取总是胜过几次随机读取相同数量的数据
     * **一个文件启动一个 map**，小文件越多，map 也越多，一个 map 启动一个 JVM 去执行，这些**任务的初始化、启动、执行会浪费大量的资源**，严重影响性能
* 数据输入小文件处理
  1. 合并小文件：对小文件进行**归档（Har）*Har 文件通过 Hadoop 的 archive 命令创建，实际上运行了 MR 任务来将小文件打包成 Har***、自定义 InputFormat 将小文件存储成 **SequenceFile 文件*用来存储二进制形式的 key-value 设计的一种平面文件* ***
  2. 采用 **CombineFileInputFormat** 来作为输入，解决输入端大量小文件场景
  3. 对于大量小文件 Job，可以开启 **JVM 重用**

* Map 阶段
  1. **增大环形缓冲区大小，溢写的比例**
  2. **减少对溢写文件的 merge 次数**
  3. 在不影响实际业务的前提下，采用 **Combiner** 提前合并，减少 IO
* Reduce 阶段
  1. 合理设置 Map 和 Reduce 数：两个都不能设置太少，也不能设置太多。太少，会导致 Task 等待，延长处理时间；太多，会导致 Map、Reduce 任务间竞争资源，造成处理超时等错误
  2. **设置 Map、Reduce 共存：**调整 slowstart.completedmaps 参数，使 Map 运行到一定程度后，Reduce 也开始运行，减少 Reduce 的等待时间
  3. **规避使用 Reduce：**因为 Reduce 在用于连接数据集时会产生大量的网络消耗
  4. 增加每个 Reduce 去 Map 中取数据的并行数
  5. 集群性能可以的前提下，增大 Reduce 端存储数据内存的大小
* IO 传输
  1. 采用数据压缩的方式，减少网络 IO 的时间。安装 Snappy 和 LZO 压缩编码器
  2. **使用 SequenceFile 二进制文件**

**解决方案：**

1. 在数据采集时，就将小文件或小批数据合并成大文件再上传 HDFS（**Archive**）
2. 在业务处理之前，在 HDFS 上使用 MR 程序对小文件进行合并（**Sequence File**）

   

* 数据倾斜

  1. 提前在 map 进行 combine，减少传输的数据量

     在 Mapper 加上 combiner 相当于提前进行 reduce，即把一个 Mapper 中的相同 key 进行了聚合，减少 shuflle 过程中传输的数据量，以及 Reducer 端的计算量

     * 在不影响业务逻辑的前提下，仅限于汇总操作（累加、最大值等），求平均值就不行
     * 如果导致数据倾斜的 key 大量分布在不同的 mapper 的时候，这种方法就不是很奏效了

  2. 导致数据倾斜的 key 大量分布在不同的 mapper

     * 局部聚合加全部聚合

       第一次在 map 阶段对那些导致了数据倾斜的 key 加上 1 到 n 的随机前缀，这样本来相同的 key 也会被分到多个 Reducer 中进行局部聚合，数量就会大大降低

       第二次 MR，去掉 key 的随机前缀，进行全局聚合

       思想：两次 MR，第一次将 key 随机散列到不同的 reducer 进行处理达到负载均衡目的，第二次再去掉 key 的随机前缀，按原 key 进行 reduce 处理

  3. 实现自定义分区

     根据数据分布情况，自定义散列函数，将 key 均匀分配到不同的 Reducer