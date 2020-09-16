## Kafka

### 概述

#### 消息队列

* 点对点模式（**一对一**，消费者主动拉取数据，消息收到后消息清除）

  消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费以后，queue 中不再有存储，所以消息消费者不可能消费到已经被消费的消息

  <img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820102427.png" style="zoom: 33%;" />

* 发布/订阅模式（**一对多**，消费者消费数据之后不会清除消息）

  消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到 topic 的消息会被所有订阅者消费

  <img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820102528.png" style="zoom:33%;" />

#### 特性

* `高吞吐、低延迟`

  kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒

* `高伸缩性`

  每个主题(topic) 包含多个分区(partition)，主题中的**分区**可以分布在不同的主机(broker)中

* 灵活性 & 峰值处理能力

  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列 能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃

* `容错性`：

  允许集群中的节点失败，某个节点宕机，Kafka 集群能够正常工作

* `高并发`：

  支持数千个客户端同时读写

* 异步通信

  很多时候，用户不想也不需要立即处理消息。消息队列提供了**异步处理机制**，允许用户 把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们

#### 使用场景

* 活动跟踪：Kafka 可以用来跟踪用户行为，将用户行为作为消息传递给 Kafka，这样就可以生成报告，可以做智能推荐，购买喜好等
* 流式处理：流式处理是有一个能够提供多种应用程序的领域
* 限流削峰：Kafka 多用于互联网领域某一时刻请求特别多的情况下，可以把请求写入Kafka 中，避免直接请求后端程序导致服务崩溃

#### Kafka 架构

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820092422.png" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820092453.png" style="zoom: 33%;" />

1. Producer ：消息生产者，使用push模式将消息发布到broker；

2. Consumer ：消息消费者，使用pull模式从broker订阅并消费消息；

3. Topic ：消息的种类称为 `主题`（Topic），可以说一个主题代表了一类消息。相当于是对消息进行分类。可以理解为一个队列，**生产者和消费者面向的都是一个 topic**；

4. Consumer Group （CG）：消费者组，由多个 consumer 组成。**消费者组内每个消费者负责消费不同分区的数据**，**一个分区只能由一个组内消费者消费**；**消费者组之间互不影响**。所有的消费者都属于某个消费者组，即**消费者组是逻辑上的一个订阅者**

   <img src="https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw7haRlQPaFcxDRgI51Mdf27RVrLfsoJbxXmJ4AuXAxzz71RzFqzJKQdkJQu64gkeGh3dF2RVGs1cg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:33%;" />

5. Broker ：一台 kafka 服务器就是一个broker。broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存；

6. Broker 集群：broker 是`集群` 的组成部分，broker 集群由一个或多个 broker 组成，每个集群都有一个 broker 同时充当了`集群控制器`的角色（自动从集群的活跃成员中选举出来）

7. offset：`偏移量`（Consumer Offset）是一种元数据，它是一个不断递增的整数值，**用来记录消费者发生重平衡时的位置**，以便用来恢复数据

8. Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，**一个topic可以分为多个partition**，**每个partition是一个有序的队列**

   <img src="https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw7haRlQPaFcxDRgI51Mdf27xf0Wrln5XU4m6cBF9LibZHYtiasO0JL7V1f7bloFHheuXxEjAyk67f2A/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:33%;" />

8. Replica：副本，为保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且kafka仍然能够继续工作，kafka提供了**副本机制**，**一个 topic的每个分区都有若干个副本**， **一个 leader 和若干个 follower**

9. leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader

10. follower：每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 follower
11. 重平衡：Rebalance。消费者组内**某个消费者实例挂掉**后，**其他消费者实例自动重新分配订阅主题分区**的过程。Rebalance 是 Kafka 消费者端实现**高可用**的重要手段
12. Zookeeper：Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance
13. 批次（batch）：为了提高效率， 消息会`分批次`写入 Kafka，批次就代指的是一组消息

### Kafka 架构深入

#### Kafka 工作流程及文件存储机制

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820103108.png" style="zoom:33%;" />

Kafka 中消息是以 **topic** 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的

topic 是**逻辑**上的概念，而 partition 是**物理**上的概念，**每个 partition 对应于一个 log 文件**，该 log 文件中**存储**的就是 **producer 生产的数据**。Producer 生产的数据会被不断**追加**到该 log 文件末端，且每条数据都有自己的 **offset**。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820103528.png" style="zoom:33%;" />

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了**分片和索引机制**，将每个 partition 分为多个 segment。每个 segment 对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic 名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first0,first-1,first-2

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820104007.png" style="zoom:33%;" />

