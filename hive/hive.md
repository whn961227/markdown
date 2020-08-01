## Hive

### 概述

Hive 是基于 Hadoop 的一个**数据仓库**工具，可以将**结构化的数据文件映射为一张表**，并提供**类 SQL** 查询功能

**本质：**将 HQL 转化为 MapReduce 程序

1. Hive 处理的数据存储在 HDFS
2. Hive 分析数据底层的实现是 MapReduce
3. 执行程序运行在 Yarn 上



### 优缺点

**优点：**

* 支持用户自定义函数
* Hive 的优势在于处理大数据，对于处理小数据没有优势
* 避免写 MR

**缺点：**

* Hive 的 HQL 表达能力有限
  * 迭代式算法无法表达
  * 数据挖掘方面不擅长
* Hive 的效率比较低
  * Hive 自动生成的 MR 作业，通常情况下不够智能化
  * Hive 调优比较困难，粒度较粗



### Hive 和数据库比较

| 对比项       | Hive           | 传统数据库               |
| ------------ | -------------- | ------------------------ |
| 数据存储位置 | HDFS           | 块设备或者本地文件系统中 |
| 数据更新     | 不支持         | 支持                     |
| 索引         | 有限的索引功能 | 支持                     |
|              |                |                          |
|              |                |                          |
|              |                |                          |



### 架构原理

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200730154647.png" style="zoom:25%;" />

* **用户接口：Client**

* **元数据：Metastore**

  元数据包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等

  默认存储在自带的 derby 数据库中，推荐使用 MySql 存储元数据

* **Hadoop：**使用 HDFS 进行存储，使用 MR 进行计算

* **驱动器：Driver**
  * **解析器**：将 SQL 字符串转换成抽象语法树 AST，这一步一般都用第三方工具库完成，比如 antlr；对 AST 进行语法分析，比如表是否存在，字段是否存在，SQL 语义是否有误
  * **编译器：**将 AST 编译生成逻辑执行计划
  * **优化器：**对逻辑执行计划进行优化
  * **执行器：**把逻辑执行计划转换成可以运行的物理计划。对于 Hive 来说，就是 MR/Spark



### 数据类型

#### 基本数据类型

| Hive 数据类型 | Java 数据类型 |
| ------------- | ------------- |
| TINYINT       | byte          |
| SAMLINT       | short         |
| INT           | int           |
| BIGINT        | long          |
| BOOLEAN       | boolean       |
| FLOAT         | float         |
| DOUBLE        | double        |
| STRING        | string        |
| TIMESTAMP     |               |
| BINARY        |               |

#### 集合数据类型

Hive 中有三种复杂数据类型 ARRAY、MAP 和 STRUCT



### 自定义函数

#### UDF

**编程步骤：**

1. 继承 org.apache.hadoop.hive.sql.UDF

2. 需要实现 evaluate 函数；evaluate 函数支持重载

3. 在 hive 的命令行窗口创建函数

   * 添加 jar

     ```java
     add jar linux_jar_path
     ```

   * 创建 function

     ```java
     create [temporary] function [dbname.]function name AS class_name;
     ```

4. 在 hive 的命令行窗口删除函数

   ```java
   Drop [temporary] function [if exists] [dbname.]function name;
   ```

> 注意事项：UDF 必须要有返回类型，可以返回 null，但是返回类型不能为 void

#### UDTF

**编程步骤：**

1. 继承 org.apache.hadoop.hive.sql.GenericUDF
2. 重写 initialize，evaluate，getDisplayString 函数

```java
// 按','分割
public class MyUDTF extends GenericUDTF {

    private List<String> dataList = new ArrayList<>();

    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
        // 定义输出数据的列名
        List<String> fieldNames = new ArrayList<>();
        fieldNames.add("word");
        // javaStringObjectInspector 定义输出数据类型
        List<ObjectInspector> fieldOIs = new ArrayList<>();
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    }

    public void process(Object[] objects) throws HiveException {
        // 1. 获取数据
        String data = objects[0].toString();
        // 2. 获取分隔符
        String splitKey = objects[1].toString();
        // 3. 切分数据
        String[] words = data.split(splitKey);
        // 4. 遍历写出
        for (String word : words) {
            // 5. 将数据放入集合
            dataList.clear();
            dataList.add(word);
            // 6. 写出数据的操作
            forward(dataList);
        }
    }

    public void close() throws HiveException {

    }
}
```



### 索引

#### 索引机制

在指定列上建立索引，会产生一张索引表（Hive 的一张物理表），里面的字段包括，**索引列的值**、**该值对应的 HDFS 文件路径**、**该值在文件中的偏移量**

