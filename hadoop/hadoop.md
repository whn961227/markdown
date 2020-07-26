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

**HDFS只是分布式文件管理系统中的一种**

#### 使用场景

HDFS 的使用场景：适合一次写入，多次读出的场景，且不支持文件的修改

#### 优缺点

**优点：**

1. 高容错性
   * 数据自动保存多个副本，通过增加副本的形式，提高容错性
   * 某一个副本丢失以后，它可以自动恢复

2. 适合处理大数据
   * 数据规模：数据量大
   * 文件规模：文件数量大

3. 可构建在廉价机器上，通过多副本机制，提高可靠性

**缺点：**

1. 不适合低延时数据访问，比如毫秒级的存储数据
2. 无法高效的对大量小文件进行存储
   * 存储大量小文件的话，它会占用 NameNode 大量的内存来存储文件目录和块信息
   * 小文件存储的寻址时间会超过读取时间，它违反了 HDFS 的设计目标
3. 不支持并发写入、文件随机修改
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

1. 如果块设置过大，一方面从磁盘传输数据的时间会明显大于寻址时间，导致程序在处理这块数据时，变得非常慢；另一方面，MapReduce 中的 map 任务通常一次只处理一个块中的数据，如果块过大运行速度会变慢，并行度降低
2. 如果设置过小，一方面存放大量小文件会占用 NameNode 大量内存来存储元数据，而 NameNode 的内存是有限的；另一方面，块过小，寻址时间增长，导致程序一直在找 block 的开始位置。

因此块适当设置大一些，减少寻址时间，那么传输一个由多个块组成的文件的时间主要取决于磁盘的传输速度

**设置多大合适呢？**

1. HDFS 中平均寻址时间是 10ms
2. 经过大量测试发现，寻址时间为传输时间的 1%时，为最佳状态，所以最佳传输时间为 1s
3. 目前磁盘的传输速度普遍为 100MB/s，最佳 block 大小为 100MB，所以设置为 128M



### packet & chunk

**packet：**client 向 DataNode，或 DataNode 的 PipeLine 之间传数据的基本单位，默认 64K

**chunk：**chunk 是 client 向 DataNode，或 DataNode 的 PipeLine 之间进行数据校验的基本单位，默认 512 Byte，因为用作校验，所以每个 chunk 需要带有 4 Byte 的校验位，每个 chunk 实际写入 packet 的大小为 516 Byte

* 首先，当数据流进入 DFSOutputStream 时，DFSOutputStream 内会有一个 chunk 大小的 buf，当数据写满这个 buf（或遇到强制 flush），会计算 checkSum 值，然后填塞进 packet
* 当一个 chunk 填塞进 Packet 后，仍然不会立即发送，而是累积到一个 packet 填满后，将这个 packet 放入 DataQueue队列
* 进入 DataQueue 队列的 packet 会被 DataSteamer 线程取出发送到 DataNode

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200726165636702.png" alt="image-20200726165636702" style="zoom: 80%;" />



### 数据块的复制策略

1. 数据的可靠性
2. 数据的写入效率
3. DN 的负载均衡

#### 机架感知（副本存储节点选择）

1. 第一个选择与 Client 最接近的机架上的 DN
2. 第二个选择与第一个 DN 不同机架的 DN
3. 第三个选择与第二个同 一个机架的 DN



### 数据流

#### 写数据流程

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200725225922.png)

1. Client 通过 DistributedFileSystem 向 NameNode 请求上传文件，NameNode 检查目标文件（文件是否存在，Client 是否有权限）。如果检查通过，NameNode 会在 edits 记录操作
2. DistributedFileSystem 返回 FSDataOutputStream 对象给 Client 用于写数据，FSDataOutputStream 封装了 DFSOutputStream 对象负责 Client 和 DataNode 以及 NameNode 之间的通信
3. Client 写数据时，DFSOutputStream 将写入的数据切分成 packets，存到内部的 dataQueue 队列，并且由 DataStreamer 消费处理。DataStreamer 请求 NameNode 分配 DataNode 列表，列表中的 DataNode 会形成 PipeLine，DataStreamer 将 Packet 发给 PipeLine 中的第一个 DN，第一个 DN 将接收到的 Packet 存储完后转发给第二个 DN，第二个 DN 存储完后再发送给第三个 DN ...，直到完成
4. DFSOutputStream 为了防止出问题时数据的丢失，维持了一个等待 DataNode 成功写入的 ACK Queue，只有当 Packet 成功写入 PipeLine 中的每个 DataNode 时，此 Packet 才从 ACK Queue 中移除
5. 当 Client 写完数据，调用 DFSOutputStream 对象的 close() 方法，该操作会将所有剩余 Packets 写到 DataNode PipeLine 并等待返回确认
6. 告知 NameNode 写入文件完成

#### 读数据流程

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200726171348.png)

1. Client 调用 DistributedFileSystem.open() 方法，由 DistributedFileSystem 通过 RPC 向 NameNode 请求返回文件的 Block 块所在的 DataNode 地址，DistributedFileSystem 返回了一个输入流对象 FSDataInputStream，该对象封装了输入流 DFSInputStream
2. 调用 FSDataInputStream.read() 方法从而让 DFSInputStream 连接到 DataNodes
3. 通过循环调用 read() 方法，从而将数据从 DataNode 传输到 Client
4. 当最后一个 Block 返回到 Client 后，DFSInputStream 关闭与 DataNode 连接



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

