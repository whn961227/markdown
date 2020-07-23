## 基础

### Java 基本功

#### JVM、JDK、JRE

##### JVM

**什么是字节码？采用字节码的好处是什么**

> 在 Java 中，JVM 可以理解的代码就叫做 **字节码**（即扩展名为 .class 的文件），它只面向虚拟机。

Java 程序从源代码到运行一般有下面 3 步：

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200720225705419.png" alt="image-20200720225705419" style="zoom:80%;" />

格外注意 .class -> 机器码 这一步。在这一步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的（也就是所谓的热点代码），所以后面引进了 **JIT 编译器**，而 JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。机器码的运行效率肯定是高于 Java 解释器的。这也解释了为什么 Java 是编译与解释共存的语言

**总结：**

* Java 虚拟机（JVM）是运行 Java 字节码的虚拟机

* JVM 有针对不同系统的特定实现（Windows、Linux、macOS），目的是使用相同的字节码，他们都会给出相同的结果
* 字节码和不同系统的 JVM 实现是 Java 语言 “一次编译，随处可以运行” 的关键所在

##### JDK 和 JRE

JDK 是 Java Development Kit，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器（Javac）和工具（如 Javadoc 和 jdb）。它能够创建和编译程序

JRE 是 Java 运行时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java虚拟机（JVM），Java 类库，Java 命令和其他的一些基础构件。但是，它不能用于创建新程序



#### 为什么说 Java 语言“编译与解释并存”？

高级编程语言按照程序的执行方式分为 **编译型** 和 **解释型** 两种。

**编译型语言** 是指编译器针对特定的操作系统将源代码一次性翻译成可被该平台执行的机器码

**解释型语言** 是指解释器对源程序逐行解释成特定平台的机器码并立即执行

Java 程序要经过先编译，后解释两个步骤，经过编译步骤，生成字节码（.class文件），这种字节码必须由 Java 解释器来解释执行



### Java 中的四种代码块

#### 简介

* **普通代码块：**定义在方法中（不管是静态方法还是普通方法） -> 方法被调用时执行
* **构造代码块：**直接定义在类中，但是没有 static -> 优先于构造方法执行，晚于静态块执行
* **静态代码块：**定义在类中，且有 static -> 最先被执行，且对于一个类的多个对象，只执行一次
* **同步代码块：**synchronized 关键字修饰的，和线程相关

#### 静态代码块和构造代码块的区别

**相同点：**都是 JVM 加载类后且在构造方法前执行，在类中可定义多个，一般在代码块中对一些 static 变量进行赋值

**不同点：**静态代码块在构造代码块前执行。静态代码块只在第一次 new 时执行一次，之后不再执行。而构造代码块每 new 一次就执行一次

#### 示例

```java
public class Person {
    static{
        System.out.println("1.我是静态块，优先于构造块执行！并且只有创建第一个对象的时候执行一次！");
    }
    {
        System.out.println("2.我是构造块，优先于构造方法执行！每创建一个对象执行一次！");
    }
    public Person() {
        System.out.println("3.我是构造方法，每创建一个对象执行一次");
    }
    public void function1(){
        System.out.println("我是非静态方法中的普通代码块，方法被调用时执行！");
    }
    public static void function2(){
        System.out.println("我是静态方法中的普通代码块，方法被调用时执行，晚于静态块执行！");
    }
}

public class HelloWrold {
    public static void main(String[] args) {
        new Person().function1();
        new Person().function1();
        System.out.println("=================");
        Person.function2();
        Person.function2();
    }
}

/**
 *	1.我是静态块，优先于构造块执行！并且只有创建第一个对象的时候执行一次！
 *	2.我是构造块，优先于构造方法执行！每创建一个对象执行一次！
 *	3.我是构造方法，每创建一个对象执行一次
 *	我是非静态方法中的普通代码块，方法被调用时执行！
 *	2.我是构造块，优先于构造方法执行！每创建一个对象执行一次！
 *	3.我是构造方法，每创建一个对象执行一次
 * 	我是非静态方法中的普通代码块，方法被调用时执行！
 *	=================
 * 	我是静态方法中的普通代码块，方法被调用时执行，晚于静态块执行！
 * 	我是静态方法中的普通代码块，方法被调用时执行，晚于静态块执行！
 */
```

