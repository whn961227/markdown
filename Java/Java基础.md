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
* **静态代码块：**定义在类中，且有 static -> 最先被执行，且对于一个类的多个对象，**只执行一次**
* **同步代码块：**synchronized 关键字修饰的，和线程相关

#### 静态代码块和构造代码块的区别

**相同点：**都是 JVM 加载类后且在构造方法前执行，在类中可定义多个，一般在代码块中对一些 static 变量进行赋值

**不同点：**静态代码块在构造代码块前执行。静态代码块只在第一次 new 时执行一次，之后不再执行。而**构造代码块每 new 一次就执行一次**

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

**构造代码块**是给**所有对象进行统一初始化**，而**构造函数**是**给对应的对象初始化**，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容



### 泛型

即“参数化类型”，就是**将类型由原来具体的类型参数化**，类似于方法中的变量参数，此时**类型也定义成参数形式**（可以称之为**类型形参**），然后**在使用/调用时传入具体的类型**（**类型实参**）

在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别称为泛型类、泛型接口和泛型方法

它提供了**编译期的类型安全**

#### Java 的泛型是如何工作的？什么是类型擦除？

泛型是通过**类型擦除**实现的，编译器在**编译时擦除了所有类型相关的信息**，所以在**运行时不存在任何类型相关的信息**。你无法在运行时访问到类型参数，因为**编译器已经把泛型类型转换成了原始类型**。

#### 什么是泛型中的限定通配符和非限定通配符？

限定通配符对类型进行了限制。有两种限定通配符，一种是 **<? extends T>**，它确保**类型必须是 T 的子类**来设定类型的**上界**，另一种是 **<? super T>**，它通过确保**类型必须是 T 的父类**来设定类型的**下界**。泛型类型**必须用限定内的类型来进行初始化**，否则会导致**编译错误**。另一方面 **< ? >** 表示了**非限定通配符**，因为 < ? > 可以**用任意类型来替代**。 



### 值传递和引用传递

> **值传递：**是指在调用函数时将实参**复制**一份传递到形参中，这样**在函数中如果对参数进行修改，将不会影响到实参**
>
> **引用传递：**是指在调用函数时将实参的地址**直接**传递到形参中，那么**在函数中对参数所进行的修改，将影响到实参**

**总结**

Java 都是值传递，关键看这个值是什么，简单变量就是复制了具体值，引用变量就是复制了地址



### == 和 equals() 的区别

**==：**作用是判断两个对象的地址是不是相等。即判断两个对象是否是同一个对象（**基本数据类型 == 比较的是值**，**引用数据类型 == 比较的是内存地址**）

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

> **总结**

**==** 对于**基本类型**来说是**值比较**，对于**引用类型**来说是比较的是**引用**；而 **equals** 默认情况下是**引用比较**，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了**值比较**，所以一般情况下 equals 比较的是值是否相等



### hashCode() 和 equals()

> **两个对象的 hashCode() 相同，则 equals() 也一定为 true，对吗？**

不对，两个对象的 hashCode()相同，equals()不一定 true

```java
String str1 = "通话";
String str2 = "重地";
System.out.println(String.format("str1: %d | str2: %d", str1.hashCode(), str2.hashCode()));
System.out.println(str1.equals(str2));
/*
str1: 1179395 | str2:1179395
false

代码解读：很显然“通话”和“重地”的 hashCode() 相同，然而 equals() 则为 false，因为在散列表中，hashCode()相等即两个键值对的哈希值相等，然而哈希值相等，并不一定能得出键值对相等。
*/
```



#### 为什么重写 equals 必须重写 hashCode 方法？

如果两个对象相等，equals 方法返回 true，则 hashcode 一定相同，hashCode() 的默认行为是对堆上的对象产生哈希值，如果没有重写 hashCode()，则 class 的两个对象无论如何都不会相等



### 重载和重写的区别

>重载就是**同样的方法**能够**根据输入数据的不同**，**做出不同的处理**
>
>重写就是**当子类继承自父类的相同方法**，**输入数据一样**，但要**实现和父类不同的功能**时，就要覆盖父类方法



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

* **装箱：**根据**基本类型创建它们对应的包装类型**
* **拆箱：**将**包装类型转换为基本数据类型**

> 通过 valueOf 方法创建 Integer 对象时，**如果数值在 [-128, 127] 之间**，**便返回指向 IntegerCache.cache 中已经存在的对象的引用**；**否则创建一个新的 Integer 对象**

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

**浅拷贝：**对**基本数据类型**进行**值传递**，对**引用数据类型**进行**引用传递**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200722172251.png" style="zoom: 25%;" />

**深拷贝：**对**基本数据类型进行值传递**，对**引用数据类型**，**创建一个新的对象**，**并复制其内容**。深拷贝相比于浅拷贝**速度较慢**并且**花销较大**。

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200722173559.png" style="zoom:25%;" />



### 面向对象和面向过程的区别