**“.index”文件存储大量的索引信息**，**“.log”文件存储大量的数据**，索引文件中的元 数据指向对应数据文件中message 的物理偏移地址

#### Kafka 生产者

##### 消息的发送过程

<img src="https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw7haRlQPaFcxDRgI51Mdf274jwyCVEC2ea85kfvH5dice8D3uOh4W8LB6ACNNE42uFp1GBoKcvbgiaA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:33%;" />

我们从创建一个`ProducerRecord` 对象开始，ProducerRecord 是 Kafka 中的一个核心类，它代表了一组 Kafka 需要发送的 `key/value` 键值对，它由记录要发送到的主题名称（Topic Name），可选的分区号（Partition Number）以及可选的键值对构成

在发送 ProducerRecord 时，我们需要将**键值对对象**由**序列化器**转换为**字节数组**，这样它们才能够在网络上传输。然后消息到达了**分区器**

如果发送过程中**指定了有效的分区号**，那么在发送记录时将**使用该分区**。如果发送过程中**未指定分区**，则将**使用key 的 hash 函数映射指定一个分区**。如果发送的过程中**既没有分区号也没有 key**，则将以**轮询的方式**分配一个分区。选好分区后，生产者就知道向哪个主题和分区发送数据了

ProducerRecord 还有关联的时间戳，如果用户没有提供时间戳，那么生产者将会在记录中使用当前的时间作为时间戳。Kafka 最终使用的时间戳取决于 topic 主题配置的时间戳类型

* 如果将主题配置为使用 `CreateTime`，则生产者记录中的时间戳将由 broker 使用
* 如果将主题配置为使用`LogAppendTime`，则生产者记录中的时间戳在将消息添加到其日志中时，将由 broker 重写

然后，这条**消息被存放在一个记录批次**里，这个批次里的所有消息会被发送到相同的主题和分区上。由一个独立的线程负责把它们发到 Kafka Broker 上

Kafka Broker 在收到消息时会返回一个响应，如果写入成功，会返回一个 RecordMetaData 对象，**它包含了主题和分区信息，以及记录在分区里的偏移量，上面两种的时间戳类型也会返回给用户**。如果写入失败，会返回一个错误。生产者在收到错误之后会尝试重新发送消息，几次之后如果还是失败的话，就返回错误消息

##### 分区策略

1）分区的原因

* **方便在集群中扩展**，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic 又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了；
* 可以**提高并发**，因为可以**以 Partition 为单位读写**了

2）分区的原则

我们需要将 producer 发送的数据封装成一个 **ProducerRecord** 对象

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820104610.png" style="zoom:33%;" />

（1）指明 partition 的情况下，直接将指明的值直接作为 partiton 值；

（2）没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；

（3）既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法

##### 创建 Kafka 生产者

要向 Kafka 写入消息，首先需要创建一个生产者对象，并设置一些属性。Kafka 生产者有3个必选的属性

* bootstrap.servers
* key.serializer
* value.serializer

```java
// 创建了一个 Properties 对象
private Properties properties = new Properties();
properties.put("bootstrap.servers","broker1:9092,broker2:9092");
// 使用 StringSerializer 序列化器序列化 key / value 键值对
properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
// 创建了一个新的生产者对象，并为键值设置了恰当的类型，然后把 Properties 对象传递给他
producer = new KafkaProducer<String,String>(properties);
```

##### 数据可靠性策略

为保证 producer 发送的数据，能可靠的发送到指定的 topic，**topic 的每个 partition 收到 producer 发送的数据**后，都**需要向 producer 发送ack（acknowledgement 确认收到）**，如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820104956.png" style="zoom:33%;" />

1）副本数据同步策略

| 方案                         | 优点                                                     | 缺点                                                       |
| ---------------------------- | -------------------------------------------------------- | ---------------------------------------------------------- |
| 半数以上完成同步，就发送 ack | 延迟低                                                   | 选举新的 leader 时，容忍 n 台 节点的故障，需要 2n+1 个副本 |
| 全部完成同步，才发送 ack     | 选举新的 leader 时，容忍 n 台节点的故障，需要 n+1 个副本 | 延迟高                                                     |

Kafka 选择了第二种方案，原因如下：

1. 同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1
2. 虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka 的影响较小

2）ISR

采用第二种方案之后，设想以下情景：leader 收到数据，所有 follower 都开始同步数据， 但有一个 follower，因为某种故障，迟迟不能与 leader 进行同步，那 leader 就要一直等下去，直到它完成同步，才能发送 ack。这个问题怎么解决呢？

