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
| 执行引擎     | 默认 MR        | 自己的执行引擎           |
| 执行延迟     | 高             | 低                       |
| 可扩展性     | 高             | 低                       |
| 数据规模     | 大             | 小                       |



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
partitioned by (month string) -- 根据新字段进行分区
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



### 查询

#### 基本查询

**全表和特定列查询**

```sql
-- 全表
select * from emp;
-- 特定列
select empno,ename from emp;
```

**列别名**

```sql
-- 紧跟列名，也可以在列名和别名之间加入关键字 AS
select ename as name, deptno dn from emp;
```

**算数运算符**

| 运算符 | 描述           |
| ------ | -------------- |
| A+B    | A和B 相加      |
| A-B    | A减去B         |
| A*B    | A和B 相乘      |
| A/B    | A除以B         |
| A%B    | A对B取余       |
| A&B    | A和B按位取与   |
| A\|B   | A和B按位取或   |
| A^B    | A和B按位取异或 |
| ~A     | A按位取反      |

**常用函数**

```sql
-- 求总行数
select count(*) cnt from emp;
-- 求最大值
select max(sal) max_sal from emp;
-- 求最小值
select min(sal) min_sal from emp;
-- 求和
select sum(sal) sum_sal from emp;
-- 求平均值
select avg(sal) avg_sal from emp;
```

**limit 语句**

```sql
-- 限制返回的行数
select * from emp limit 5;
```

#### Where 语句

```sql
-- 使用 where 将不满足条件的行过滤掉，where 字句紧随 from 字句
select * from emp where sal > 1000;
```

**比较运算符**

| 操作符                  | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| A <=> B                 | 如果 A 和 B 都为 NULL，则返回 TRUE，其他的和等号（=）操作符的结果一致，如果任一为 NULL 则结果为 NULL |
| A <> B, A != B          | 如果 A 不等于 B，则返回 TRUE，否则返回 FALSE，如果任一为 NULL 则结果为 NULL |
| A [NOT] BETWEEN B AND C | 如果A，B 或者 C 任一为 NULL，则结果为 NULL。如果 A 的值大于等于 B 而且小于或等于 C，则结果为 TRUE，反之为 FALSE。如果使用 NOT 关键字则可达到相反的效果 |
| A IS NULL               | 如果 A 等于 NULL，则返回 TRUE，反之返回 FALSE                |
| IN(数值1, 数值2)        | 使用 IN 运算显示列表中的值                                   |
| A [NOT] LIKE B          | B 是一个 SQL 下的简单正则表达式，如果 A 与其匹配的话，则返回 TRUE；反之返回 FALSE。B 的表达式说明如下：‘x%’表示A必须以字母‘x’开头，‘%x’表示A必须以字母’x’结尾，而‘%x%’表示A包含有字母’x’,可以位于开头，结尾或者字符 |
| A RLIKE B, A REGEXP B   | B是一个正则表达式，如果A与其匹配，则返回TRUE；反之返回FALSE。匹配使用的是JDK中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串A相匹配，而不是只需与其字符串匹配 |

**Like 和 RLike**

RLike 字句是 Hive 中的Like 功能的扩展，可以通过 Java 的正则表达式来指定匹配条件

**逻辑运算符（And/Or/Not）**

#### 分组

**Group By 语句**

Group by 语句通常会和聚合函数一起使用，按照一个或多个列的结果进行分组，然后对每个组执行聚合操作

**Having 语句**

1. where 针对表中的列发挥作用，查询数据；having 针对查询结果中的列发挥作用，筛选数据
2. where 后面不能写分组函数，having 后面可以使用分组函数
3. having 只用于 group by 分组统计语句

```sql
-- 求每个部门的平均薪水大于 2000 的部门
select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal > 2000
```

#### Join 语句

**等值 Join**

Hive 支持通常的 SQL JOIN 语句，但是只支持等值连接，不支持非等值连接

**表的别名**

1. 使用别名可以简化查询
2. 使用表名前缀可以提高执行效率

**内连接：**只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来

**左外连接：**JOIN 操作符左边表中符合 Where 字句的所有记录将会被返回

**右外连接：**JOIN 操作符右边表中符合 Where 字句的所有记录将会被返回

**满外连接：**将会返回所有表中符合 where 语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，就使用 NULL 值替代

**多表连接：**连接 n 个表，至少需要 n-1 个连接条件

**笛卡尔积**

笛卡尔积会在下面条件产生：

1. 省略连接条件
2. 连接条件无效
3. 所有表中的所有行互相连接

**连接谓词中不支持 or**

#### 排序

**全局排序：**Order by

