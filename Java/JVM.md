## JVM

### 运行时数据区

**JDK 1.8 之前：**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727104854.png" style="zoom: 25%;" />

**JDK 1.8：**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727105335.png" style="zoom:25%;" />

#### 方法区

用于存放已**被加载的类信息**、**常量**、**静态变量**、**即时编译器编译后的代码**等数据

方法区是一个 JVM 规范，永久代与元空间都是其一种实现方式。在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中。**元空间存储类的元信息**，**静态变量和常量池等放入堆中**

##### 运行时常量池

Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域

除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()



#### 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成

另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存

> 程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡

#### Java 虚拟机栈

Java 虚拟机栈是线程私有的，它的生命周期与线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的

Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口等信息

**局部变量表：**主要存放了编译器可知的**基本数据类型**（short int long float double boolean char byte）、**对象引用**（reference类型，不等同于对象本身，可能是指向对象起始地址的引用指针，可能是指向代表对象的句柄或其他与此对象相关的位置）、**returnAddress类型**（指向一条字节码指令的地址）

**操作数栈：**虚拟机把操作数栈作为工作区，大多数指令都要从这里弹出数据，执行运算，然后把结果压回操作数栈。

```java
begin  
iload_0    // push the int in local variable 0 onto the stack  
iload_1    // push the int in local variable 1 onto the stack  
iadd       // pop two ints, add them, push result  
istore_2   // pop int, store into local variable 2  
end  
    
//   在这个字节码序列里，前两个指令iload_0和iload_1将存储在局部变量中索引为0和1的整数压入操作数栈中，其后iadd指令从操作数栈中弹出那两个整数相加，再将结果压入操作数栈。第四条指令istore_2则从操作数栈中弹出结果，并把它存储到局部变量区索引为2的位置。
```

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727171520.png" style="zoom:25%;" />

**动态链接：**符号引用在运行期间转换为直接引用（对比静态链接）

Java 虚拟机栈会出现两种错误：StackOverFlowError 和 OutOfMemoryError

#### 本地方法栈

类似 Java 虚拟机栈，为虚拟机使用到的 Native 方法服务



### 如何识别垃圾

##### 引用计数法

简单地说，就是**对象被引用一次**，在**它的对象头上加一次引用次数**，如果没有被引用（引用次数为 0），则此对象可回收

它无法解决一个主要的问题：**循环引用**

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXibdb3wDDhArnFibhWhWcHkcMR5QrwrE2pNogJ6rHWicYz7bjE2NibW1ocg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

虽然 a，b 都被置为 null 了，但是由于之前它们指向的对象互相指向了对方（引用计数都为 1），所以无法回收

##### 可达性算法

可达性算法的原理是以一系列叫做  **GC Root** 的对象为起点出发，引出它们指向的下一个节点，再以下个节点为起点，引出此节点指向的下一个结点。。。（这样通过 GC Root 串成的一条线就叫**引用链**），直到所有的结点都遍历完毕，如果相关**对象不在任意一个以 GC Root 为起点的引用链中**，则这些对象会被判断为「垃圾」,会被 GC 回收

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXvZLmx9HdeNibRQjnFZwUBFXTQ6mJy79ajx4BdqFMicIiaW4BNQaGTZWibg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

如图示，如果用可达性算法即可解决上述循环引用的问题，因为从**GC Root** 出发没有到达 a,b,所以 a，b 可回收

a, b 对象可回收，就一定会被回收吗？并不是，对象的 **finalize 方法**给了对象一次垂死挣扎的机会，当对象不可达（可回收）时，当发生 GC 时，会先判断对象是否执行了 finalize 方法，如果未执行，则会先执行 finalize 方法，我们可以在此方法里将当前对象与 GC Roots 关联，这样执行 finalize 方法之后，GC 会再次判断对象是否可达，如果不可达，则会被回收，如果可达，则不回收！

*注意： finalize 方法只会被执行一次，如果第一次执行 finalize 方法此对象变成了可达确实不会回收，但如果对象再次被 GC，则会忽略 finalize 方法，对象会被回收！这一点切记！*

那么这些 **GC Roots** 到底是什么东西呢，哪些对象可以作为 GC Root 呢，有以下几类

* 虚拟机栈（栈帧中的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象

> **虚拟机栈中引用的对象**

如下代码所示，a 是栈帧中的本地变量，当 a = null 时，由于此时 a 充当了 **GC Root** 的作用，a 与原来指向的实例 **new Test()** 断开了连接，所以对象会被回收。