**面向过程：**面向过程性能比面向对象高。因为类**调用时需要实例化**，开销比较大，比较消耗资源，但是，面向过程没有面向对象易维护、易复用、易扩展

**面向对象：**面向对象**易维护、易复用、易扩展**。因为面向对象有**封装、继承、多态**性的特性，所以可以设计出低耦合的系统，是系统更加灵活、更加易于维护，但是面向对象性能比面向过程低

> Java 性能差的主要原因是 Java是半解释语言



### 面向对象三大特征

#### 封装

封装是指把一个对象的状态信息（也就是**属性**）**隐藏在对象内部**，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性

#### 继承

继承是使用**已存在的类的定义作为基础建立新类**的技术，新类的定义可以**增加新的数据或新的功能**，也可以**用父类的功能**，但**不能选择性地继承父类**。通过使用继承，可以**快速地创建新类**，可以**提高代码的重用**，**程序的可维护性**，节省大量创建新类的时间，提高开发效率

**关于继承的 3 点：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问的，**只是拥有**
2. 子类可以拥有自己的属性和方法，即子类可以**对父类进行扩展**
3. 子类可以用自己的方式实现父类的方法（**重写**）

#### 多态

多态分为**编译时多态**和**运行时多态**：

* 编译时多态主要指**方法的重载**
* 运行时多态指程序中定义的**对象引用所指向的具体类型在运行期间才确定**

运行时多态有三个条件：

* 继承
* 覆盖（重写）
* 向上转型

> **向上转型**

```java

public class Animal {
    public void eat(){
        System.out.println("animal eatting...");
    }
}

public class Cat extends Animal{

    public void eat(){

        System.out.println("我吃鱼");
    }
}

public class Dog extends Animal{

    public void eat(){

        System.out.println("我吃骨头");
    }

    public void run(){
        System.out.println("我会跑");
    }
}

public class Main {

    public static void main(String[] args) {

        Animal animal = new Cat(); //向上转型
        animal.eat();

        animal = new Dog();
        animal.eat();
    }

}

//结果:
//我吃鱼
//我吃骨头
```

当父类引用变量引用子类对象时，**被引用对象的类型决定了调用谁的成员方法**，**引用变量类型决定可调用的方法**。**Animal**是引用变量类型，它决定哪些方法可以调用：`eat()`方法可以调用，而**cat**是被引用对象的类型，它决定了调用谁的方法：调用cat 的方法。

* 子类引用的对象转换为父类类型称为向上转型。
* `Animal animal = new Cat();`将子类对象Cat转化为父类对象Animal。这个时候animal这个引用调用的方法是子类方法。

> **向上转型应注意的问题**

* 向上转型时，子类单独定义的方法会丢失。比如上面Dog类中定义的run方法，当animal引用指向Dog类实例时是访问不到run方法的，`animal.run()`会报错。

* 子类引用不能指向父类对象。`Cat c = (Cat)new Animal()`这样是不行的。

> 1. 将子类对象赋值给父类对象，父类对象就成了子类的上转型对象，但是这**只能访问从父类继承的方法和变量**或者**重写的方法**。
> 2. 只能让上转型对象调用（访问）子类中与父类有关的成员，**子类中自己后定义的成员不能被调用（变量或方法）**

> **向上转型的好处**

* **减少重复代码，使代码变得简洁。**
* **提高系统扩展性。**

> **静态方法的调用**

```java
class A {
	public static void show(){
        System.out.println("hhhh");
    }
}

class B extends A {
    public static void show(){
        System.out.println("你哈");
    }
}
public class C{

    public static void main(String[] args) {
        A b=new B();
        b.show();
    }
}
/*
静态方法不具有多态性，因为静态和类相关，和对象实例无关，静态方法可以继承和重写，但重写只是形式上的（算不上重写），父类方法并没有被覆盖掉。
*/
```



### 接口和抽象类

#### 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明，如果一个类中包含抽象方法，那么这个类必须声明为抽象类

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承

> **抽象类必须要有抽象方法吗**

不需要，抽象类不一定非要有抽象方法。

```java
abstract class Cat {
    public static void sayHi() {
        System.out.println("Hi~");
    }
}
```

上面代码，抽象类并没有抽象方法但完全可以正常运行。

> **普通类和抽象类有哪些区别？**

* 普通类不能包含抽象方法，抽象类可以包含抽象方法。
* **抽象类不能直接实例化**，**普通类可以直接实例化**。

> **抽象类能使用 final 修饰吗？**

不能，定义抽象类就是让其他类继承的，如果定义为 final 该类就不能被继承，这样彼此就会产生矛盾，所以 final 不能修饰抽象类

#### 接口

接口是抽象类的延申，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现

从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected

接口的字段默认都是 static 和 final 的

#### 比较