**按照别名排序**

**多个列排序**

```sql
-- 按照部门和工资升序排序
select ename, deptno, sal from emp order by deptno, sal;
```

**每个 MR 内部排序：**Sort by，每个 Reducer 内部进行排序，对全局结果集来说不是排序

**分区：**Distribute by，类似 MR 中的 partition，进行分区，结合 Sort by 使用

> 注意，Hive 要求 Distribute by 语句要写在 Sort by 语句之前

```sql
-- 先按照部门编号分区，再按照员工编号降序排序
select * from emp distribute by deptno sort by empno desc;
```

**Cluster By：**当 distribute by 和 sort by 字段相同时，可以使用 cluster by 方式

cluster by 除了具有 distribute by 的功能外还兼具 sort by 的功能，但是排序只能是**升序**排序，不能指定排序规则为 ASC 或者 DESC

#### 分桶及抽样查询

**分区针对的是数据的存储路径；分桶针对的是数据文件**

分区提供一个隔离数据和优化查询的遍历方式

分桶是将数据集分解成更容易管理的若干部分的另一个技术

```sql
-- 需要设置属性
set hive.enforce.bucketing = true;
set mapreduce.job.reduces = -1;

-- 创建普通表
create table stu(id int, name string)
row format delimited fields terminated by '\t';
-- 导入数据
load data local inpath '/opt/module/datas/student.txt' into table stu;
-- 创建分桶表
create table stu_buck(id int, name string)
clustered by(id)
into 4 buckets
row format delimited fields terminated by '\t';
-- 导入数据到分桶表，通过子查询的方式
insert into table stu_buck
select id, name from stu;
```

**分桶抽样查询**

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果，Hive 可以通过对表进行抽样来满足这个需求

```sql
select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

> 注：tablesample 是抽样语法，语法：tablesample (bucket x out of y)
>
> y 必须是 table 总 bucket 数的倍数或者因子。Hive 根据 y 的大小，决定抽样的比例。例如，table 总共分了 4 份，当 y = 2 时，抽取 (4/2=) 2 个 bucket 的数据，当 y = 8 时，抽取 (4/8=)1/2 个 bucket 的数据
>
> x 表示从哪个 bucket 开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上 y。例如，table 总 bucket 数为 4，tablesample(bucket 1 out of 2)，表示总共抽取 (4/2=) 2 个 bucket 的数据，抽取第 1(x) 个和第 3(x+y) 个 bucket 的数据
>
> 注意：x 的值必须小于等于 y 的值



#### 其他常用查询函数

**空字段赋值：**NVL (string1, replace_with)，如果 string1 为 NULL，则 NVL 函数返回 replace_with 的值，否则返回 string1 的值，如果两个参数都为 NULL，则返回 NULL

**CASE WHEN**

```sql
-- 求出不同部门男女各多少人
select
	dept_id,
	sum(case sex when '男' then 1 else 0 end) male_count,
	sum(case sex when '女' then 1 else 0 end) female_count,
from
	emp
group by
	dept_id;
```

**行转列**

CONCAT (string A/col, string B/col ...)：返回输入字符串连接后的结果，支持任意个输入字符串

CONCAT_WS (separator, str1, str2, ...)：它是一个特殊形式的 CONCAT()。第一个参数是参数间的分隔符

COLLECT_SET (col)：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生 array 类型字段

```sql
/*
name	constellation	blood_type
孙悟空    	白羊座            	A
大海	    射手座	            A
宋宋	    白羊座	            B
猪八戒    	白羊座            	A
凤姐	    射手座	            A


射手座,A            大海|凤姐
白羊座,A            孙悟空|猪八戒
白羊座,B            宋宋
*/
select
	base,
	concat_ws("|", collect_set(t1.name))
from
	(select
        name,
        concat(constellation, ",", blood_type) base
    from
        person_info) t1
group by
	t1.base;
```

**列转行**

EXPLODE (col)：将 hive 一列中复杂的 array 或者 map 结构拆分成多行

LATERAL VIEW

* 用法：LATERAL VIEW udft (expression) tableAlias AS columnAlias
* 解释：用于和 split，explode 等 UDTF 一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合

```sql
/*
movie			category
《疑犯追踪》		悬疑,动作,科幻,剧情
《Lie to me》		悬疑,警匪,动作,心理,剧情
《战狼2》		战争,动作,灾难


《疑犯追踪》      悬疑
《疑犯追踪》      动作
《疑犯追踪》      科幻
《疑犯追踪》      剧情
《Lie to me》   悬疑
《Lie to me》   警匪
《Lie to me》   动作
《Lie to me》   心理
《Lie to me》   剧情
《战狼2》        战争
《战狼2》        动作
《战狼2》        灾难
*/
select
	movie,
	category_name
