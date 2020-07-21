## JVM

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
3. 从内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口

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

##### 解析

解析阶段是虚拟机将常量池内的 **符号引用** 替换为 **直接引用** 的过程。

符号引用就是一组符号来描述目标，可以是任何字面量；直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用

##### 初始化

初始化是类加载的最后一步，也是真正执行类中定义的 Java 程序代码（字节码），初始化阶段是执行类构造器 `<clinit>()` 方法的过程

`<clinit>()`是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。

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

* 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，比如使用 new 关键字实例化对象的时候；读取或赋值类的静态变量；调用类的静态方法
* 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行初始化，则需要先触发初始化
* 当初始化类的时候，发现其父类还没有进行初始化，需要先触发其父类的初始化
* 当虚拟机启动时，用户需要定义一个要执行的主类（包含 main 方法的类），虚拟机会先初始化主类
* MethodHandle 和 VarHandle 可以看作是轻量级的反射调用机制，而要想使用这2个调用， 就必须先使用 findStaticVarHandle 来初始化要调用的类