1. **接口的方法默认是 public，所有方法在接口中不能有实现**（**Java 8 开始接口方法可以有默认实现（default关键字）**），而**抽象类可以有非抽象方法**
2. 接口中除了 static、final 变量，不能有其他变量，而抽象类中则不一定
3. **一个类可以实现多个接口**，但**只能实现一个抽象类**。**接口本身可以通过 extends 关键字扩展多个接口**
4. **接口中的方法默认修饰符是 Public**，抽象类中的方法可以有 Public，Protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰）
5. 实现：抽象类的子类使用 **extends** 来继承；接口必须使用 **implements** 来实现接口
6. 实现数量：**类可以实现很多个接口**；但是**只能继承一个抽象类**
7. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范



### String、StringBuilder 和 StringBuffer区别

#### String 不可变原因

String 声明的是**不可变的对象**，每次操作都会生成新的 String 对象，然后将指针指向新的 String 对象，String 类中使用 **final 关键字修饰字符数组**来保存字符串，`private final char value[]`，所以 String 对象不可变

StringBuilder 和 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是**使用字符数组保存字符串** `char[] value`，但是没有用 final 关键字修饰，所以这两个对象是可变的

>  **String str="i"与 String str=new String("i")一样吗？**

不一样，因为内存的分配方式不一样。String str="i"的方式，java 虚拟机会将其分配到**常量池**中；而 String str=new String("i") 则会被分到**堆内存**中



#### 线程安全性

String 中的对象是不可变的，可以理解为常量，线程安全

StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是**线程安全**的