Leader 维护了一个动态的 in-sync replica set (ISR)，意为**和 leader 保持同步的 follower 集合**。**当 ISR 中的 follower 完成数据的同步**之后，**leader 就会给 prodcucer 发送 ack**。**如果 follower 长时间未向 leader 同步数据**，则**该 follower 将被踢出 ISR** ，该时间阈值由 replica.lag.time.max.ms 参数设定。**Leader 发生故障**之后，就会**从 ISR中选举新的 leader**

3）ack 应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失， 所以没必要等 ISR 中的 follower 全部接收成功

所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡， 选择以下的配置

**acks 参数配置：**

0：producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还没有写入磁盘就已经返回，**当 broker 故障时有可能丢失数据**；

1：producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，**如果在 follower 同步成功之前 leader 故障，那么将会丢失数据**；

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820112223.png" style="zoom:33%;" />

-1（all）：producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是**如果在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，那么会造成数据重复**。

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820112750.png" style="zoom:33%;" />

4）故障处理细节

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820113059.png" style="zoom:33%;" />

LEO：指的是**每个副本最大的 offset**；

HW：指的是**消费者能见到的最大的 offset**，**ISR队列中最小的LEO**

（1）follower 故障

follower 发生故障后会被**临时踢出 ISR**，待该 follower 恢复后，follower 会**读取本地磁盘记录的上次的HW**，并**将 log 文件高于HW的部分截取掉**，**从HW开始向 leader 进行同步**。等该 **follower 的LEO大于等于该 Partition 的HW**，即 follower 追上 leader 之后，就可以**重新加入 ISR** 了

（2）leader 故障

leader 发生故障之后，会**从 ISR 中选出一个新的 leader**，之后，为保证多个副本之间的数据一致性，**其余的 follower 会先将各自的 log 文件高于HW的部分截掉**，然后**从新的 leader 同步数据**

##### Exactly Once 语义

将**服务器的 ACK 级别设置为-1**，可以保证 Producer 到 Server 之间**不会丢失数据**，即 **At Least Once** 语义。相对的，将**服务器 ACK 级别设置为 0**，可以保证生产者每条消息只会被发送一次，即 **At Most Once** 语义。

**At Least Once 可以保证数据不丢失**，但是**不能保证数据不重复**；相对的，**At Least Once 可以保证数据不重复**，但是**不能保证数据不丢失**。但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者**要求数据既不重复也不丢失**，即 **Exactly Once** 语义。在 0.11 版本以前的Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。

0.11 版本的Kafka，引入了一项重大特性：**幂等性**。所谓的幂等性就是**指 Producer 不论向 Server 发送多少次重复数据，Server 端都只会持久化一条**。幂等性结合 At Least Once 语义，就构成了Kafka 的 Exactly Once 语义。即：
$$
At Least Once + 幂等性 = Exactly Once
$$
要启用幂等性，只需要将 Producer 的参数中 enable.idompotence 设置为 true 即可。Kafka 的幂等性实现其实就是**将原来下游需要做的去重放在了数据上游**。开启幂等性的 **Producer 在初始化的时候会被分配一个 PID**，**发往同一 Partition 的消息会附带 Sequence Number**。而 **Broker 端会对<PID, Partition, SeqNumber>做缓存**，当具有相同主键的消息提交时，Broker 只会持久化一条

但是 **PID 重启就会变化**，同时不同的 Partition 也具有不同主键，所以**幂等性无法保证跨分区跨会话的 Exactly Once**。

##### 压缩机制

Kafka 的消息分为两层：**消息集合** 和 **消息**。一个消息集合中包含若干条日志项，而日志项才是真正封装消息的地方。Kafka 底层的消息日志由一系列消息集合日志项组成。Kafka 通常不会直接操作具体的一条条消息，它总是在消息集合这个层面上进行`写入`操作。

在 Kafka 中，压缩会发生在两个地方：Kafka Producer 和 Kafka Consumer，为什么启用压缩？说白了就是消息太大，需要`变小一点` 来使消息发的更快一些

Kafka Producer 中使用 `compression.type` 来开启压缩

**有压缩必有解压缩**，Producer 使用压缩算法压缩消息后并发送给服务器后，由 Consumer 消费者进行解压缩，因为采用的何种压缩算法是随着 key、value 一起发送过去的，所以消费者知道采用何种压缩算法

##### Kafka 重要参数配置

**key.serializer**

**value.serializer**

**acks**

**buffer.memory**：此参数用来设置生产者内存缓冲区的大小，生产者用它缓冲要发送到服务器的消息

**compression.type**

**retries**：生产者从服务器收到的错误有可能是临时性的错误（比如分区找不到首领），在这种情况下，`reteis` 参数的值决定了生产者可以重发的消息次数，如果达到这个次数，生产者会放弃重试并返回错误