可以看出：静态块总是最先执行的，并且只有在创建类的第一个实例的时候才会执行一次；第二执行的是构造块；第三执行的是构造方法

#### 构造代码块和构造函数的区别

构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容



### 泛型

即“参数化类型”，就是将类型由原来具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）

在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别称为泛型类、泛型接口和泛型方法

它提供了编译期的类型安全

#### Java 的泛型是如何工作的？什么是类型擦除？

泛型是通过类型擦除实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息。你无法在运行时访问到类型参数，因为编译器已经把泛型类型转换成了原始类型。

#### 什么是泛型中的限定通配符和非限定通配符？

限定通配符对类型进行了限制。有两种限定通配符，一种是 <? extends T>，它确保类型必须是 T 的子类来设定类型的上界，另一种是 <? super T>，它通过确保类型必须是 T 的父类来设定类型的下届。泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。另一方面 < ? > 表示了非限定通配符，因为 < ? > 可以用任意类型来替代。 



### 值传递和引用传递

> **值传递：**是指在调用函数时将实参**复制**一份传递到形参中，这样在函数中如果对参数进行修改，将不会影响到实参
>
> **引用传递：**是指在调用函数时将实参的地址**直接**传递到形参中，那么在函数中对参数所进行的修改，将影响到实参

**总结**

Java 都是值传递，关键看这个值是什么，简单变量就是复制了具体值，引用变量就是复制了地址



### == 和 equals() 的区别

**==：**作用是判断两个对象的地址是不是相等。即判断两个对象是否是同一个对象（基本数据类型 == 比较的是值，引用数据类型 == 比较的是内存地址）

**equals：**作用是判断两个对象是否相等，它不能用于比较基本数据类型的变量。

Object 类 equals() 方法：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

equals() 方法存在两种使用情况：

* 类没有重写 equals() 方法。等价于通过 == 比较对象
* 类重写了 equals() 方法。一般重写来比较两个对象的内容，如果内容相等，则返回 true



### hashCode() 和 equals()

#### 为什么重写 equals 必须重写 hashCode 方法？

如果两个对象相等，equals 方法返回 true，则 hashcode 一定相同，hashCode() 的默认行为是对堆上的对象产生哈希值，如果没有重写 hashCode()，则 class 的两个对象无论如何都不会相等



### 重载和重写的区别

>重载就是同样的方法能够根据输入数据的不同，做出不同的处理
>
>重写就是当子类继承自父类的相同方法，输入数据一样，但要实现和父类不同的功能时，就要覆盖父类方法



### 基本数据类型

| 基本类型 | 字节 | 包装类    |
| -------- | ---- | --------- |
| short    | 2    | Short     |
| int      | 4    | Integer   |
| long     | 8    | Long      |
| float    | 4    | Float     |
| double   | 8    | Double    |
| byte     | 1    | Byte      |
| char     | 2    | Character |
| boolean  | 未定 | Boolean   |

#### 自动装箱和拆箱

* **装箱：**根据基本类型创建它们对应的包装类型
* **拆箱：**将包装类型转换为基本数据类型

> 通过 valueOf 方法创建 Integer 对象时，如果数值在 [-128, 127] 之间，便返回指向 IntegerCache.cache 中已经存在的对象的引用；否则创建一个新的 Integer 对象