```java
public class Test {
    public static  void main(String[] args) {
        Test a = new Test();
        a = null;
    }
}
```

> **方法区中类静态属性引用的对象**

如下代码所示，当栈帧中的本地变量 a = null 时，由于 a 原来指向的对象与 GC Root (变量 a) 断开了连接，所以 a 原来指向的对象会被回收，而由于我们给 s 赋值了变量的引用，s 在此时是类静态属性引用，充当了 GC Root 的作用，它指向的对象依然存活！

```java
public  class Test {
    public static Test s;
    public static void main(String[] args) {
        Test a = new Test();
        a.s = new Test();
        a = null;
    }
}
```

> **方法区中常量引用的对象**

如下代码所示，常量 s 指向的对象并不会因为 a 指向的对象被回收而回收

```java
public  class Test {
	public  static  final Test s = new Test();
    public static void main(String[] args) {
        Test a = new Test();
        a = null;
    }
}
```

> **本地方法栈中 JNI 引用的对象**

这是简单给不清楚本地方法为何物的童鞋简单解释一下：所谓本地方法就是一个 **java 调用非 java 代码的接口**，该方法并非 Java 实现的，可能由 C 或 Python等其他语言实现的， **Java 通过 JNI 来调用本地方法**，而本地方法是以**库文件**的形式存放的（**在 WINDOWS 平台上是 DLL 文件形式**，**在 UNIX 机器上是 SO 文件形式**）。通过调用本地的库文件的内部方法，使 JAVA 可以实现和本地机器的紧密联系，调用系统级的各接口方法

当**调用 Java 方法**时，**虚拟机会创建一个栈桢并压入 Java 栈**，而当它**调用的是本地方法**时，虚拟机会保持 Java 栈不变，不会在 Java 栈祯中压入新的祯，虚拟机只是简单地**动态连接并直接调用指定的本地方法**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXuhbd9HCLBIYRBekQqics7XBrj0vVUVSPTHTibJbutrdGRWib1MlDd16JQ/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

### 垃圾回收算法

上一节我们知道了可以通过可达性算法来识别哪些数据是垃圾，那该怎么对这些垃圾进行回收呢。主要有以下几种方式方式

* 标记清除算法
* 复制算法
* 标记整理算法

> **标记清除算法**

步骤很简单

1. 先根据可达性算法**标记**出相应的可回收对象（图中黄色部分）
2. 对可回收的对象进行回收

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXUakPW6I2h1xoQpXMevUzM2eGwG0ia9YrPL1QCDvXaW1mwa3CrvHeRibA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 80%;" />

问题：**内存碎片**

> **复制算法**

把堆等分成两块区域, A 和 B，区域 A 负责分配对象，区域 B 不分配, 对区域 A 使用以上所说的标记法把存活的对象标记出来，然后把区域 A 中存活的对象都复制到区域 B（存活对象都依次**紧邻排列**）最后把 A 区对象全部清理掉释放出空间，这样就解决了内存碎片的问题了。

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200826213139.png" style="zoom: 67%;" />

不过复制算法的缺点很明显，比如给堆分配了 500M 内存，结果只有 250M 可用，**空间平白无故减少了一半**！这肯定是不能接受的！另外每次回收也要**把存活对象移动到另一半**，**效率低下**（我们可以想想删除数组元素再把非删除的元素往一端移，效率显然堪忧）

> **标记整理算法**

前面两步和标记清除法一样，不同的是它在标记清除法的基础上添加了一个整理的过程 ，即将所有的存活对象都往一端移动,紧邻排列（如图示），再清理掉另一端的所有区域，这样的话就解决了内存碎片的问题。

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXS35icx19mib3dZCEm3fOdlw2clYbWTJthU5ibn9Ztc8kQuaosxOachQWw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 33%;" />

但是缺点也很明显：每进一次垃圾清除都要频繁地移动存活的对象，效率十分低下。

> **分代收集算法**

分代收集算法根据**对象存活周期的不同**将堆分成新生代和老生代（Java8以前还有个永久代）,默认比例为 1 : 2，新生代又分为 Eden 区， from Survivor 区（简称S0），to Survivor 区(简称 S1),三者的比例为 8: 1 : 1，这样就可以根据新老生代的特点选择最合适的垃圾回收算法，我们把新生代发生的 GC 称为 Young GC（也叫 Minor GC）,老年代发生的 GC 称为 Old GC（也称为 Full GC）。

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200826213457.png" style="zoom:50%;" />

**工作原理**

