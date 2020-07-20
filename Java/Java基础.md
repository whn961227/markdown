## 基础

### Java 基本功

#### JVM、JDK、JRE

##### JVM

**什么是字节码？采用字节码的好处是什么**

> 在 Java 中，JVM 可以理解的代码就叫做 **字节码**（即扩展名为 .class 的文件），它只面向虚拟机。

Java 程序从源代码到运行一般有下面 3 步：

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200720225705419.png" alt="image-20200720225705419" style="zoom:80%;" />

格外注意 .class -> 机器码 这一步，

**总结：**

* Java 虚拟机（JVM）是运行 Java 字节码的虚拟机

* JVM 有针对不同系统的特定实现（Windows、Linux、macOS），目的是使用相同的字节码，他们都会给出相同的结果
* 字节码和不同系统的 JVM 实现是 Java 语言 “一次编译，随处可以运行” 的关键所在

