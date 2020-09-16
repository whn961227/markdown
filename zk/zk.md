## Zookeeper

### 概述

ZK 是一个开源的分布式的，为分布式应用提供协调服务的 Apache 项目

**工作机制**

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914142014.png)

### 内部原理

#### 选举机制

1. 半数机制：集群中半数以上机器存活，集群可用。所以 ZK 适合安装奇数台服务器

#### 节点类型

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914141734.png)

#### 监听器原理

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200914145751.png)