在执行索引字段查询时，首先额外生成一个 MR Job，根据对索引列的过滤条件，从索引表中过滤出索引列的值对应的 HDFS 文件路径和偏移量，输出到 HDFS 上的一个文件中，然后根据这些文件中的 HDFS 路径和偏移量，筛选原始 input 文件，生成新的 split，作为整个 job 的 split，这样就达到不用扫描全表的目的

#### 索引建立过程

**创建索引**

```sql
create index test_index on table test(key)
as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
with deferred rebuild;
```

Hive 中会创建一张索引表，也是物理表

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200731104618.png" style="zoom:25%;" />

其中，索引表中 key 字段，就是原表中 key 字段的值，`_bucketname` 字段，代表数据文件对应的 HDFS 文件路径，`_offsets` 代表该 key 值在文件中的偏移量，有可能有多个偏移量，因此，该字段类型为数组

其实，索引表相当于一个在原表索引列上的一个汇总表

**生成索引数据**

```sql
alter index test_index on test rebuild;
```

用一个 MR 任务，以 test 的数据作为 input，将索引字段 key 中的每一个值及其对应的 HDFS 文件和偏移量输出到索引表中

**自动使用索引**

```sql
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
SET hive.optimize.index.filter=true;
SET hive.optimize.index.filter.compact.minsize=0;
```

查询时索引如何起效：

```sql
select * from test where key = '13400000144_1387531071_460606566970889';
```

1. 首先用一个 job，从索引表中过滤出 `key = '13400000144_1387531071_460606566970889'`的记录，将其对应的 HDFS 文件路径和偏移量输出到 HDFS 临时文件中
2. 接下来，直接定位到该临时文件，根据里面的 HDFS 文件路径和偏移量，用一个 map task 即可完成查询，最终目的是为了减少查询时候的 input size

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200731110132.png" style="zoom:25%;" />

**不使用索引**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200731110257.png" style="zoom:25%;" />

从以上过程可以看成，Hive 索引的使用过程比较繁琐：

1. 每次查询的时候都要先用一个 MR job 扫描索引表，如果索引列的值非常稀疏，那么索引表本身也会非常大
2. 索引表不会自动 rebuild，如果表数据新增或删除，那么必须手动 rebuild 索引表数据



### DDL 数据定义

#### 创建表

**管理表（内部表）：**Hive 创建内部表时，会将数据移动到数据仓库指向的路径，在删除内部表时，内部表的元数据和数据会被一起删除

```sql
create table if not exists student2(
	id int, name string
)
row format delimited fields terminated by '\t'
/*
stored as 指定存储文件类型
常用的存储文件类型：sequencefile(二进制序列文件)、textfile(文本)、rcfile(列式存储格式文件)
如果文件数据是纯文本，可以使用 stored as textfile。
如果数据需要压缩，使用 stored as sequencefile
*/
stored as textfile
-- 指定表在 HDFS 上的存储位置
location '/user/hive/warehouse/student2';
```

**外部表：**因为表是外部表，所以 Hive 并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉

external 关键字可以让用户创建一个外部表，若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变

```sql
create external table if not exists dept(
	deptno int,
    dname string,
    loc int
)
row format delimited fields terminated by '\t';
```

#### 分区表

分区表实际上就是对应一个 HDFS 文件系统上的独立文件夹，该文件夹下是该分区的所有数据文件。Hive 中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过 where 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多

**创建分区表**

```sql
create table test_partition(
	deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

**加载数据到分区表中**

```sql
load data local inpath '/opt/module/datas/test.txt' into table test_partition partition(month='201809');
load data local inpath '/opt/module/datas/test.txt' into table test_partition partition(month='201808');
load data local inpath '/opt/module/datas/test.txt' into table test_partition partition(month='201807');
```

**查询分区表中数据**

单分区查询

```sql
select * from test_partition where month = '201809';
```

多分区联合查询

```sql
select * from test_partition where month = '201809'
union
select * from test_partition where month = '201808';
union
select * from test_partition where month = '201807';
```

**增加分区**

创建单个分区

```sql
alter table test_partition add partition(month = '201806');
```

同时创建多个分区

```sql
alter table test_partition add partition(month = '201805') partition(month = '201804');
```

**删除分区**

删除单个分区

```sql
alter table test_partition drop partition(month = '201806');
```

同时删除多个分区

```sql
alter table test_partition drop partition(month = '201805'), partition(month = '201804');
```

**查看分区**

```sql
show partitions test_partition;
```

**查看分区表结构**

```sql
desc formatted test_partition;
```

**创建二级分区表**

```sql
create table test_partition(
	deptno int, dname string, loc string
)
partitioned by (month string, day string)
row format delimited fields terminated by '\t';
```

**加载数据到二级分区表**

```sql
load data local inpath '/opt/module/datas/dept.txt' into table
 default.test_partition partition(month='201709', day='13');
```

**查询分区数据**

```sql
select * from dept_partition2 where month='201709' and day='13';
```