```java
Integer i1 = 40;
Integer i2 = 40;
Integer i3 = 0;
Integer i4 = new Integer(40);
Integer i5 = new Integer(40);
Integer i6 = new Integer(0);

System.out.println("i1=i2   " + (i1 == i2));
System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
System.out.println("i1=i4   " + (i1 == i4));
System.out.println("i4=i5   " + (i4 == i5));
System.out.println("i4=i5+i6   " + (i4 == i5 + i6));   
System.out.println("40=i5+i6   " + (40 == i5 + i6)); 

/**
当 == 运算符的两个操作数都是包装类型的引用，则比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）

i1=i2   true
i1=i2+i3   true
i1=i4   false
i4=i5   false
i4=i5+i6   true
40=i5+i6   true
 */
```



### 深拷贝和浅拷贝

**浅拷贝：**对基本数据类型进行值传递，对引用数据类型进行引用传递

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200722172251.png" style="zoom: 25%;" />

**深拷贝：**对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200722173559.png" style="zoom:25%;" />



### 面向对象和面向过程的区别

**面向过程：**面向过程性能比面向对象高。因为类调用时需要实例化，开销比较大，比较消耗资源，但是，面向过程没有面向对象易维护、易复用、易扩展

**面向对象：**面向对象易维护、易复用、易扩展。因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，是系统更加灵活、更加易于维护，但是面向对象性能比面向过程低

> Java 性能差的主要原因是 Java是半解释语言



### 面向对象三大特征

#### 封装

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性

#### 继承

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间，提高开发效率

**关于继承的 3 点：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问的，**只是拥有**
2. 子类可以拥有自己的属性和方法，即子类可以对父类进行扩展
3. 子类可以用自己的方式实现父类的方法

#### 多态

多态分为编译时多态和运行时多态：

* 编译时多态主要指方法的重载
* 运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定

运行时多态有三个条件：

* 继承
* 覆盖（重写）
* 向上转型



### 接口和抽象类

#### 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明，如果一个类中包含抽象方法，那么这个类必须声明为抽象类

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承

#### 接口

接口是抽象类的延申，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现

从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected

接口的字段默认都是 static 和 final 的

#### 比较

1. 接口的方法默认是 public，所有方法在接口中不能有实现（Java 8 开始接口方法可以有默认实现（default关键字）），而抽象类可以有非抽象方法
2. 接口中除了 static、final 变量，不能有其他变量，而抽象类中则不一定
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口本身可以通过 extends 关键字扩展多个接口
4. 接口方法默认修饰符是 Public，抽象方法可以有 Public，Protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰）
5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范



### String、StringBuilder 和 StringBuffer区别

#### String 不可变原因

String 类中使用 final 关键字修饰字符数组来保存字符串，`private final char value[]`，所以 String 对象不可变

StringBuilder 和 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串 `char[] value`，但是没有用 final 关键字修饰，所以这两个对象是可变的

#### 线程安全性

String 中的对象是不可变的，可以理解为常量，线程安全

StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的

StringBuilder 没有对方法加同步锁，所以是非线程安全的

#### 性能

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象

StringBuilder 比 StringBuffer 性能高



### Object 类的方法

```java
// 用于返回当前运行对象的 Class 对象
public final native Class<?> getClass();
// 返回对象的哈希码，主要使用在哈希表中
public native int hashCode();
// 用于比较两个对象的内存地址是否相等
public boolean equals(Object obj);
// 用于创建并返回当前对象的拷贝
protected native Object clone() throws CloneNotSupportedException; 
// 返回类的名字
public String toString();
// 唤醒一个在此对象 monitor 上等待的线程
public final native void notify();
// 唤醒在此对象 monitor 上等待的所有线程
public final native void notifyAll();
// 暂停线程的执行
public final native void wait(long timeout) throws InterruptedException;
//  nanos 表示额外时间
public final void wait(long timeout, int nanos) throws InterruptedException;
// 一直等待，没有超时时间的概念
public final void wait() throws InterruptedException;
// 对象被垃圾回收器回收的时候触发的操作
protected void finalize() throws Throwable;
```



### Java 序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用 transient 关键字修饰

transient 关键字的作用是：阻止实例中那些用此关键字修饰的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。transient 只能修饰变量，不能修饰类和方法