**batch.size**：当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。当批次被填满，批次里的所有消息会被发送出去。不过生产者井不一定都会等到批次被填满才发送，任意条数的消息都可能被发送。

**client.id**：此参数可以是任意的字符串，服务器会用它来**识别消息的来源**，**一般配置在日志**里

#### Kafka 消息发送

实例化生产者对象后，接下来就可以开始发送消息了，发送消息主要由下面几种方式

##### 简单消息发送

```java
// 这个构造函数，需要传递的是 topic主题，key 和 value
ProducerRecord<String,String> record = new ProducerRecord<String, String>("CustomerCountry","West","France");
// 生产者(producer)的 send() 方法需要把 ProducerRecord 的对象作为参数进行发送
producer.send(record);
```

我们可以从生产者的架构图中看出，消息是先被写入分区中的缓冲区中，然后分批次发送给 Kafka Broker

<img src="https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw7haRlQPaFcxDRgI51Mdf27U4gKtXrlRWJaJs4gibxO060c2r9icIic7ia1iclZFgRGdIpdGQTwy3Re76g/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:33%;" />

发送成功后，send() 方法会返回一个 `Future(java.util.concurrent)` 对象，Future 对象的类型是 `RecordMetadata`类型，我们上面这段代码**没有考虑返回值**，所以没有生成对应的 Future 对象，所以没有办法知道消息是否发送成功。如果不是很重要的信息或者对结果不会产生影响的信息，可以使用这种方式进行发送。

##### 同步发送消息

```java

ProducerRecord<String,String> record = new ProducerRecord<String, String>("CustomerCountry","West","France");

try{
  RecordMetadata recordMetadata = producer.send(record).get();
}catch(Exception e){
  e.printStackTrace()；
}
```

这种发送消息的方式较上面的发送方式有了改进，首先调用 send() 方法，然后再调用 get() 方法等待 Kafka 响应。如果服务器返回错误，get() 方法会抛出异常，如果没有发生错误，我们会得到 `RecordMetadata` 对象，可以用它来查看消息记录。

##### 异步发送消息

同步发送消息都有个问题，那就是同一时间只能有一个消息在发送，这会造成许多消息无法直接发送，造成消息滞后，无法发挥效益最大化。

为了在异步发送消息的同时能够对异常情况进行处理，生产者提供了回掉支持。下面是回调的一个例子

```java

ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>("CustomerCountry", "Huston", "America");
        producer.send(producerRecord,new DemoProducerCallBack());


class DemoProducerCallBack implements Callback {

  public void onCompletion(RecordMetadata metadata, Exception exception) {
    if(exception != null){
      exception.printStackTrace();;
    }
  }
}
```

首先实现回调需要定义一个实现了`org.apache.kafka.clients.producer.Callback`的类，这个接口只有一个 `onCompletion`方法。如果 kafka 返回一个错误，onCompletion 方法会抛出一个非空(non null)异常，这里我们只是简单的把它打印出来，如果是生产环境需要更详细的处理，然后在 send() 方法发送的时候传递一个 Callback 回调的对象

#### Kafka 消费者

##### 创建消费者

在读取消息之前，需要先创建一个 `KafkaConsumer` 对象。创建 KafkaConsumer 对象与创建 KafkaProducer 对象十分相似 --- 把需要传递给消费者的属性放在 `properties` 对象中

```java
Properties properties = new Properties();
properties.put("bootstrap.server","192.168.1.9:9092");     properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");   properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(properties);
```

###### 主题订阅

创建好消费者之后，下一步就开始订阅主题了。`subscribe()` 方法接受一个主题列表作为参数，使用起来比较简单

```java
consumer.subscribe(Collections.singletonList("customerTopic"));
```

##### 消费方式

**consumer 采用 pull（拉）模式从 broker 中读取数据**。

**push（推）模式很难适应消费速率不同的消费者**，因为消息发送速率是由 broker 决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 **pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息**。

pull 模式不足之处是，**如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据**。针对这一点，Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。

##### 消费能力

**总结起来就是如果应用需要读取全量消息，那么请为该应用设置一个消费组；如果该应用消费能力不足，那么可以考虑在这个消费组里增加消费者**。

##### 分区分配策略

一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及 到 partition 的分配问题，即确定那个 partition 由哪个 consumer 来消费

Kafka 有两种分配策略，一是 RoundRobin，一是 Range

1）RoundRobin

2）Range

##### offset 的维护

