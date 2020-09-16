## Flume

### 定义

Flume 是 Cloudera 提供的一个高可用的，高可靠的，**分布式的海量日志采集、聚合和传输的系统**

### Flume 组成

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914150950.png)

Taildir Source：断点续传、多目录

File Channel：数据存储在磁盘，宕机数据可以保存。但是传输速率慢，适合对数据传输可靠性要求高的场景

Memory Channel：数据存储在内存中，宕机数据丢失。传输速率快，适合对数据传输可靠性要求不高的场景，比如，普通日志数据

Kafka Channel：减少了 Flume 的 Sink 阶段，提高了传输效率

### Put 事务，Take 事务

Source 到 Channel 是 Put 事务

Channel 到 Sink 是 Take 事务

### Agent 内部原理

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914152749.png)