1. **对象在新生代的分配与回收**

   由以上的分析可知，大部分对象在很短的时间内都会被回收，对象一般分配在 Eden 区

   <img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXEJ4Z6GYhCxxwJSTqVIeRoq68CO11TzEgKKibhtgRlvhd2oXoAPJsib7g/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

   当 Eden 区将满时，触发 Minor GC

   <img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXlpg7Rtt17cDV7PxeuKep7sMX3WhwP2KggiaG49G5UZmlk54zzNLKWcg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

   大部分对象在短时间内都会被回收, 所以经过 Minor GC 后只有少部分对象会存活，它们会被移到 S0 区（这就是为啥空间大小  Eden: S0: S1 = 8:1:1, Eden 区远大于 S0,S1 的原因，因为在 Eden 区触发的 Minor GC 把大部对象（接近98%）都回收了,只留下少量存活的对象，此时把它们移到 S0 或 S1 绰绰有余）同时对象年龄加一（对象的年龄即发生 Minor GC 的次数），最后把 Eden 区对象全部清理以释放出空间,动图如下

   <img src="https://mmbiz.qpic.cn/mmbiz_gif/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXdZ6EYjicg7nuibII5xZBwiaGq2S454iaFRibc5Z1ibiaLKnYyldeQyzAO9ibRg/640?wx_fmt=gif&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt="img" style="zoom: 80%;" />

   当触发下一次 Minor GC 时，会把 Eden 区的存活对象和 S0（或S1） 中的存活对象（S0 或 S1 中的存活对象经过每次 Minor GC 都可能被回收）一起移到 S1（Eden 和 S0 的存活对象年龄+1）, 同时清空 Eden 和 S0 的空间

   <img src="https://mmbiz.qpic.cn/mmbiz_gif/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXlN3YAWPibCzl5Lw4ZWKHAU8f2YZFEO3ugjYbQ4fSfUCFyU2Mmp904kg/640?wx_fmt=gif&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt="img" style="zoom: 80%;" />

   若再触发下一次 Minor GC，则重复上一步，只不过此时变成了 从 Eden，S1 区将存活对象复制到 S0 区,每次垃圾回收, S0, S1 角色互换，都是从 Eden ,S0(或S1) 将存活对象移动到 S1(或S0)。也就是说在 Eden 区的垃圾回收我们采用的是**复制算法**，因为在 Eden 区分配的对象大部分在 Minor GC 后都消亡了，只剩下极少部分存活对象（这也是为啥 Eden:S0:S1 默认为 8:1:1 的原因），S0,S1 区域也比较小，所以最大限度地降低了复制算法造成的对象频繁拷贝带来的开销。

2. **对象何时晋升老年代**

   * **当对象的年龄达到了我们设定的阈值**，则会从S0（或S1）晋升到老年代

     <img src="https://mmbiz.qpic.cn/mmbiz_gif/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXfflQYTnxmA64gbBCogBQncpxu0AumticAib02Cv8oEdafymtcVSwPBQQ/640?wx_fmt=gif&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt="img" style="zoom:80%;" />

     如图示：年龄阈值设置为 15， 当发生下一次 Minor GC 时，S0 中有个对象年龄达到 15，达到我们的设定阈值，晋升到老年代！

   * **大对象** 当某个对象分配需要大量的连续内存时，此时对象的创建不会分配在 Eden 区，会直接分配在老年代，因为如果把大对象分配在 Eden 区, Minor GC 后再移动到 S0,S1 会有很大的开销（对象比较大，复制会比较慢，也占空间），也很快会占满 S0,S1 区，所以干脆就直接移到老年代
   * 还有一种情况也会让对象晋升到老年代，即在 **S0（或S1） 区相同年龄的对象大小之和大于 S0（或S1）空间一半以上**时，则**年龄大于等于该年龄的对象也会晋升到老年代**。

3. **空间分配担保**

   在发生 MinorGC 之前，虚拟机会先检查**老年代最大可用的连续空间**是否大于**新生代所有对象的总空间**，如果**大于**，那么Minor GC 可以确保是**安全**的，如果**不大于**，那么虚拟机会查看 HandlePromotionFailure 设置值**是否允许担保失败**。如果允许，那么会继续检查**老年代最大可用连续空间**是否大于**历次晋升到老年代对象的平均大小**，如果**大于**则进行 **Minor GC**，**否则可能进行一次 Full GC**。