StringBuilder 没有对方法加同步锁，所以是**非线程安全**的

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
// nanos 表示额外时间
public final void wait(long timeout, int nanos) throws InterruptedException;
// 一直等待，没有超时时间的概念
public final void wait() throws InterruptedException;
// 对象被垃圾回收器回收的时候触发的操作
protected void finalize() throws Throwable;
```



### Java 序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用 **transient** 关键字修饰

transient 关键字的作用是：阻止实例中那些用此关键字修饰的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。**transient 只能修饰变量**，**不能修饰类和方法**



### final、static、this、super 关键字总结

#### final

1. final 修饰的类叫**最终类**，**该类不能被继承**，final 类中的所有成员方法都会被隐式的指定为 final 方法
2. **final 修饰的方法不能被重写**
3. final 修饰的变量是**常量**，常量**必须初始化**，如果是**基本数据类型**的变量，其数值一旦在初始化后便**不能更改**；如果是**引用类型**的变量，则在对其初始化之后便**不能让其指向另一个对象**

#### static

static 关键字主要有以下四种使用场景：

1. **修饰成员变量和成员方法：**被 static 修饰的成员属于类，不属于这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被 static 声明的成员变量属于静态成员变量，静态变量存放在 Java 内存区域的方法区
2. **静态代码块：**静态代码块定义在类中方法外，静态代码块在非静态代码块（构造代码块）之前执行（静态代码块 -> 非静态代码块 -> 构造方法）。该类不管创建多少对象，静态代码块只执行一次
3. **静态内部类（static 修饰类的话只能修饰内部类）：**静态内部类与非静态内部类的最大区别：非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：
   * 它的创建是不需要依赖外围类的创建
   * 它不能使用任何外围类的非 static 成员变量和方法

4. **静态导包（用来导入类中的静态资源）：**import static，可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法

#### this

this 关键字用于**引用类的当前实例**

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

super 关键字用于**从子类访问父类的变量和方法**

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

* **在构造器中使用 super() 调用父类中的其他构造方法时**，**该语句必须处于构造器的首行**，否则编译器会报错。另外，**this 调用本类中的其他构造方法时**，**也要放在首行**
* **this、super 不能用在 static 方法中**

**解释：**

**被 static 修饰的成员属于类**，不属于单个这个类的某个对象，被类中所有对象共享。而 **this 代表对本类对象的引用**，**指向本类对象**；而 **super 代表对父类对象的引用，指向父类对象**；所以，**this 和 super 是属于对象范畴的东西，而静态方法是属于类范畴的东西**



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



### Java BIO

**BIO**：传统的网络通讯模型，就是 BIO，同步阻塞 IO

它其实就是**服务端创建一个 ServerSocket**， 然后就是**客户端用一个 Socket 去连接服务端的那个 ServerSocket**， **ServerSocket 接收到了一个的连接请求就创建一个 Socket 和一个线程去跟那个 Socket 进行通讯**。

接着**客户端和服务端就进行阻塞式的通信**，**客户端发送一个请求**，**服务端Socket进行处理后返回响应**。

**在响应返回前**，**客户端**那边就**阻塞等待**，上门事情也做不了。

这种方式的缺点：**每次一个客户端接入，都需要在服务端创建一个线程来服务这个客户端**

这样**大量客户端**来的时候，就会造成**服务端的线程数量可能达到了几千甚至几万**，这样就可能会**造成服务端过载过高**，最后**崩溃**死掉。

BIO 模型图：

<img src="https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVNP0MCbH5iaZ1kSG4njLGG3hXkY63N1ibT7SUSBmlkFo1T4M35hlbrLSw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 67%;" />

**Acceptor：**

传统的 IO 模型的网络服务的设计模式中有俩种比较经典的设计模式：一个是**多线程**， 一种是**依靠线程池来进行处理**

如果是基于多线程的模式来的话，就是这样的模式，这种也是 Acceptor 线程模型。

<img src="https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVGgPqeeqfPUc3WMHdwP2icdqeyP1fLMNAQMH08rUg2KTLF2bsWI4xKlg/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom:80%;" />

### Java NIO

NIO 是一种同步非阻塞 IO, 基于 Reactor 模型来实现的。

其实相当于就是**一个线程处理大量的客户端的请求**，通过一个线程轮询大量的 channel，每次就获取一批有事件的channel，然后对每个请求启动一个线程处理即可。

这里的核心就是非阻塞，就那个 **selector 一个线程就可以不停轮询 channel**，**所有客户端请求都不会阻塞**，直接就会进来，大不了就是等待一下排着队而已。

这里面**优化 BIO 的核心**就是，一个客户端并不是时时刻刻都有数据进行交互，没有必要死耗着一个线程不放，所以客户端选择了让线程歇一歇，**只有客户端有相应的操作的时候才发起通知**，**创建一个线程来处理请求**。

> **为什么说 NIO 为啥是同步非阻塞？**

因为无论多少客户端都可以接入服务端，客户端接入并不会耗费一个线程，只会**创建一个连接然后注册到 selector** 上去，这样你就可以去干其他你想干的其他事情了

**一个 selector 线程不断的轮询所有的 socket 连接**，发现有事件了就通知你，然后你就**启动一个线程处理一个请求**即可，这个过程的话就是**非阻塞**的。

但是这个处理的过程中，你还是要先**读取数据，处理，再返回**的，这是个**同步**的过程。

**NIO：模型图**

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMV5gDRaeJenSQ50g97N68RGzytWM1Ul9QibFT8381UG9evPiaInqIKe5VQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Reactor 模型:**

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVHaea33Nt2wjs36XKvG6P7Lk7GvVdE7lp1kgYAWNRAFp4aia8gaMwzwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### NIO 核心组件详解

**多路复用机制实现 Selector**

首先我们来了解下传统的 Socket 网络通讯模型。

**传统Socket通讯原理图**

<img src="https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVPBWhch0tXTWl73IDynfMic6dtWb5Ujcrpx8alFTaQiaptt1M94zMrU3A/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" alt="img" style="zoom: 80%;" />

为什么传统的socket不支持海量连接？

**每次一个客户端接入，都是要在服务端创建一个线程来服务这个客户端的**

这会导致大量的客户端的时候，服务端的线程数量可能达到几千甚至几万，几十万，这会导致服务器端程序负载过高，不堪重负，最终系统崩溃死掉。

接着来看下 NIO 是如何**基于 Selector 实现多路复用机制支持的海量连接**。

**NIO原理图**

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVMHY3tc8kfLlMnVBFibM3ibxKULbQEuP2ISYmEIwycOByOWibwMPedhUdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

多路复用机制是如何支持海量连接？

NIO 的线程模型对 Socket 发起的连接不需要每个都创建一个线程，完全可以**使用一个 Selector 来多路复用监听 N 多个 Channel 是否有请求**，该请求是对应的连接请求，还是发送数据的请求

这里面是基于操作系统底层的 Select 通知机制的，**一个 Selector 不断的轮询多个 Channel，这样避免了创建多个线程**

只有当每个 Channel 有对应的请求的时候才会创建线程，可能说1000个请求， 只有100个请求是有数据交互的

这个时候可能 server 端就提供 10 个线程就能够处理这些请求。这样的话就可以避免了创建大量的线程。

**NIO 如何通过 Buffer 来缓冲数据的**

NIO 中的 Buffer 是个什么东西 ？

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVWFDOUicLicQJuKlicrARGN15PXpTJuf1SIic09EwwcTm3IZy5Gvy4Xkzrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

学习 NIO，首当其冲就是要了解所谓的 Buffer 缓冲区，这个东西是 NIO 里比较核心的一个部分

一般来说，如果你要通过 **NIO 写数据到文件或者网络**，或者是**从文件和网络读取数据**出来此时就需要**通过 Buffer 缓冲区**来进行。Buffer 的使用一般有如下几个步骤：

写入数据到 Buffer，调用 flip() 方法，从 Buffer 中读取数据，调用 clear() 方法或者 compact() 方法。

Buffer 中对应的 Position， Mark， Capacity，Limit 都啥？

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVC5duJ3GMTURh7JLLxx9fsHpgyylUe0UBw83VZ5WoglKxElkS6vJTlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* **capacity**：缓冲区容量的大小，就是里面包含的数据大小。
* **limit**：对 buffer 缓冲区使用的一个限制，从这个 index 开始就不能读取数据了。
* **position**：代表着数组中可以开始读写的 index， 不能大于 limit。
* **mark**：标记，表示记录当前 position 的位置，可以通过 reset() 恢复到 mark 位置

**如何通过 Channel 和 FileChannel 读取 Buffer 数据写入磁盘的**

NIO中，Channel是什么？ 

Channel是NIO中的数据通道，类似流，但是又有些不同

**Channel既可从中读取数据**，**又可以从写数据到通道中**，但是**流的读写通常是单向的**。

Channel可以**异步的读写**。**Channel中的数据总是要先读到一个Buffer中**，或者**从缓冲区中将数据写到通道中**。

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVUdpia43f2s5bVTVEshQGPfIZZguNFLwyNvD0Mhz67XYIvZCNt6MqVKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



#### NIO 和 IO 的区别

| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
| （无） | 选择器     |

NIO 的核心在于：**通道（Channel）**和**缓冲区（Buffer）**。通道表示打开到 IO 设备（例如：文件、套接字）的连接。若需要使用 NIO，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理，简而言之，**Channel 负责传输，Buffer 负责存储**

#### 缓冲区（Buffer）

* 缓冲区（Buffer）：一个用于特定基本数据类型的容器。有 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类

* Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的

```java
/**
 * 一. 缓冲区（Buffer）：在 Java NIO 中负责数据的存取。缓冲区就是数组，用于存储不同数据类型的数据
 * 
 * 根据数据类型的不同（boolean 除外），提供了相应类型的缓冲区：
 * ByteBuffer CharBuffer ShortBuffer IntBuffer LongBuffer FloatBuffer DoubleBuffer
 * 通过 allocate() 获取缓冲区
 *
 * 二. 缓冲区存取数据的两个核心方法：
 * put()：存入数据到缓冲区
 * get()：获取缓冲区中的数据
 * 
 * 三. 缓冲区中的四个核心属性
 * capacity：容量，表示缓冲区中最大存储数据的容量。一旦声明不能改变
 * limit：界限，表示缓冲区中可以操作数据的大小。（limit 后数据不能进行读写）
 * position：位置，表示缓冲区中正在操作数据的位置
 *
 * mark：标记，表示记录当前 position 的位置，可以通过 reset() 恢复到 mark 位置
 * 
 * 0 <= mark <= position <= limit <= capacity
 * 
 * 四. 直接缓冲区和非直接缓冲区
 * 非直接缓冲区：通过 allocate() 方法分配缓冲区，将缓冲区建立在 JVM 的内存中
 * 直接缓冲区：通过 allocateDirect() 方法分配直接缓冲区，将缓冲区建立在物理内存中，可以提高效率
 */