from
	movie_info lateral view explode(category) table_tmp as category_name;
```

**窗口函数**

OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变化而变化

CURRENT ROW：当前行

n PRECEDING：往前 n 行数据

n FOLLOWING：往后 n 行数据

UNBOUNDED：起点

UNBOUNDED PRECEDING：表示从前面的起点

UNBOUNDED FOLLOWING：表示到后面的终点

LAG (col, n)：往前第 n 行数据

LEAD (col, n)：往后第 n 行数据

NTILE (n)：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从 1 开始，对于每一行，NTILE 返回此行所属的组的编号

> 注意，n 必须为 int 类型

```sql
/*
jack,2017-01-01,10
tony,2017-01-02,15
jack,2017-02-03,23
tony,2017-01-04,29
jack,2017-01-05,46
jack,2017-04-06,42
tony,2017-01-07,50
jack,2017-01-08,55
mart,2017-04-08,62
mart,2017-04-09,68
neil,2017-05-10,12
mart,2017-04-11,75
neil,2017-06-12,80
mart,2017-04-13,94
*/

-- 查询在 2017 年 4 月份购买过的顾客及总人数
select
	name,
	count(*) over()
from
	business
where
	substring(orderdate, 1, 7) = '2017-04'
group by
	name
-- 查询顾客的购买明细及月购买总额
select
	name,
	orderdate,
	cost,
	sum(cost) over(partition by month(orderdate))
from
	business
-- 将 cost 按照日期累加
select
	name,
	orderdate,
	cost,
	sum(cost) over() as sample1, -- 所有行相加
	sum(cost) over(partition by name) as sample2, -- 按 name 分组，组内数据相加
	sum(cost) over(partition by name order by orderdate) as sample3, -- 按 name 分组，组内数据累加
	sum(cost) over(partition by name order by orderdate between unbounded preceding and current row) as sample4, -- 和 sample3 一样，由起点到当前行的聚合
	sum(cost) over(partition by name order by orderdate between 1 preceding and current row) as sample5, -- 当前行和前面一行做聚合
	sum(cost) over(partition by name order by orderdate between 1 preceding and 1 following) as sample6, -- 当前行和前面一行及后面一行做聚合
	sum(cost) over(partition by name order by orderdate between current row and unbounded following) as sample7 -- 当前行和后面所有行做聚合
from
	business;
-- 查看顾客上次的购买时间
select
	name,
	orderdate,
	cost,
	lag(orderdate, 1, '1900-01-01') over(partition by name order by orderdate) as time1,
	lag(orderdate, 2) over(partition by name order by orderdate) as time2
from
	business;
-- 查询前 20% 时间的订单信息
select
	*
from
	(select
        name,
        orderdate,
        cost,
        ntile(5) over(order by orderdate) sorted
    from
        business) t
where
	t.sorted = 1;