4. **Stop The World**

   如果老年代满了，会触发 Full GC, Full GC 会同时回收新生代和老年代（即**对整个堆进行GC**），它会导致 Stop The World（简称 STW），造成挺大的性能开销。

   什么是 STW ？所谓的 STW, 即在 GC（minor GC 或 Full GC）期间，**只有垃圾回收器线程在工作**，**其他工作线程则被挂起**。

   <img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXY4aYK4dSJRXgwl6UJeVzNalZLcj4ExZOiasRpobMqh7nLJdJvUVO5eg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

   *画外音：为啥在垃圾收集期间其他工作线程会被挂起？想象一下，你一边在收垃圾，另外一群人一边丢垃圾，垃圾能收拾干净吗。*

   一般 Full GC 会导致工作线程停顿时间过长（因为Full GC 会清理**整个堆**中的不可用对象，一般要花较长的时间），如果在此 server 收到了很多请求，则会被拒绝服务！所以我们要尽量减少 Full GC（Minor GC 也会造成 STW,但只会触发轻微的 STW,因为 Eden 区的对象大部分都被回收了，只有极少数存活对象会通过复制算法转移到 S0 或 S1 区，所以相对还好）。

   现在我们应该明白把新生代设置成 Eden, S0，S1区或者给对象设置年龄阈值或者默认把新生代与老年代的空间大小设置成 1:2 都是为了**尽可能地避免对象过早地进入老年代，尽可能晚地触发 Full GC**。想想新生代如果只设置 Eden 会发生什么，后果就是每经过一次 Minor GC，存活对象会过早地进入老年代，那么老年代很快就会装满，很快会触发 Full GC，而对象其实在经过两三次的 Minor GC 后大部分都会消亡，所以有了 S0,S1的缓冲，只有少数的对象会进入老年代，老年代大小也就不会这么快地增长，也就避免了过早地触发 Full GC。

   由于 Full GC（或Minor GC） 会影响性能，所以我们要在一个合适的时间点发起 GC，这个时间点被称为 **Safe Point**，这个时间点的选定**既不能太少以让 GC 时间太长导致程序过长时间卡顿**，**也不能过于频繁以至于过分增大运行时的负荷**。一般当**线程在这个时间点上状态是可以确定**的，如确定 GC Root 的信息等，可以使 JVM 开始安全地 GC。Safe Point 主要指的是以下特定位置：

   * 循环的末尾
   * 方法返回前
   * 调用方法的 call 之后
   * 抛出异常的位置

另外需要注意的是由于新生代的特点（大部分对象经过 Minor GC后会消亡）， **Minor GC 用的是复制算法**，而在老生代由于对象比较多，占用的空间较大，使用复制算法会有较大开销（复制算法在对象存活率较高时要进行多次复制操作，同时浪费一半空间）所以根据老生代特点，在**老年代进行的 GC 一般采用的是标记整理法**来进行回收。

### 垃圾收集器种类

主要有以下垃圾收集器

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXzJK4ic99AOYK1D1icmJan8QMHcUNu8b6fRT34xicv52j6ticmgQcNbwhsA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

* 在新生代工作的垃圾回收器：Serial, ParNew, ParallelScavenge
* 在老年代工作的垃圾回收器：CMS，Serial Old, Parallel Old
* 同时在新老生代工作的垃圾回收器：G1

图片中的垃圾收集器如果存在连线，则代表它们之间可以配合使用，接下来我们来看看各个垃圾收集器的具体功能

#### 新生代收集器

> **Serial 收集器**

Serial 收集器是**工作在新生代**的，**单线程的垃圾收集器**，单线程意味着它只会使用一个 CPU 或一个收集线程来完成垃圾回收，不仅如此，还记得我们上文提到的 STW 了吗，它在进行垃圾收集时，其他用户线程会暂停，直到垃圾收集结束，也就是说在 GC 期间，此时的应用不可用。

看起来单线程垃圾收集器不太实用，不过我们需要知道的任何技术的使用都不能脱离场景，在 **Client 模式**下，它简单有效（与其他收集器的单线程比），对于限定单个 CPU 的环境来说，**Serial 单线程模式无需与其他线程交互**，**减少了开销**，**专心做 GC 能将其单线程的优势发挥到极致**，另外在用户的桌面应用场景，分配给虚拟机的内存一般不会很大，收集几十甚至一两百兆（仅是新生代的内存，桌面应用基本不会再大了），STW 时间可以控制在一百多毫秒内，只要不是频繁发生，这点停顿是可以接受的，所以对于运行在 Client 模式下的虚拟机，Serial 收集器是新生代的默认收集器

> **ParNew 收集器**