public class TestBuffer {
    public void test() {
        String str = "abcde";
        
        // 1.分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        
        // 2.利用 put() 存入数据到缓冲区中
        buf.put(str.getBytes());
        
        // 3. 切换读取数据模式
        buf.flip();
        
        // 4.利用 get() 读取缓冲区中的数据
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst, 0, dst.length));
        
        // 5.rewind()：可重复读数据
        buf.rewind();
        
        // 6.clear()：清空缓冲区，但是缓冲区的数据依然存在，但是处于“被遗忘”状态
        buf.clear();
    }
}
```

##### 非直接缓冲区

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723214016.png" style="zoom: 67%;" />

##### 直接缓冲区

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723214342.png" style="zoom:67%;" />

效率高，但是分配和销毁需要消耗较大资源；不易控制

建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区

#### 通道（Channel）

* 通道（Channel）：由 java.nio.channels 包定义的。Channel 表示 IO 源与目标打开的连接。Channel 类似于传统的“流”，只不过 **Channel 本身不能直接访问数据**，**Channel 只能与 Buffer 进行交互**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723215911.png" style="zoom:67%;" />

```java
/**
 * 一. 通道（Channel）：用于源节点与目标节点的连接。在 Java NIO 中负责缓冲区中数据的传输。Channel 本身不存储数据，因此需要配合缓冲区进行传输
 * 
 * 二. 通道的主要实现类
 * java.nio.channels.Channel 接口：
 * 		|--FileChannel
 * 		|--SocketChannel
 * 		|--ServerSocketChannel
 * 		|--DatagramChannel
 *
 * 三. 获取通道
 * 1. Java 针对支持通道的类提供了 getChannel() 方法
 *		本地 IO：
 *		FileInputStream / FileOutputStream
 *		RandomAccessFile
 *
 *		网络 IO：
 * 		Socket
 *		ServerSocket
 *		DatagramSocket
 * 2. 在 JDK 1.7 中的 NIO.2 针对各个通道提供了静态方法 open()
 * 3. 在 JDK 1.7 中的 NIO.2 的 Files 工具类的 newByteChannel() 
 *
 * 四. 通道之间的数据传输
 * transferFrom()
 * transferTo()
 *
 * 五. 分散（Scatter）与聚集（Gather）
 * 分散读取（Scattering Reads）：将通道中的数据分散到多个缓冲区中
 * 聚集写入（Gathering Writes）：将多个缓冲区中的数据聚集到通道中
 *
 * 六. 字符集：Charset
 * 编码：字符串 -> 字节数组
 * 解码：字节组数 -> 字符串
 */
