## JVM

### 运行时数据区

**JDK 1.8 之前：**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727104854.png" style="zoom: 25%;" />

**JDK 1.8：**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727105335.png" style="zoom:25%;" />

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

在使用软引用时，如果内存的空间足够，软引用就能继续被使用，而不会被垃圾回收器回收；只有在内存空间不足时，软引用才会被垃圾回收器回收

**弱引用**

```java
WeakReference<String> weakName = new WeakReference<String>("hello");
```

具有弱引用的对象拥有更短暂的生命周期，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象

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
2. 将该字节流表示的静态存储结构转换为方法区的运行时存储结构
3.  中生成一个代表该类的 Class 对象（**这个对象包含了完整的类的结构信息**），作为方法区中该类各种数据的访问入口

##### 验证

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

##### 准备

**类变量** 是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存

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

符号引用就是一组符号来描述目标，可以是任何字面量；直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用

##### 初始化

初始化是类加载的最后一步，也是真正执行类中定义的 Java 程序代码（字节码），初始化阶段是执行类构造器 `<clinit>()` 方法的过程

`<clinit>()`是由编译器自动收集类中所有**类变量**的赋值动作和**静态语句块**中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。

> 特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如以下代码：

```java
public class Test {
    static {
        i = 0;					// 给变量赋值可以正常编译通过
        System.out.print(i);	// 这句编译器会提式‘illegal forward reference’
    }
    static int i = 0;
}
```

由于父类的 `<clinit>()`方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：

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

**`<init>()`**

对象构造时用以初始化对象的，构造方法以及构造代码块中的代码

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

* 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
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

一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载

##### 好处

使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。 rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object