**ParNew 收集器是 Serial 收集器的多线程版本**，除了使用多线程，其他像收集算法,STW,对象分配规则，回收策略与 Serial 收集器完成一样，在底层上，这两种收集器也共用了相当多的代码，它的垃圾收集过程如下

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXaCHQlQ54mGjmJ5ac4D700X8nFBcMqpayjGWdrdhNcMPY89NZpPvLSA/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:50%;" />

**ParNew 主要工作在 Server 模式**，我们知道服务端如果接收的请求多了，响应时间就很重要了，**多线程可以让垃圾回收得更快**，也就是**减少了 STW 时间**，能**提升响应时间**，所以是许多运行在 Server 模式下的虚拟机的首选新生代收集器，另一个与性能无关的原因是因为除了 Serial  收集器，**只有它能与 CMS 收集器配合工作**，CMS 是一个划时代的垃圾收集器，是真正意义上的**并发收集器**，它第一次实现了垃圾收集线程与用户线程（基本上）同时工作，它采用的是传统的 GC 收集器代码框架，与 Serial,ParNew 共用一套代码框架，所以能与这两者一起配合工作，而后文提到的 Parallel Scavenge 与 G1 收集器没有使用传统的 GC 收集器代码框架，而是另起炉灶独立实现的，另外一些收集器则只是共用了部分的框架代码,所以无法与 CMS 收集器一起配合工作。

在多 CPU 的情况下，由于 ParNew 的多线程回收特性，毫无疑问垃圾收集会更快，也能有效地减少 STW 的时间，提升应用的响应速度。

> **Parallel Scavenge 收集器**

Parallel Scavenge 收集器也是一个使用**复制算法**，**多线程**，工作于新生代的垃圾收集器，看起来功能和 ParNew 收集器一样，它有啥特别之处吗

**关注点不同**，CMS 等垃圾收集器关注的是尽可能缩短垃圾收集时用户线程的停顿时间，而 Parallel Scavenge 目标是达到一个**可控制的吞吐量**（**吞吐量 = 运行用户代码时间 / （运行用户代码时间+垃圾收集时间）**），也就是说 **CMS 等垃圾收集器更适合用到与用户交互的程序，因为停顿时间越短，用户体验越好**，而 **Parallel Scavenge 收集器关注的是吞吐量，所以更适合做后台运算等不需要太多用户交互的任务**。

Parallel Scavenge 收集器提供了两个参数来精确控制吞吐量，分别是**控制最大垃圾收集时间**的 -XX:MaxGCPauseMillis 参数及**直接设置吞吐量大小**的 -XX:GCTimeRatio（默认99%）

除了以上两个参数，还可以用 Parallel Scavenge 收集器提供的第三个参数 **-XX:UseAdaptiveSizePolicy**，开启这个参数后，就不需要手工指定新生代大小，Eden 与 Survivor 比例（SurvivorRatio）等细节，只需要设置好基本的堆大小（-Xmx 设置最大堆）,以及最大垃圾收集时间与吞吐量大小，虚拟机就会根据当前系统运行情况收集监控信息，**动态调整这些参数**以尽可能地达到我们设定的最大垃圾收集时间或吞吐量大小这两个指标。自适应策略也是 Parallel Scavenge  与 ParNew 的重要区别！

#### 老年代收集器

> **Serial Old 收集器**

上文我们知道， Serial 收集器是工作于新生代的单线程收集器，与之相对地，**Serial Old 是工作于老年代的单线程收集器**，此收集器的主要意义在于给 Client 模式下的虚拟机使用，如果在 Server 模式下，则它还有两大用途：一种是在 JDK 1.5 及之前的版本中与 Parallel Scavenge 配合使用，另一种是作为 CMS 收集器的后备预案,在并发收集发生 Concurrent Mode Failure 时使用（后文讲述）,它与 Serial 收集器配合使用示意图如下

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXXsEhJL7oahZ9JjKw7EKJ1rr1ic6fPTrEzLia8Ede4T2uqZdOUeqrf0nw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

> **Parallel Old 收集器**

Parallel Old 是相对于 Parallel Scavenge 收集器的老年代版本，使用**多线程和标记整理法**，两者组合示意图如下，这两者的组合由于都是多线程收集器，真正实现了「吞吐量优先」的目标

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXyM3YO3k6awoMwkkZLT6CSBnuwaYtrBOlaqBzkbas4Jf5OGolSh5lIg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:67%;" />

> **CMS 收集器**