public class TestChannel {
    // 1.利用通道完成文件的复制（非直接缓冲区）
    public void test() {
        FileInputStream fis = null;
        FileOutputStream fos = null
        // 获取通道
        FileChannel inChannel = null;
        FileChannel outChannel = null;
        try {
            fis = new FileInputStream("1.jpg");
            fos = new FileOutputStream("2.jpg");
            inChannel = fis.getChannel();
            outChannel = fos.getChannel();
            // 2) 分配指定大小的缓冲区
            ByteBuffer buf = ByteBuffer.allocate(1024);
            // 3) 将通道中的数据存入缓冲区
            while(inChannel.read(buf) != -1) {
                buf.flip(); // 切换读取数据模式
                // 4) 将缓冲区中的数据写入通道中
                outChannel.write(buf);
                buf.clear(); // 清空缓冲区
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (outChannel != null) {
                try {
                    outChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (inChannel != null) {
                try {
                    inChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    // 2. 使用直接缓冲区完成文件的复制（内存映射文件）
    public void test() throws IOException {
        FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);
        // 内存映射文件
        MappedByteBuffer inMappedBuf = inChannel.map(MapMode.READ_ONLY, 0, inChannel.size());
        MappedByteBuffer outMappedBuf = outChannel.map(MapMode.READ_WRITE, 0, inChannel.size());
        // 直接对缓冲区进行数据的读写操作
        byte[] dst = new byte[inMappedBuf.limit()];
        inMappedBuf.get(dst);
        outMappedBuf.put(dst);
        // 关闭通道
        inChannel.close();
        outChannel.close();
    }
    
    // 通道之间的数据传输（直接缓冲区）
     public void test() throws IOException {
         FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);
         
         inChannel.transferTo(0, inChannel.size(), outChannel);
         // outChannel.transferFrom(inChannel, 0, inChannel.size());
         
         inChannel.close();
         outChannel.close();
     }
    
    // 分散和聚集
    public void test() throws IOException {
        RandomAccessFile raf1 = new RandomAccessFile("1.txt", "rw");
        
        // 1. 获取通道
        FileChannel channel1 = raf1.getChannel();
        
        // 2. 分配指定大小的缓冲区
        ByteBuffer buf1 = ByteBuffer.allocate(100);
        ByteBuffer buf2 = ByteBuffer.allocate(1024);
        
        // 3. 分散读取
        ByteBuffer[] bufs = {buf1, buf2};
        channel1.read(bufs);
        
        // 4. 聚集写入
        RandomAccessFile raf2 = new RandomAccessFile("2.txt", "rw");
        FileChannel channel2 = raf2.getChannel();
        
        channel2.write(bufs);
    }
    
    // 字符集
    public void test() {
        Charset cs1 = Charset.forName("GBK");
        
        // 获取编码器
        CharsetEncoder ce = cs1.newEncoder();
        // 获取解码器
        CharsetDecoder cd = cs1.newDecoder();
        
        CharBuffer cBuf = CharBuffer.allocate(1024);
        cBuf.put("威武！");
        cBuf.flip();
        
        // 编码
        ByteBuffer bBuf = ce.encode(cBuf);
        
        // 解码
        bBuf.flip();
        CharBuffer cBuf2 = cd.decode(bBuf);
    }
}
```

#### 分散（Scatter）和聚集（Gather）

* 分散读取（Scattering Reads）是指从 Channel 中读取的数据“分散”到多个 Buffer 中

> 注意：按照缓冲区的顺序，从 Channel 中读取的数据依次将 Buffer 填满

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723225113.png" style="zoom: 67%;" />

* 聚集写入（Gathering Writes）是指将多个 Buffer 中的数据“聚集”到 Channel

> 注意：按照缓冲区的顺序，写入 position 和 limit 之间的数据到 Channel

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200723225334.png" style="zoom:67%;" />

#### NIO 网络通信

##### 阻塞式

```java
/**
 * 一. 使用 NIO 完成网络通信的三个核心：
 * 1. 通道（Channel）: 负责连接
 * 			java.nio.channels.Channel 接口：
 * 				|--SelectableChannel
 *					|--SocketChannel
 *					|--ServerSocketChannel
 *					|--DatagramChannel
 *					
 *					|--Pipe.SinkChannel
 *					|--Pipe.SourceChannel
 * 2. 缓冲区（Buffer）：负责数据的存取
 * 3. 选择器（Selector）：是 SelectableChannel 的多路复用器，用于监控 SelectableChannel 的 IO 状况
 * 
 */
public class TestBlockingNIO {
    // 客户端
    public void client() throws IOException {
        // 1. 获取通道
        SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));

        FileChannel inChannel = FileChannel.open(Paths.get("1.jpg"), standardOpenOption.READ);

        // 2. 分配指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        // 3. 读取本地文件，并发送到服务器
        while (inChannel.read(buf) != -1) {
            buf.flip();
            sChannel.write(buf);
            buf.clear();
        }

        // 4. 关闭通道
        inChannel.close();
        sChannel.close();
    }
    
    // 服务端
    public void server() throws IOException{
        // 1. 获取通道
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        
        FileChannel outChannel = FileChannel.open(Paths.get("2.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)
        
        // 2. 绑定连接
        ssChannel.bind(new InetSocketAddress(9898));
        
        // 3. 获取客户端连接的通道
        SocketChannel sChannel = ssChannel.accept();
        
        // 4. 分配指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        
        // 5. 接收客户端的数据，并保存到本地
        while (sChannel.read(buf) != -1) {
            buf.flip();
            outChannel.write(buf);
            buf.clear();
        }
        
        // 6. 关闭通道
        sChannel.close();
        outChannel.close();
        ssChannel.close();
    }
}
```

##### 非阻塞式

````java
public class TestNonBlockingNIO {
    // 客户端
    public void client() throws IOException{
        // 1. 获取通道
        SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
        
        // 2. 切换成非阻塞模式
        sChannel.configureBlocking(false);
        
        // 3. 分配指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        
        // 4. 发送数据给服务端
        buf.put(new Date().toString().getBytes());
        buf.flip();
        sChannel.write(buf);
        buf.clear();
        
        // 5. 关闭通道
        sChannel.close();
    }
    
    // 服务端
    public void server() throws IOException{
        // 1. 获取通道
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        
        // 2. 切换非阻塞模式
        ssChannel.configureBlocking(false);
        
        // 3. 绑定连接
        ssChannel.bind(new InetSocketAddress(9898));
        
        // 4. 获取选择器
        Selector selector = Selector.open();
        
        // 5. 将通道注册到选择器上，并且指定“监听接收事件”
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
        
        // 6. 轮询式的获取选择器上已经“准备就绪”的事件
        while (selector.select() > 0) {
            // 7. 获取当前选择器中所有注册的“选择键（已就绪的监听事件）”
            Iterator<Selectionkey> it = selector.selectedKeys().iterator();
            
            while (it.hasNext()) {
                // 8. 获取准备“就绪”的事件
                SelectionKey sk = it.next();
                // 9. 判断具体是什么事件准备就绪
                if (sk.isAcceptable()) {
                    // 10. 如果“接收就绪”，获取客户端连接
                    SocketChannel sChannel = ssChannel.accept();
                    // 11. 切换非阻塞模式
                    sChannel.configureBlocking(false);
                    
                    // 12. 将该通道注册到选择器上
                    sChannel.register(selector, SelectionKey.OP_READ);
                } else if (sk.isReadable()) {
                    // 13. 获取当前选择器上“读就绪”状态的通道
                    SocketChannel sChannel = (SocketChannel)sk.channel();
                    // 14. 读取数据
                    ByteBuffer buf = ByteBuffer.allocate(1024);
                    
                    int len = 0
                    while ((len = sChannel.read(buf)) > 0){
                        buf.flip();
                        System.out.println(new String(buf.array(), 0, len));
                        buf.clear();
                    }
                }
                // 15. 取消选择键 SelectionKey
                it.remove();
            }
        }
    }
}
````

###### SelectionKey

* SelectionKey：表示 SelectableChannel 和 Selector 之间的注册关系。每次向选择器注册通道时就会选择一个事件（选择键）。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作

* 可以监听的事件类型（可使用 SelectionKey 的四个常量表示）：

  * 读：SelectionKey.OP_READ
  * 写：SelectionKey.OP_WRITE
  * 连接：SelectionKey.OP_CONNECT
  * 接收：SelectionKey.OP_ACCEPT

* 若注册时不止监听一个事件，则可以使用“位或”操作符连接

  例：

  ```java
  int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
  ```

###### UDP

```java
public class TestNonBlockingNIO {
    public void send() throws IOException {
        DatagramChannel dc = DatagramChannel.open();
        
        dc.configureBlocking(false);
        
        ByteBuffer buf = ByteBuffer.allocate(1024);
        
        Scanner scan = new Scanner(System.in);
        
        while (scan.hasNext()) {
            String str = scan.next();
            buf.put(str.getBytes());
            buf.flip();
            dc.send(buf, new InetSocketAddress("127.0.0.1", 9898));
            buf.clear();
        }
        
        dc.close();
    }
    
    public void receive() throws IOException {
        DatagramChannel dc = DatagramChannel.open();
        
        dc.configureBlocking(false);
        
        dc.bind(new InetSocketAddress(9898));
        
        Selector selector = Selector.open();
        
        dc.register(selector, SelectionKey.OP_READ);
        
        while (selector.select() > 0) {
            Iterator<Selectionkey> it = selector.selectedKeys().iterator();
            
            while (it.hasNext()) {
                SelectionKey sk = it.next();
                
                if (sk.isReadable()) {
                    ByteBuffer buf = ByteBuffer.allocate(1024);
                    
                    dc.receive(buf);
                    buf.flip();
                    System.out.println(new String(buf.array(), 0, buf.limit()));
                    buf.clear();
                }
            }
            
            it.remove();
        }
    }
}
```

#### 管道（Pipe）

* Java NIO 管道是 2 个线程之间的单向数据连接。Pipe 有一个 source 通道和一个 sink 通道。数据会被写到 sink 通道，从 source 通道获取

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200724145742.png" style="zoom:33%;" />

```java
public class TestPipe {
    public void test() throws IOException {
        // 1. 获取通道
        Pipe pipe = Pipe.open();
        
        // 2. 将缓冲区中的数据写入管道
        ByteBuffer buf = ByteBuffer.allocate(1024);
        
        Pipe.SinkChannel sinkChannel = pipe.sink();
        buf.put("通过单向管道发送数据".getBytes());
        buf.flip();
        sinkChannel.write(buf);
        
        // 3. 读缓冲区中的数据
        Pipe.SourceChannel sourceChannel = pipe.source();
        buf.flip();
        int len = sourceChannel.read(buf);
        System.out.println(new String(buf.array(), 0, len));
        
        sourceChannel.close();
        sinkChannel.close();
    }
}
```



### Java AIO

AIO：异步非阻塞 IO，基于 Proactor 模型实现。

每个连接发送过来的请求，都会**绑定一个 Buffer**，然后**通知操作系统去完成异步的读**，这个时间你就可以去做其他的事情

等到**操作系统完成读之后**，就会**调用你的接口**，**给你操作系统异步读完的数据**。这个时候你就可以拿到数据进行处理，将数据往回写

在往回写的过程，同样是**给操作系统一个Buffer**，**让操作系统去完成写**，**写完了来通知你**。

这俩个过程都有 buffer 存在，**数据都是通过 buffer 来完成读写**。

这里面的主要的区别**在于将数据写入的缓冲区后，就不去管它，剩下的去交给操作系统去完成**。

**操作系统写回数据也是一样，写到 Buffer 里面，写完后通知客户端来进行读取数据**。

> **为什么说 AIO 是异步非阻塞？**

通过 AIO 起个文件 IO 操作之后，你立马就返回可以干别的事儿了，接下来你也不用管了，操作系统自己干完了 IO 之后，告诉你说 ok 了

当你基于 AIO 的 api 去读写文件时， 当你发起一个请求之后，剩下的事情就是交给了操作系统

当读写完成后， 操作系统会来回调你的接口， 告诉你操作完成

在这期间不需要等待， 也不需要去轮询判断操作系统完成的状态，你可以去干其他的事情。

同步就是自己还得主动去轮询操作系统，异步就是操作系统反过来通知你。所以来说， AIO 就是异步非阻塞的。

**AIO：模型图**

![img](https://mmbiz.qpic.cn/mmbiz_png/1J6IbIcPCLaRj51teUPPUDVohsACoBMVAW3dXN5DbvKianRuLTiccBZBoBTyYS5nqMOVbV1eGQ0Q3tQoQuUFZoOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





### 反射

Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种**动态获取的信息**以及**动态调用对象的方法**的功能称为 Java 语言的反射机制

**好处**

* 可以动态创建对象和编译，体现出很大的灵活性

**坏处**

* 对性能有影响。使用反射基本上是一种解释操作，我们可以告诉 JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于直接执行相同的操作

#### 获取 Class 对象的方式

如果我们动态获取到这些信息，我们需要依靠 Class 对象。Class 类对象将一个类的方法、变量等信息告诉运行的程序。Java 提供了几种方式获取 Class 对象：

1. 知道**具体类的情况**下可以使用：

   ```java
   Class alunbarClass = TargetOjbect.class;
   ```

但是我们一般是不知道具体类的，基本都是通过遍历包下面的类来获取 Class 对象

2. 通过 **`Class.forName()` 传入类的路径**获取：

   ```java
   Class alunbarClass = Class.forName("cn.javaguide.TargetObject");
   ```

3. 已知某个类的实例，调用该**实例的 getClass() 方法获取 Class 对象**

   ```java
   Class alunbarClass = targetObject.getClass();
   ```

4. 基本内置类型的包装类都有一个 Type 属性

##### 代码示例

```java
public class TargetObject {
    private String value;

    public TargetObject() {
        this.value = "test";
    }

    public void publicMethod(String s) {
        System.out.println("I love " + s);
    }

    private void privateMethod() {
        System.out.println("value " + value);
    }
}

public class Mytest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        Class<?> targetClass = Class.forName("TargetObject");
        TargetObject targetObject = (TargetObject) targetClass.newInstance();

        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method);
        }

        Method publicMethod = targetClass.getDeclaredMethod("publicMethod", String.class);
        publicMethod.invoke(targetObject, "trytry");

        Field field = targetClass.getDeclaredField("value");
        // 不能直接操作私有属性，需要关闭程序的安全检查
        field.setAccessible(true);
        field.set(targetObject, "javaguide");

        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}
```