```

**Rank**

rank()：排序相同时会重复，总数不会变

dense_rank()：排序相同时会重复，总数会减少

row_number()：会根据顺序计算



###  调优

#### Fetch 抓取

Fetch 抓取是指 **Hive 中对某些情况的查询可以不必使用 MR 计算**

在 hive-default.xml.template 文件中 hive.fetch.task.conversion 默认是 more，该属性修改为 **more** 后，在**全局查找、字段查找、limit 查找**等都不走 MR

#### 本地模式

在 Hive 的输入数据量非常小的情况下，为查询触发执行任务消耗的时间可能会比实际 Job 的执行时间要多，**Hive 可以通过本地模式在单台机器上处理所有的任务，对于小数据集，执行时间可以明显被缩短**

通过设置 hive.exec.model.local.auto 的值为 true，让 Hive 在适当的时候自动启动这个优化

#### 表的优化

##### 小表、大表 join

参照[ MR 中 join 的多种引用](# join的多种引用)

##### 大表 join 大表

**空 key 过滤**

**空 key 转换**

##### Group by

默认情况下，Map 阶段同一 key 数据分发到一个 Reduce，当一个 key 数据过大时就发生数据倾斜

并不是所有的聚合操作都需要在 Reduce 端完成，很多聚合操作都可以先在 Map 端进行部分聚合，最后在 Reduce 端得出最终结果

1. 开启 Map 端聚合参数设置

   * 是否在 Map 端进行聚合，默认为 true

     hive.map.aggr = true

   * 在 Map 端进行聚合操作的条目数目

     hive.groupby.mapaggr.checkinterval = 100000

   * 有数据倾斜的时候进行负载均衡（默认是 false）

     hive.groupby.skewindata = true

   当选项设定为 true 时，生成的查询计划会有两个 MR job。第一个 MR job 中，Map 的输出结果会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 group by key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR job 中在根据预处理的数据结果按照 group by key 分布到 Reduce 中（这个过程可以保证相同的 group by key 被分布到同一个 Reduce 中），最后完成最终的聚合操作

##### Count（distinct）去重统计

数据量小的时候无所谓，数据量大的情况下，由于 Count distinct 操作需用一个 Reduce Task 来完成，这个 Reduce 需要处理的数据量太大，就会导致整个 job 很难完成，一般 count distinct 使用先 group by 再 count 的方式替换

##### 笛卡尔积

##### 行列过滤

列处理：在 select 中，只拿需要的列，如果有，尽量使用分区过滤，少用 select *

行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在 where 后，那么就会先全表关联，之后再过滤

##### 动态分区调整

##### 分桶

参考 [分桶及抽样查询](#分桶及抽样查询)

##### 分区

参考 [分区表](#分区表)

#### 数据倾斜

##### 合理设置 Map 数

1. 通常情况下，作业会通过 input 的目录产生一个或者多个 map 任务

2. 是不是 map 数越多越好

   不是，如果一个任务有很多小文件（远远小于块大小 128m），则每个小文件也会被当作一个块，用一个 map 任务来完成，而**一个 map 任务启动和初始化的时间远远大于逻辑处理的时间**，就会造成很大的资源浪费，而且，同时可执行的 map 数是受限的

3. 是不是保证每个 map 处理接近 128M 的文件块就 OK 了

   不一定，比如一个 127M 的文件，正常会用一个 map 去完成，但这个文件只有一个或两个小字段，却又几千万的记录，如果 map 处理的逻辑比较复杂，用一个 map 任务去做，肯定比较耗时

##### 小文件进行合并

在 map 执行前合并小文件，减少 map 数：CombineHiveInputFormat 具有对小文件进行合并的功能（系统默认的格式），HiveInputFormat 没有对小文件合并功能

set hive.input.format = org.apache.hadoop.hive.ql.io.CombineHiveInputFormat

##### 复杂文件增加 Map 数

当 input 的文件都很大，任务逻辑复杂，map 执行非常慢的时候，可以考虑增加 Map 数，来使得每个 map 处理的数据量减少，从而提高任务的执行效率

增加 map 的方法：根据 computeSliteSize(Math.max(minSize, Math.min(maxSize, blocksize))) = blocksize = 128M 公式，调整 maxSize 最大值，让 maxSize 最大值低于 blocksize 就可以增加 map 的个数

##### 合理设置 Reduce 数

1. 确定 Reduce 个数的方法

   * 每个 Reduce 处理的数据量默认是 256M

     hive.exec.reducers.bytes.per.reducer = 256000000

   * 每个任务最大的 reduce 数，默认为 1009

     hive.exec.reducers.max = 1009

   * 计算 reducer 数的公式

     N = min( 参数 2，总输入数据量 / 参数 1 )

2. 调整 reduce 个数

   1. 方法一：

      set hive.exec.reducers.bytes.per.reducer = 50000000

   2. 方法二：

      set mapred.reduce.tasks = 15

3. Reduce 个数并不是越多越少

   * 过多的启动和初始化 Reduce 会消耗时间和资源
   * 有多少个 Reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题

#### 并行执行

Hive 会将一个查询转化成一个或者多个阶段。这样的阶段可以是 MR 阶段、抽样阶段、合并阶段、limit 阶段。或者 Hive 执行过程中可能需要的其他阶段。默认情况下，Hive 一次只会执行一个阶段。不过，某个特定的 job 可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个 job 的执行时间缩短

```shell
set hive.exec.parallel = true; # 打开任务并行执行
set hive.exec.parallel.thread.number = 16; # 同一个 sql 允许最大并行度，默认为 8
```

#### 严格模式

Hive 提供了严格模式，可以防止用户执行那些可能意想不到的不好的影响的查询

```shell
# 通过设置属性 hive.mapred.mode 值为默认是非严格模式 nonstrict
<property>
    <name>hive.mapred.mode</name>
    <value>strict</value>
</property>
```

开启严格模式可以禁止 3 种类型的查询

1. 对于分区表，**除非 where 语句中含有分区字段过滤条件来限制范围，否则不允许执行**
2. 对于**使用了 order by 语句的查询，要求必须使用 limit 语句**
3. **限制笛卡尔积的查询**

#### JVM 重用

#### 推测执行

#### 压缩

#### 执行计划（Explain）