CMS 收集器是以实现最短 STW 时间为目标的收集器，如果应用很重视服务的响应速度，希望给用户最好的体验，则 CMS 收集器是个很不错的选择！

我们之前说老年代主要用标记整理法，而 CMS 虽然工作于老年代，但采用的是**标记清除法**，主要有以下四个步骤

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXBk0HdaA4x1FqkobKRxdEribRMQn86zDt9FpceO1icmLwd6oichSjk4ibZQ/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

从图中可以的看到初始标记和重新标记两个阶段会发生 STW，造成用户线程挂起，不过**初始标记仅标记 GC Roots 能关联的对象，速度很快**，**并发标记是进行 GC Roots  Tracing 的过程**，**重新标记是为了修正并发标记期间因用户线程继续运行而导致标记产生变动的那一部分对象的标记记录**，这一阶段停顿时间一般比初始标记阶段稍长，但**远比并发标记时间短**。

整个过程中耗时最长的是并发标记和标记清理，不过这两个阶段用户线程都可工作，所以不影响应用的正常使用，所以总体上看，可以认为 CMS 收集器的内存回收过程是与用户线程一起并发执行的。

但是 CMS 收集器远达不到完美的程度，主要有以下三个缺点

- CMS 收集器对 CPU 资源非常敏感  原因也可以理解，比如本来我本来可以有 10 个用户线程处理请求，现在却要分出 3 个作为回收线程，吞吐量下降了30%，**CMS 默认启动的回收线程数是 （CPU数量+3）/ 4**, 如果 CPU 数量只有一两个，那吞吐量就直接下降 50%,显然是不可接受的
- CMS 无法处理**浮动垃圾**（Floating Garbage）,可能出现 「Concurrent Mode Failure」而导致另一次 Full GC 的产生，由于**在并发清理阶段用户线程还在运行**，所以清理的同时新的垃圾也在不断出现，**这部分垃圾只能在下一次 GC 时再清理掉（即浮云垃圾）**，同时在**垃圾收集阶段用户线程也要继续运行**，就需要预留足够多的空间要确保用户线程正常执行，这就意味着 CMS 收集器不能像其他收集器一样等老年代满了再使用，JDK 1.5 默认当老年代使用了 68% 空间后就会被激活，当然这个比例可以通过 -XX:CMSInitiatingOccupancyFraction 来设置，但是如果设置地太高很容易导致在 CMS 运行期间预留的内存无法满足程序要求，会导致 **Concurrent Mode Failure** 失败，这时会启用 Serial Old 收集器来重新进行老年代的收集，而我们知道 Serial Old 收集器是单线程收集器，这样就会导致 STW 更长了。
- CMS 采用的是标记清除法，上文我们已经提到这种方法会产生大量的**内存碎片**，这样会给大内存分配带来很大的麻烦，如果无法找到足够大的连续空间来分配对象，将会触发 Full GC，这会影响应用的性能。当然我们可以开启 -XX:+UseCMSCompactAtFullCollection（默认是开启的），用于在 CMS 收集器顶不住要进行 FullGC 时开启内存碎片的合并整理过程，内存整理会导致 STW，停顿时间会变长，还可以用另一个参数 -XX:CMSFullGCsBeforeCompation 用来设置执行多少次不压缩的 Full GC 后跟着带来一次带压缩的。

> **G1（Garbage First） 收集器**

G1 收集器是面向服务端的垃圾收集器，被称为驾驭一切的垃圾回收器，主要有以下几个特点

* 像 CMS 收集器一样，能与应用程序线程并发执行。
* 整理空闲空间更快。
* 需要 GC 停顿时间更好预测。
* 不会像 CMS 那样牺牲大量的吞吐性能。
* 不需要更大的 Java Heap

与 CMS 相比，它在以下两个方面表现更出色

1. **运作期间不会产生内存碎片**，**G1 从整体上看采用的是标记-整理法**，**局部（两个 Region）上看是基于复制算法实现的**，两个算法都不会产生内存碎片，收集后提供规整的可用内存，这样有利于程序的长时间运行。
2. 在 STW 上建立了**可预测**的停顿时间模型，用户可以指定期望停顿时间，G1 会将停顿时间控制在用户设定的停顿时间以内。

为什么G1能建立可预测的停顿模型呢，主要原因在于 G1 对堆空间的分配与传统的垃圾收集器不一器，传统的内存分配就像我们前文所述，是连续的，分成新生代，老年代，新生代又分 Eden,S0,S1,如下

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdXunqRMuhq6FZbcKicNibcvKmeTwic05ordoZ2wgW4hH5KtMlKu8DvQibE5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而 G1 各代的存储地址不是连续的，每一代都使用了 n 个不连续的大小相同的 Region，每个 Region 占有一块连续的虚拟内存地址，如图示