### final、static、this、super 关键字总结

#### final

1. final 修饰的类不能被继承，final 类中的所有成员方法都会被隐式的指定为 final 方法
2. final 修饰的方法不能被重写
3. final 修饰的变量是常量，如果是基本数据类型的变量，其数值一旦在初始化后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能让其指向另一个对象

#### static

static 关键字主要有以下四种使用场景：

1. **修饰成员变量和成员方法：**被 static 修饰的成员属于类，不属于这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被 static 声明的成员变量属于静态成员变量，静态变量存放在 Java 内存区域的方法区
2. **静态代码块：**静态代码块定义在类中方法外，静态代码块在非静态代码块（构造代码块）之前执行（静态代码块 -> 非静态代码块 -> 构造方法）。该类不管创建多少对象，静态代码块只执行一次
3. **静态内部类（static 修饰类的话只能修饰内部类）：**静态内部类与非静态内部类的最大区别：非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：
   * 它的创建是不需要依赖外围类的创建
   * 它不能使用任何外围类的非 static 成员变量和方法

4. **静态导包（用来导入类中的静态资源）：**import static，可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法

#### this

this 关键字用于引用类的当前实例

```java
class Manager {
    Employees[] employees;
    
    void manageEmployees() {
    	int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
    
    void report(){}
}
```

上面示例中，this 关键字用在两个地方：

* this.employees.length：访问类 Manager 的当前实例的变量
* this.report()：调用类 Manager 的当前实例的方法

此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。但是，使用此关键字可能会使代码更易读或易懂

#### super

super 关键字用于从子类访问父类的变量和方法

```java
public class Super {
    protected int number;
    
    protected showNumber(){
        System.out.println("number = " + number);
    }
}

public class Sub extends Super{
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

在上面的例子中，Sub 类访问父类成员变量 number 并调用其父类 Super 的 showNumber() 方法

**使用 this 和 super 要注意的问题：**

* 在构造器中使用 super() 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行
* this、super 不能用在 static 方法中

**解释：**

被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以，**this 和 super 是属于对象范畴的东西，而静态方法是属于类范畴的东西**



### 异常

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723155838.png" style="zoom: 25%;" />

**Error（错误）：是程序无法处理的错误**，表示运行应用程序中较严重的问题。错误表示故障发生于虚拟机自身，或者发生在虚拟机试图执行应用时。

**Exception（异常）：是程序本身可以处理的异常**

**异常和错误的区别：**异常能被程序本身处理，错误是无法处理的

#### Throwable 类常用方法

* public String getMessage()：返回异常发生时的简要描述
* public String toString()：返回异常发生时的详细信息
* public String getLocalizedMessage()：返回异常对象的本地化信息
* public void printStackTrace()：在控制台上打印 Throwable 对象封装的异常信息

#### try-catch-finally

* try：用于捕获异常，其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally
* catch：用于处理 try 捕获到的异常
* finally：无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 或 catch 中遇到 return 语句时，finally 语句块将在方法放回前被执行

**以下 4 中特殊情况下，finally 不会被执行：**

1. 在 finally 语句块第一行发生了异常
2. 在前面的代码中用了 System.exit(init) 已退出程序。exit 是带参函数；若该语句在异常语句之后，finally 会执行
3. 程序所在的线程死亡
4. 关闭 CPU

> 当 try 语句和 finally 语句中都有 return 语句时，在方法放回之前，finally 语句的内容将被执行，并且 finally 语句的返回值会覆盖原始的返回值



### Java NIO

#### NIO 和 IO 的区别

| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
| （无） | 选择器     |

NIO 的核心在于：通道（Channel）和缓冲区（Buffer）。通道表示打开到 IO 设备（例如：文件、套接字）的连接。若需要使用 NIO，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理，简而言之，**Channel 负责传输，Buffer 负责存储**

#### 缓冲区（Buffer）

* 缓冲区（Buffer）：一个用于特定基本数据类型的容器。有 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类