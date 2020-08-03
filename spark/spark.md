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

1. 把用户程序转为作业（job）
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

#### Yarn 模式

Spark 客户端直接连接 Yarn，不需要额外构建 Spark 集群，有 yarn-client 和 yarn-cluster 两种模式，主要区别在于：Driver 程序的运行节点

yarn-client：Driver 程序运行在客户端，适用于交互、调试，希望立即看到 app 的输出

yarn-cluster：Driver 程序运行在由 RM 启动的 AppMaster，适用于生产环境



### WordCount 程序

```scala
object WordCount {
    def main(args: Array[String]):Unit={
        // 创建 SparkConf 并设置 APP 名称
        val conf = new SparkConf().setAppName("WordCount");
        // 创建 SparkContext，该对象是提交 Spark APP 的入口
        val sc= new SparkContext(conf);
    }
}
```



## Spark Core