![img](https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdX7eYnibUFHuo7AibZyLsaHq6tWpWnwLO3jj1rk3mThrAPSsdmVMrgzlLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

除了和传统的新老生代，幸存区的空间区别，Region还多了一个H，它代表Humongous，这表示这些Region存储的是巨大对象（humongous object，H-obj），即大小大于等于region一半的对象，这样超大对象就直接分配到了老年代，防止了反复拷贝移动。那么 G1 分配成这样有啥好处呢？

传统的收集器如果发生 Full GC 是对整个堆进行全区域的垃圾收集，而**分配成各个 Region 的话**，**方便 G1 跟踪各个 Region 里垃圾堆积的价值大小**（回收所获得的空间大小及回收所需经验值），这样**根据价值大小维护一个优先列表**，**根据允许的收集时间**，**优先收集回收价值最大的 Region**,也就**避免了整个老年代的回收**，也就减少了 STW 造成的停顿时间。同时由于只收集部分 Region,可就做到了 STW 时间的可控。

G1 收集器的工作步骤如下

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

<img src="https://mmbiz.qpic.cn/mmbiz_png/OyweysCSeLUrYqPicjVwjuMChPrPicNHdX8S08rDRVliaVW84ibCM9kzCtIxCIYqFGGbiad6VPBV9qSZEqOJtTuyyicQ/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

可以看到整体过程与 CMS 收集器非常类似，筛选阶段会根据各个 Region 的回收价值和成本进行排序，根据用户期望的 GC 停顿时间来制定回收计划。



### 四种引用

**强引用**

```java
Object obj = new Object(); // 强引用
obj = null; //取消强引用
```

只要强引用在，垃圾收集器永远不会回收被引用的对象。即使当前内存空间不足，JVM 也不会回收它，而是抛出 OutofMemoryError 错误，使程序异常终止。如果想中断强引用和某个对象之间的关联，可以显式地将引用赋值为 null，这样 JVM 在合适的时间就会回收该对象

**软引用**

```java
SoftReference<String> softName = new SoftReference<>("张三");
```

在使用软引用时，如果**内存的空间足够**，软引用就能继续被使用，而**不会被垃圾回收器回收**；只有在**内存空间不足**时，**软引用才会被垃圾回收器回收**

**弱引用**

```java
WeakReference<String> weakName = new WeakReference<String>("hello");
```

具有弱引用的对象拥有更短暂的生命周期，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了**只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存**。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象

**虚引用**

如果一个对象仅持有虚引用，那么它相当于没有引用，在任何时候都可能被垃圾回收器回收

虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动

```java
ReferenceQueue<String> queue = new ReferenceQueue<String>();
PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
```



### 类加载机制

类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

#### 类的生命周期

一个类的完整生命周期如下：

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200721093214.png)

#### 类加载过程

系统加载 .class 文件主要三步：**加载 -> 连接 -> 初始化**

**连接**过程又可以分为三步：**验证 -> 准备 -> 解析**

##### 加载

加载是类加载的第一个阶段，完成以下三件事：

1. 通过全类名获取定义此类的二进制字节流
2. 将该字节流表示的静态存储结构转换为方法区的运行时数据结构
3.  在内存中生成一个代表该类的 Class 对象（**这个对象包含了完整的类的结构信息**），作为方法区中该类各种数据的访问入口

##### 验证

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

##### 准备

**类变量** 是被 static 修饰的变量，准备阶段为**类变量分配内存**并**设置初始值**，使用的是方法区的内存

实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次

初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0 ，例如下面的常量 value 被初始化为 123 而不是 0

```java
public static final int value = 123;
```

##### 解析（静态链接）

解析阶段是虚拟机将常量池内的 **符号引用** 替换为 **直接引用** 的过程。

符号引用就是一组符号来描述目标，可以是任何**字面量**；直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用

##### 初始化

初始化是类加载的最后一步，也是真正执行类中定义的 Java 程序代码（字节码），初始化阶段是执行类构造器 `<clinit>()` 方法的过程

`<clinit>()`是由编译器自动收集类中所有**类变量的赋值动作**和**静态语句块**中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。

> 特别注意的是，静态语句块只能**访问到定义在它之前的类变量**，**定义在它之后的类变量只能赋值**，不能访问。例如以下代码：

