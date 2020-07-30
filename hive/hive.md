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

* **数据存储位置**



### 架构原理

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200730154647.png" style="zoom:25%;" />

* **用户接口：Client**

* **元数据：Metastore**

  元数据包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等

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