由于 consumer 在消费过程中可能会出现断电宕机等故障，consumer 恢复后，需要从故障前的位置的继续消费，所以 consumer 需要**实时记录**自己消费到了哪个 **offset**，以便故障恢复后继续消费。

![](https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820135546.png)

Kafka 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始， consumer 默认将 offset 保存在Kafka 一个内置的 topic 中，该 topic 为**__consumer_offsets**

##### 消费者重平衡

我们从上面的`消费者演变图`中可以知道这么一个过程：最初是一个消费者订阅一个主题并消费其全部分区的消息，后来有一个消费者加入群组，随后又有更多的消费者加入群组，而新加入的消费者实例`分摊`了最初消费者的部分消息，这种把分区的所有权通过一个消费者转到其他消费者的行为称为`重平衡`，英文名也叫做 `Rebalance` 

<img src="https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw7haRlQPaFcxDRgI51Mdf275CDVlX6jFQr4kCtLjFaeFsR5PIwM8Qwdvh0fcFdCr1HVIb16yOFzBA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

重平衡非常重要，它为消费者群组带来了`高可用性` 和 `伸缩性`，我们可以放心的添加消费者或移除消费者，不过在正常情况下我们并不希望发生这样的行为。在重平衡期间，消费者无法读取消息，造成整个消费者组在重平衡的期间都不可用。另外，当分区被重新分配给另一个消费者时，消息当前的读取状态会丢失，它有可能还需要去刷新缓存，在它重新恢复状态之前会拖慢应用程序。

消费者通过向`组织协调者`（Kafka Broker）发送心跳来维护自己是消费者组的一员并确认其拥有的分区。对于不同不的消费群体来说，其组织协调者可以是不同的。只要消费者定期发送心跳，就会认为消费者是存活的并处理其分区中的消息。当消费者检索记录或者提交它所消费的记录时就会发送心跳。

如果过了一段时间 Kafka 停止发送心跳了，会话（Session）就会过期，组织协调者就会认为这个 Consumer 已经死亡，就会触发一次重平衡。如果消费者宕机并且停止发送消息，组织协调者会等待几秒钟，确认它死亡了才会触发重平衡。在这段时间里，**死亡的消费者将不处理任何消息**。在清理消费者时，消费者将通知协调者它要离开群组，组织协调者会触发一次重平衡，尽量降低处理停顿。

重平衡的过程对消费者组有极大的影响。因为每次重平衡过程中都会导致万物静止，在**重平衡期间**，消费者组中的**消费者实例都会停止消费**，等待重平衡的完成。而且重平衡这个过程很慢......

#### Kafka 高效读写数据

1）Kafka 本身是分布式集群，同时采用分区技术，并发度高

1）**顺序写磁盘**

Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端，为**顺序写**。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其**省去了大量磁头寻址的时间**

2）**零拷贝技术**

Kafka 实现了`零拷贝`原理来快速移动数据，避免了内核之间的切换

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820140124.png" style="zoom:33%;" />

3）**消息压缩**

批处理能够进行更有效的数据压缩并减少 I/O 延迟

4）**分批发送**

Kafka 可以将数据记录分批发送，从生产者到文件系统（Kafka 主题日志）到消费者，可以端到端的查看这些批次的数据

#### Zookeeper 在 Kafka 中的作用

Kafka 集群中有一个 broker 会被选举为 **Controller**，负责**管理集群 broker 的上下线**，**所有 topic 的分区副本分配**和 **leader 选举**等工作

Controller 的管理工作都是依赖于 Zookeeper 的

以下为 partition 的 leader 选举过程：

<img src="https://cdn.jsdelivr.net/gh/whn961227/images@master/data/20200820142446.png" style="zoom:33%;" />

#### Kafka 事务

Kafka 从 0.11 版本开始引入了事务支持。事务可以**保证 Kafka 在 Exactly Once 语义的基础上**，生产和消费可以**跨分区和会话**，要么全部成功，要么全部失败

##### Producer 事务

为了实现跨分区跨会话的事务，需要引入一个**全局唯一的 Transaction ID**，并将 Producer 获得的 PID 和 Transaction ID 绑定。这样当Producer重启后就可以通过正在进行的Transaction ID 获得原来的 PID

为了管理Transaction，Kafka 引入了一个新的组件 **Transaction Coordinator**。**Producer** 就是通过**和 Transaction Coordinator 交互**获得 **Transaction ID 对应的任务状态**。Transaction Coordinator 还负责**将事务所有写入 Kafka 的一个内部 Topic**，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行

##### Consumer 事务

上述事务机制主要是从 Producer 方面考虑，对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况

### Zookeeper 中存储 Kafka 哪些信息

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914171850.png)