```java
public class Test {
    static {
        i = 0;					// 给变量赋值可以正常编译通过
        System.out.print(i);	// 这句编译器会提式‘illegal forward reference’
    }
    static int i = 0;
}
```

由于父类的 `<clinit>()`方法先执行，也就意味着**父类中定义的静态语句块的执行要优先于子类**。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
    System.out.println(Sub.B);	// 2
}
```

接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 `<clinit>()`方法。但接口与类不同的是，执行接口的 `<clinit>()` 方法不需要先执行父接口的 `<clinit>()` 方法。只有当父接口中定义的静态变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 `<clinit>()`方法

**`<init>():`**对象构造时用以初始化对象的，构造方法以及构造代码块中的代码

###### 示例

```java
public class Test {
    private static Test instance;

    static {
        System.out.println("static开始");
        // 下面这句编译器报错，非法向前引用
        // System.out.println("x=" + x);
        instance = new Test();
        System.out.println("static结束");
    }

    public Test() {
        System.out.println("构造器开始");
        System.out.println("x=" + x + ";y=" + y);
        // 构造器可以访问声明于他们后面的静态变量
        // 因为静态变量在类加载的准备阶段就已经分配内存并初始化0值了
        // 此时 x=0，y=0
        x++;
        y++;
        System.out.println("x=" + x + ";y=" + y);
        System.out.println("构造器结束");
    }

    public static int x = 6;
    public static int y;

    public static Test getInstance() {
        return instance;
    }

    public static void main(String[] args) {
        Test obj = Test.getInstance();
        System.out.println("x=" + obj.x);
        System.out.println("y=" + obj.y);
    }
}

/**
 *	static开始
 *	构造器开始
 *	x=0;y=0
 *	x=1;y=1
 *	构造器结束
 *	static结束
 *	x=6
 * 	y=1
 */
```

虚拟机首先执行的是类加载初始化过程中的 `<clinit>()`方法，也就是静态变量赋值以及静态代码块中的方法，如果`<clinit>()`方法中触发了对象的初始化，也就是`<init>()`方法，那么会进入执行`<init>()`方法，执行`init<>()`方法完成之后，再回来继续执行`<clinit>()`方法

###### 类初始化时机

* 遇到 **new、getstatic、putstatic、invokestatic** 这四条字节码指令时，比如使用 new 关键字实例化对象的时候；读取或赋值类的静态变量；调用类的静态方法
* 使用 java.lang.reflect 包的方法对类进行**反射**调用的时候，如果类没有进行初始化，则需要先触发初始化
* 当初始化类的时候，发现其父类还没有进行初始化，需要先触发其**父类**的初始化
* 当虚拟机启动时，用户需要定义一个要执行的主类（包含 main 方法的类），虚拟机会先初始化**主类**
* MethodHandle 和 VarHandle 可以看作是轻量级的反射调用机制，而要想使用这2个调用， 就必须先使用 findStaticVarHandle 来初始化要调用的类

###### 类的被动引用（不会发生类的初始化）

* 当访问一个**静态域**时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
* 通过数组定义类引用，不会触发此类的初始化
* 引用常量不会触发此类的初始化（常量在准备阶段就存入调用类的常量池中了）



### 类与类加载器

两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间

这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果为 true，也包括使用 instanceof 关键字做对象所属关系判定结果为 true

#### 类加载器分类

从 Java 虚拟机角度，只存在以下两种不同的类加载器：

* 启动类加载器（Bootstrap ClassLoader），使用 C++ 实现，是虚拟机自身的一部分
* 所有其他类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader

从 Java 开发人员的角度看，类加载器可以划分得更细致一些：

* 启动类加载器（Bootstrap ClassLoader）：此类加载器负责将存放在 <JRE_HOME>\lib 目录中的，或者被 `-Xbootclasspath` 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即时放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可
* 扩展类加载器（Extension ClassLoader）：这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 <JAVA_HOME>\lib\ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器
* 应用程序类加载器（Application ClassLoader）：这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器

### 双亲外派机制

应用程序是由三种类加载器相互配合从而实现类加载，除此之外还可以加入自己定义的类加载器

下图展示了类加载器之间的层次关系，称为双亲委派模型。该模型要求除了顶层的启动类加载器外，其他的类加载器都要有自己的父类加载器。这里的父子关系一般是通过组合关系来实现的，而不是继承关系

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200724225842.png) 

##### 工作过程

**一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载**

##### 好处

使得 **Java 类**随着它的类加载器一起具有一种带有**优先级**的层次关系，从而使得基础类得到统一

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。 rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object