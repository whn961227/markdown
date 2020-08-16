##  并发编程

### 并行与并发

单核 CPU 下，线程实际是 **串行执行** 的。任务调度器将 CPU 的时间片分给不同的线程使用

一般会将这种线程轮流使用 CPU 的做法称为 **并发**，concurrent

多核 CPU 下，每个核都可以调度运行线程，这时候线程是 **并行** 的

#### 结论

1. 单核CPU下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用CPU，不至于一个线程总占用CPU，别的线程无法运行
2. 多核CPU可以并行跑多个线程，但能否提高程序运行效率还是要分情况的
   * 有的任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分
   * 也不是所有任务都需要拆分，任务的目的不同，拆分和效率就没什么意义
3. IO 操作不占用 CPU，只是一般拷贝文件使用的是 **阻塞 IO**， 这时相当于线程虽然不用 CPU，但需要一直等待 IO 结束，没能充分利用线程。所以才有 **非阻塞 IO** 和 **异步 IO** 的优化



### 同步与异步

从方法调用的角度：

* 需要等待结果返回才能继续运行就是 **同步**
* 不需要等待结果返回就能继续运行就是 **异步**



### 创建和运行线程

* **直接使用 Thread**

  ```java
  // 创建线程对象
  Thread t = new Thread() {
      public void run() {
          // 要执行的任务
      }
  };
  // 启动线程
  t.start();
  ```

* **使用 Runnable 配合 Thread**

  ```java
  Runnable runnable = new Runnable() {
      public void run(){
          // 要执行的任务
      }
  };
  // 创建线程对象
  Thread t = new Thread(runnable);
  // 启动线程
  t.start();
  ```

* **FutureTask 配合 Thread**

  ```java
  // 创建任务对象
  FutureTask<Integer> task3 = new FutureTask<>(() -> {
  	log.debug("hello");
      return 100;
  });
  
  // 参数 1 是任务对象， 参数 2 是线程名字 
  new Thread(task3, "t3").start();
  
  // 主线程阻塞，同步等待 task 执行完毕的结果
  Integer result = task3.get();
  log.debug("结果是：{}", result);
  ```



### 原理之线程运行

#### 栈与栈帧

每个线程启动后，虚拟机就会为其分配一块栈内存

* 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占的内存
* 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法

#### 线程上下文切换

以下原因导致CPU不再执行当前线程，转而执行另外一个线程：

* 线程的 CPU **时间片用完**
* **垃圾回收**
* 有**更高优先级的线程**需要运行
* 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法

当上下文切换发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java中对应的概念就是**程序计数器**，它的作用是记住下一条jvm指令的执行地址，是线程私有的

* 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
* 频繁的上下文切换会影响性能



### Start 与 Run

```java
public static void main(String[] args) {
    // 创建线程对象
    Thread t1 = new Thread("t1") {
        public void run() {
            // 要执行的任务
        }
    };

    t1.run();
}
```

程序仍然在 **main线程** 运行



### sleep 与 yield

#### sleep

1. 调用 sleep 会让当前线程从 Running 进入 **Timed Waiting** 状态
2. 其他线程可以使用 interrupt 打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
3. 睡眠结束后的线程未必会立刻得到执行
4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性

#### yield

1. 调用 yield 会让当前线程从 Running 进入 **Runnable** 就绪状态，然后调度执行其他线程
2. 具体的实现依赖于操作系统的任务调度器



### 线程优先级

* 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
* 如果 CPU 比较忙，那么优先级高的线程会获得更多的时间片，但 CPU 闲时，优先级几乎没作用



 ### join 方法详解

**等待线程运行结束，用于线程同步**

```java
static int r = 0;
public static void main(String[] args) throws InterruptedException {
    test1();
}
private static void test1() throws InterruptedException {
    log.debug("开始");
    Thread t1 = new Thread(() -> {
    	log.debug("开始");
        sleep(1);
        log.debug("结束");
        r = 10;
    });
    t1.start();
    // t1.join();
    log.debug("结果为:{}", r);
    log.debug("结束");
}
```

分析

* 因为主线程和 t1 线程是并行执行的，t1 线程需要 1s 之后才能赋值 r = 10
* 主线程一开始就打印 r 的值，所以只能打印出 r = 0

解决方法

* 用 sleep（不知道 t1 线程执行时间，不推荐）
* 用 join，加在 t1.start() 之后



### Inerrupt 方法详解

#### 打断 sleep，wait，join 的线程

**清空打断状态**，以异常的方式表示被打断，打断标记为 false

#### 打断正常运行的线程

**不会清空打断状态**，打断标记为 true，由被打断线程自己决定是否停止运行，可以通过打断标记来判断

#### 打断 park 线程

不会清空打断状态，打断标记为 true，如果打断标记为 true，再次调用 park 方法将失效





### isInterrupted 和 interrupted

isInterrupted 判断打断标记，**不会清空打断状态**；interrupted 判断打断标记，**清空打断状态**



### 模式之两阶段终止

```java
class TwoPhaseTermination {
    private Thread monitor;

    // 启动监控线程
    public void start() {
        monitor = new Thread(() -> {
            while (true) {
                Thread thread = Thread.currentThread();
                if (thread.isInterrupted()) {
                    System.out.println("料理后事...");
                    break;
                }
                try {
                    Thread.sleep(1000); // 打断情况 1
                    System.out.println("执行监控记录"); // 打断情况 2
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    // 重新设置打断标记
                    thread.interrupt();
                }
            }
        });
        monitor.start();
    }

    // 停止监控线程
    public void stop() {
        monitor.interrupt();
    }
}
```



### 主线程与守护线程

默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其他非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束

> **垃圾回收线程就是守护线程**
>
> Tomcat中的Acceptor和Poller线程都是守护线程，所以Tomcat接收到shutdown命令后，不会等待他们处理完当前请求



### 五种状态

* **初始状态**：仅在语言层面创建了线程对象，还未与操作系统线程关联
* **可运行状态**：线程已经被创建（与操作系统线程关联），可以由CPU调度执行
* **运行状态**：获取了CPU时间片，运行时的状态
  * 当CPU时间片用完，会从 **运行状态** 转换至 **可运行状态** ，导致线程上下文切换

* **阻塞状态**： 
  * 如果调用了阻塞API，如BIO读写文件，这时线程实际不会用到CPU，会导致线程上下文切换，进入 **阻塞状态**
  * 等BIO操作完毕，会由操作系统唤醒阻塞的线程，转换至 **可运行状态**
  * 与 **可运行状态** 的区别是，对 **阻塞状态** 的线程来说只要它们一直不唤醒，调度器就不会考虑调度它们

* **终止状态**：表示线程已经执行完毕，生命周期已经结束，不会再转换为其他状态



### 共享带来的问题

#### 临界区

* 一个程序运行多个线程本身是没有问题的
* 问题出在多个线程访问共享资源
  * 多个线程读共享资源其实也没有问题
  * 在多个线程对共享资源**读写**操作时发生指令交错，就会出现问题

* 一段代码块内**如果存在对共享资源的多线程读写操作，称这段代码块为临界区**

#### 竞态条件

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件

#### 互斥解决

为了避免临界区的竞态条件发生，有多种手段可以达到目的

* 阻塞式的解决方案：synchronized，lock
* 非阻塞式的解决方案：原子变量



### synchronized 解决方案

synchronized，俗称 **对象锁** ，采用互斥的方式让同一时刻至多只有一个线程能持有 **对象锁** ，其他线程再想获取这个 **对象锁** 时就会被阻塞住。这样就能保证拥有锁的线程可以安全的执行临界区内的代码，不用担心线程上下文切换

>虽然 java 中互斥和同步都可以采用 synchronized 关键字来完成，但它们还是有区别的：
>
>* 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区代码
>* 同步是由于线程执行的先后、顺序不同，需要一个线程等待其他线程运行到某个点

实际是用 **对象锁** 保证了临界区内代码的原子性，临界区内的代码对外是不可分割的，不会被线程切换所打断

* 修饰**实例方法**，对**当前实例对象 this 加锁**
* 修饰**静态方法**，对**当前类的Class对象加锁**
* 修饰**代码块**，**指定一个加锁的对象，给对象加锁**



### 变量的线程安全分析

#### 成员变量和静态变量是否线程安全

* 如果没有共享，则线程安全
* 如果被共享了，根据它们的状态是否能够改变，又分为两种情况
  * 如果只有读操作，则线程安全
  * 如果有读写操作，则这段代码是临界区，需要考虑线程安全

#### 局部变量是否线程安全

* 局部变量是线程安全的
* 但局部变量引用的对象未必
  * 如果该对象没有逃离方法的作用范围，它是线程安全的
  * 如果该对象逃离方法的作用范围，需要考虑线程安全



### Monitor概念

#### Java对象头构成

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200713104455729.png" alt="image-20200713104455729" style="zoom: 67%;" />

* 对象头
  * **Mark Word**（标记字段）：默认存储对象的 hashcode，分代年龄和锁标志位信息。它会根据对象的状态复用自己的存储空间，也就是说在运行期间 Mark Word 里存储的数据会随着 **锁标志位** 的变化而变化
  * **Klass Point**（类型指针）：**对象指向它的类元数据的指针**，虚拟机通过这个指针来确定这个对象是哪个类的实例
* 实例数据
  * 这部分主要是存放类的数据信息，父类的信息
* 对齐填充
  * 由于虚拟机要求对象起始地址必须是8字节的整数倍，填充数据不是必须存在的，仅仅是为了字节对齐

> 一个空对象占8字节，因为对齐填充的关系，不到8个字节对齐填充会自动补齐

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200712221143.png"  />

#### Monitor

![image-20200719223648861](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719223648861.png)

* 刚开始 Monitor 的 Owner 为 null
* 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2 ， Monitor 中只能有一个Owner
* 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入EntryList Blocked 状态
* Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争是非公平的
* WaitSet 中的线程是之前获得过锁，但条件不满足进入 Waiting 状态的线程

> * synchronized 必须是进入同一个对象的 monitor
> * 不加 synchronized 的对象不会关联 monitor，不遵从以上规则



### Synchronized 原理进阶

#### 轻量级锁

如果一个对象虽然有多线程访问，但多线程访问的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化

* 创建**锁记录**（Lock Record）对象，每个线程对应的栈帧都会包含一个 **锁记录** 的结构，内部可以存储锁定对象的 Mark Word

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200712225008.png)

* 让锁记录中的 **Object reference 指向锁对象**，尝试使用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录

  ![image-20200712225249200](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200712225249200.png)

* 如果 cas 替换成功，**对象头中存储了锁记录地址和状态 00**，表示由该线程给对象加锁

  ![](https://raw.githubusercontent.com/whn961227/images/master/data/20200712225431.png)

* 如果 cas 失败，有两种情况
  * 如果是其他线程已经持有了该 Object 的轻量级锁，这时表明有**竞争**，进入**锁膨胀**过程
  * 如果是自己执行了 synchronized **锁重入**，那么**再添加一条 Lock Record 作为重入的计数**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200712225746648.png" alt="image-20200712225746648" style="zoom:80%;" />

* 当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示**重入计数减 1**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200712225923341.png" alt="image-20200712225923341" style="zoom: 80%;" />

* 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时**使用 cas 将 Mark Word 的值恢复给对象头**
  * 成功，则解锁成功
  * 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入**重量级锁解锁**流程

#### 锁膨胀

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其他线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁

* 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁

  <img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200713094139464.png" alt="image-20200713094139464" style="zoom: 33%;" />

* 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程

  * **为 Object 对象申请 Monitor 锁**，**让 Object 指向重量级锁地址**
  * 然后**自己进入 Monitor 的 EntryList Blocked 状态**

  <img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200713094510276.png" alt="image-20200713094510276" style="zoom: 50%;" />

* 当 Thread-0 退出同步块解锁时，使用 CAS 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级锁解锁流程，即**按照 Monitor 地址找到 Monitor 对象**，**设置 Owner 为 null**，**唤醒 EntryList 中 Blocked 线程**

#### 自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞

#### 偏向锁

轻量级锁在没有竞争时，每次重入仍然需要执行 CAS 操作

Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。

禁用偏向锁 VM 参数 -XX:-UseBiasedLocking，直接使用轻量级锁

#### 偏向状态

一个对象创建时：

* 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的 thread、epoch、age 都为 0
* 偏向锁默认是 **延迟** 的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数 -XX:BiasedLockingStartupDelay = 0 来禁用延迟
* 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、age 都为 0，第一次用到 hashcode 时才会赋值

##### 撤销-调用对象 hashCode

调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，如果调用 hashCode 会导致偏向锁被撤销

* 轻量级锁会在 **锁记录** 中记录 hashCode
* 重量级锁会在 **Monitor** 中记录 hashCode

##### 撤销-其他线程使用对象

当有其他线程使用偏向锁对象时，会将偏向锁升级为轻量级锁

##### 撤销-调用 wait/notify

##### 批量重定向

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍然有机会重新偏向 T2，重偏向会重置对象的 Thread ID

当撤销偏向锁阈值超过 20 次后，jvm 会给这些对象加锁时重新偏向至加锁线程

##### 批量撤销

当撤销偏向锁阈值超过 40 次后，整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的

##### 锁消除



### 原理之 wait/notify

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200714105927.png" style="zoom:33%;" />

* Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
* Blocked 和 Waiting 的线程都处于阻塞状态，不占用 CPU 时间片
* **Blocked 线程会在 Owner 线程释放锁时唤醒**
* **WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒**，但**唤醒后**不意味着立刻获得锁，**仍需进入 EntryList 重新竞争**

#### API介绍

* obj.wait() 让进入 object monitor 的线程到 waitSet 等待
* obj.notify() 在 object上 正在 waitSet 等待的线程中**挑一个唤醒**
* obj.notifyAll() 让 object 上正在 waitSet 等待的线程**全部唤醒**

属于 Object 对象的方法，**必须获得此对象的锁**，才能调用这几个方法



### sleep(long n) 和 wait(long n) 的区别

1. **sleep**是 **Thread** 方法，而 **wait** 是 **Object** 的方法
2. sleep **不需要**强制和 synchronized 配合使用，但 wait **需要**和 synchronized 一起用
3. **sleep** 在睡眠的同时，**不会释放对象锁**，但 **wait** 在等待的时候会**释放对象锁**
4. 线程调用两个方法都是进入 TIMED-WAITING 状态



### 同步模式之保护性暂停

用在一个线程等待另一个线程的执行结果

要点：

* 有一个结果需要从一个线程传递到另一个线程，让它们关联同一个 GuardedObject
* 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
* JDK 中，join 的实现、Future 的实现，采用的就是此模式
* 因为要等待另一方的结果，因此归类为同步模式

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200720104813.png)

```java
class GuardedObject {
    // 结果
    private Object response;

    // 获取结果
    // timeout 表示等待时长
    public Object get(long timeout){
        synchronized (this) {
            // 开始时间
            long begin = System.currentTimeMillis();
            // 经历时间
            long passedTime = 0;
            // 没有结果
            while (response == null) {
                // 这轮时间应该等待的时间
                long waitTime = timeout - passedTime;
                // 经历时间超过了最大等待时间，退出循环
                if (waitTime <= 0)
                    break;
                try {
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 求得经历时间
                passedTime = System.currentTimeMillis() - begin;
            }
            return response;
        }
    }

    // 生产结果
    public void complete(Object response) {
        synchronized (this) {
            // 给结果成员变量赋值
            this.response = response;
            this.notifyAll();
        }
    }
}
```

### 保护性暂停 - 扩展

```java
class People extends Thread {
    @Override
    public void run() {
        // 收信
        GuardedObject guardedObject = Mailboxes.createGuardedObject();
        System.out.println("开始收信 id：" + guardedObject.getId());
        Object mail = guardedObject.get(5000);
        System.out.println("收到信 id：" + guardedObject.getId() + "， 内容：" + mail);
    }
}

class Postman extends Thread {
    private int id;
    private String mail;

    public Postman(int id, String mail) {
        this.id = id;
        this.mail = mail;
    }

    @Override
    public void run() {
        GuardedObject guardedObject = Mailboxes.getGuardedObject(id);
        System.out.println("开始送信 id：" + id + "，内容：" + mail);
        guardedObject.complete(mail);
    }
}

class Mailboxes {
    private static Map<Integer, GuardedObject> boxes = new Hashtable<>();

    private static int id = 1;
    // 产生唯一id
    private static synchronized int generateId() {
        return id++;
    }

    public static GuardedObject getGuardedObject(int id) {
        return boxes.remove(id);
    }

    public static GuardedObject createGuardedObject() {
        GuardedObject go = new GuardedObject(generateId());
        boxes.put(go.getId(), go);
        return go;
    }

    public static Set<Integer> getIds() {
        return boxes.keySet();
    }
}

class GuardedObject {

    // 标识 GuardedObject
    private int id;

    public GuardedObject(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    // 结果
    private Object response;

    // 获取结果
    // timeout 表示等待时长
    public Object get(long timeout){
        synchronized (this) {
            // 开始时间
            long begin = System.currentTimeMillis();
            // 经历时间
            long passedTime = 0;
            // 没有结果
            while (response == null) {
                // 这轮时间应该等待的时间
                long waitTime = timeout - passedTime;
                // 经历时间超过了最大等待时间，退出循环
                if (waitTime <= 0)
                    break;
                try {
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 求得经历时间
                passedTime = System.currentTimeMillis() - begin;
            }
            return response;
        }
    }

    // 生产结果
    public void complete(Object response) {
        synchronized (this) {
            // 给结果成员变量赋值
            this.response = response;
            this.notifyAll();
        }
    }
}
```



### 异步模式之生产者/消费者

要点：

* 与前面的保护性暂停中的 GuardObject 不同，不需要产生结果和消费结果的线程一一对应
* 消费队列可以用来平衡生产和消费的线程资源
* 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
* **消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据**
* JDK 中各种阻塞队列，采用的就是这种模式

```java
// 消息队列类，java 线程之间通信
class MessageQueue {
    // 消息的队列集合
    private LinkedList<Message> list = new LinkedList<>();
    // 队列容量
    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    // 获取消息
    public Message take() {
        // 检查队列是否为空
        synchronized (list) {
            while (list.isEmpty()) {
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 从队列头部获取消息返回
            Message message = list.removeFirst();
            list.notifyAll();
            return message;
        }
    }

    // 存入消息
    public void put(Message message) {
        synchronized (list) {
            while (list.size() == capacity) {
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 将消息加入队列尾部
            list.addLast(message);
            list.notifyAll();
        }
    }
}

final class Message {
    private int id;
    private Object value;

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public Object getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", value=" + value +
                '}';
    }
}
```



### Park 与 Unpark

```java
LockSupport.park(); // 暂停当前线程
LockSupport.unpark(); // 恢复某个线程的运行
```

**与 Object 的 wait 与 notify 相比**

* wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark不必
* park 与 unpark 是以线程为单位来 **阻塞** 和 **唤醒** 线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程
* park 与 unpark 可以先 unpark，而 wait 与 notify 不能先 notify

#### 原理之 park & unpark

每个线程都有自己的一个 Parker 对象，由三部分组成 `_counter`，`_cond`，`_mutex`

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200714164718.png" style="zoom: 25%;" />

1. 当前线程调用 Unsafe.park() 方法
2. 检查 _counter，本情况为 0，这时，获得 _mutex 互斥锁
3. 线程进入 _cond 条件变量阻塞
4. 设置 _counter = 0

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200714165624.png" style="zoom: 25%;" />

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1

2. 唤醒 _cond 条件变量中的 Thread_0
3. Thread_0 恢复运行
4. 设置 _counter 为 0

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200714165926.png" style="zoom:25%;" />

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为1
2. 当前线程调用 Unsafe.park() 方法
3. 检查 _counter，本情况为 1，这时线程无需阻塞，继续运行
4. 设置 _counter 为 0



### 多把锁

将锁的粒度细分：

* 好处，可以增强并发度
* 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁



### 活跃性

#### 死锁

一个线程需要同时获取多把锁，这时就容易发生死锁

t1 线程 获得 A 对象 锁，接下来想获取 B 对象 的锁

t2 线程 获得 B 对象 锁，接下来想获取 A 对象 的锁

```java
Object A = new Object();
Object B = new Object();

Thread t1 = new Thread(()->{
   synchronized(A){
       sleep(1);
       synchronized(B){
           
       }
   } 
}, "t1");

Thread t2 = new Thread(()->{
   synchronized(B){
       sleep(0.5);
       synchronized(A){
           
       }
   } 
}, "t2");

t1.start();
t2.start();
```

#### 定位死锁

* 检测死锁可以使用 jconsole 工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁

#### 活锁

活锁出现在两个线程相互改变对方的结束条件，最后谁也无法结束

#### 饥饿

一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束



### ReentrantLock

相比于 synchronized ，具备如下特点

* **可中断**
* **可以设置超时时间**
* **可以设置公平锁**
* **支持多个条件变量**

与 synchronized 一样，都支持可重入

```java
// 获取锁
reentrantLock.lock();
try {
    // 临界区
} finally {
    // 释放锁
    reentrantLock.unlock();
}
```

#### 可重入

**可重入** 是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获得这把锁

如果是 **不可重入锁**，那么第二次获得锁时，自己也会被挡住

#### 可打断

停止无限制等待，被动

```java
reentrantLock.lockInterruptibly();
```

#### 锁超时

主动

```java
boolean tryLock()
boolean tryLock(long, TimeUnit)
```

#### 公平锁

ReentrantLock 默认是不公平的

#### 条件变量

synchronized 中也有条件变量，就是 waitSet，当条件不满足时进入 waitSet 等待

ReentrantLock 的条件变量比 synchronized 强大之处在于，它支持多个条件变量

* synchronized 是那些不满足条件的线程都在一个waitSet 中等待
* 而 ReentrantLock 支持多个 condition，唤醒时是按照 condition 来唤醒

使用流程

* **await 前需要获得锁**
* await 执行后，会**释放锁**，进入 **conditionObject** 等待
* await 的线程被唤醒(signal()、signalAll())（或打断、或超时）去重新竞争 lock 锁
* 竞争 lock 锁成功后，从 await 后继续执行



### 设计模式-固定运行顺序

#### wait & notify

```java
/**
先打印2，再打印1
*/
static final Object lock = new Object();
static boolean t2runned = false;

public static void main(String[] args) {
	Thread t1 = new Thread(()->{
       synchronized (lock) {
           while (!t2runned) {
               try{
                   lock.wait();
               } catch (InterruptedException e){
                   e.printStackTrace();
               }
           }
           log.debug("1");
       }
    }, "t1");
    
    Thread t2 = new Thread(()->{
       synchronized (lock) {
           log.debug("2");
           t2runned = true;
           lock.notify();
       }
    }, "t2");
    
    t1.start();
    t2.start();
}
```

#### await & signal

```java
/**
先打印2，再打印1
*/
static final ReentrantLock LOCK = new ReentrantLock();
static final Condition condition = LOCK.newCondition();
static boolean t2runned = false;

public static void main(String[] args) {
    Thread t1 = new Thread(()->{
        try {
            LOCK.lock();
            while(!t2runned){
                condition.await();
            }
            System.out.println("1");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            LOCK.unlock();
        }
    }, "t1");

    Thread t2 = new Thread(()->{
        try {
            LOCK.lock();
            System.out.println("2");
            t2runned = true;
            condition.signal();
        } finally {
            LOCK.unlock();
        }
    }, "t2");

    t1.start();
    t2.start();
}
```

#### park & unpark

```java
Thread t1 = new Thread(()->{
    LockSupport.park();
    System.out.println("1");
}, "t1");

Thread t2 = new Thread(()->{
    System.out.println("2");
    LockSupport.unpark(t1);
}, "t2");

t1.start();
t2.start();
```



### 设计模式-交替输出

#### wait & notify

```java
/**
abcabcabcabcabc
*/
public class Test {
    private int flag;
    private int loopNumber;

    public Test(int flag, int loopNumber) {
        this.flag = flag;
        this.loopNumber = loopNumber;
    }

    public void print(String str, int waitFlag, int nextFlag) {
        for (int i = 0; i < loopNumber; i++) {
            synchronized (this) {
                while (flag != waitFlag) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.print(str);
                flag = nextFlag;
                this.notifyAll();
            }
        }
    }

    public static void main(String[] args) {
        Test test = new Test(1, 5);
        new Thread(()->{
            test.print("a",1,2);
        }).start();
        new Thread(()->{
            test.print("b",2,3);
        }).start();
        new Thread(()->{
            test.print("c",3,1);
        }).start();
    }
}
```

#### await & signal

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        Awaitsignal awaitsignal = new Awaitsignal(5);
        Condition a = awaitsignal.newCondition();
        Condition b = awaitsignal.newCondition();
        Condition c = awaitsignal.newCondition();
        new Thread(()->{
            awaitsignal.print("a",a,b);
        }).start();
        new Thread(()->{
            awaitsignal.print("b",b,c);
        }).start();
        new Thread(()->{
            awaitsignal.print("c",c,a);
        }).start();

        Thread.sleep(1000);
        awaitsignal.lock();
        a.signal();
        awaitsignal.unlock();
    }
}

class Awaitsignal extends ReentrantLock {
    private int loopNumger;

    public Awaitsignal(int loopNumger) {
        this.loopNumger = loopNumger;
    }

    public void print(String str, Condition current, Condition next) {
        for (int i = 0; i < loopNumger; i++) {
            lock();
            try {
                current.await();
                System.out.print(str);
                next.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                unlock();
            }
        }
    }
}
```

#### park & unpark

```java
public class Test {
    static Thread a;
    static Thread b;
    static Thread c;

    public static void main(String[] args) throws InterruptedException {
        ParkUnpark pu = new ParkUnpark(5);
        a = new Thread(() -> {
            pu.print("a", b);
        });
        b = new Thread(() -> {
            pu.print("b", c);
        });
        c = new Thread(() -> {
            pu.print("c", a);
        });
        a.start();
        b.start();
        c.start();

        LockSupport.unpark(a);
    }
}

class ParkUnpark {
    private int loopNumber;

    public ParkUnpark(int loopNumber) {
        this.loopNumber = loopNumber;
    }

    public void print(String str, Thread next) {
        for (int i = 0; i < loopNumber; i++) {
            LockSupport.park();
            System.out.print(str);
            LockSupport.unpark(next);
        }
    }
}

```



### Volatile 原理

volatile 的底层实现原理是**内存屏障**

* 对 volatile 变量的**写指令后会加入写屏障**
* 对 volatile 变量的**读指令前会加入读屏障**

用来修饰成员变量和静态成员变量，可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取值，线程操作 volatile 变量都是直接操作主存

仅用在一个写线程，多个读线程的情况

#### 特性

* 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的（实现可见性）
* 禁止指令重排序（实现有序性）
* volatile只能保证对单次读/写的原子性，i++这种操作不能保证原子性

> synchronized 语句块既可以保证代码块的原子性，也同时可以保证代码块内变量的可见性，但缺点是synchronized 是属于重量级操作，性能相对较低

#### 如何保证可见性

* **写屏障保证在该屏障之前，对共享变量的改动，都同步到主存中**
* **读屏障保证在该屏障之后，对共享变量的读取，加载的是主存中的最新数据**

#### 如何保证有序性

* 写屏障会确保指令重排序时，**不会将写屏障之前的代码排在写屏障之后**
* 读屏障会确保指令重排序时，**不会将读屏障之后的代码排在读屏障之前**

**不能解决指令交错**：

* 写屏障仅仅时保证之后的读能够读到最新结果，但不能保证读跑到它前面去
* 而有序性的保证也只是保证了本线程内相关代码不被重排序

#### Volatile 底层

当多个处理器的任务都涉及到同一块主内存区域时，将可能导致各自的缓存数据不一致，为了解决一致性问题，需要各个处理器访问缓存时遵循一些协议

#### Intel 的 MESI（缓存一致性）协议

当 CPU 写数据时，如果发现操作的变量是共享变量，即在其他 CPU 中也存在该变量的副本，会发出信号通知其他 CPU 将该变量的缓存行置为无效状态，因此当其他 CPU 需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会内存重新读取

#### 嗅探

每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里

##### 缺点

由于 volatile 的 MESI 缓存一致性协议，需要不断的**从主内存嗅探**和 cas 不断循环，无效交互会导致总线带宽达到峰值，所以不要大量使用 volatile



### 设计模式 - 两阶段终止 - volatile

```java
class TwoPhaseTermination {
    private Thread monitor;
    private volatile boolean stop = false;

    // 启动监控线程
    public void start() {
        monitor = new Thread(() -> {
            while (true) {
                Thread thread = Thread.currentThread();
                if (stop) {
                    System.out.println("料理后事...");
                    break;
                }
                try {
                    Thread.sleep(1000); // 打断情况 1
                    System.out.println("执行监控记录"); // 打断情况 2
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        monitor.start();
    }

    // 停止监控线程
    public void stop() {
        stop = true;
        monitor.interrupt();
    }
}
```



### 设计模式 - 犹豫模式

Balking（犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回

```java
class TwoPhaseTermination {
    private Thread monitor;
    private volatile boolean stop = false;
    // 判断是否执行过 start 方法
    private boolean starting = false;

    // 启动监控线程
    public void start() {
        synchronized (this) {
            if(starting)
                return;
            starting = true;
        }
        monitor = new Thread(() -> {
            while (true) {
                Thread thread = Thread.currentThread();
                if (stop) {
                    System.out.println("料理后事...");
                    break;
                }
                try {
                    Thread.sleep(1000); // 打断情况 1
                    System.out.println("执行监控记录"); // 打断情况 2
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        monitor.start();
    }

    // 停止监控线程
    public void stop() {
        stop = true;
        monitor.interrupt();
    }
}
```



### JMM（Java 内存模型）

所有的共享变量都存储于主内存，每一个线程有自己的工作内存，线程的工作内存，保留了被线程使用的变量的工作副本

**线程对变量的所有的操作（读，取）都必须在工作内存中完成**，而不能直接读写主内存中的变量

**不同线程之间不能直接访问对方工作内存中的变量**，线程间变量的值的传递需要**通过主内存中转**完成

**本地内存和主内存的关系**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200708161935.png" style="zoom:33%;" />



### 可见性问题解决方案

* **加锁**

  某一个线程进入 synchronized 代码块前后，线程会获得锁，**清空工作内存**，从主内存拷贝共享变量最新的值到工作内存中成为副本，执行代码，将修改后的副本的值刷新回主内存中，线程释放锁

  而获取不到锁的线程会阻塞等待，所以变量的值肯定一直都是最新的

* **Volatile 修饰共享变量**

  每个线程操作数据的时候会把数据从主内存读取到自己的工作内存，如果操作了数据并且写回了，其他已经读取的线程的变量副本就失效了



### double-checked locking 问题

```java
public final class Singleton {
    private Singleton() {}
    private static Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized 代码块
        if (INSTANCE == null) {
            // 首次访问会同步，之后的使用没有 synchronized
            synchronized (Singleton.class) {
                // 也许有其他线程已经创建实例，所以再判断一次
                if (INSTANCE == null) 
                    INSTANCE = new Singleton();
            }
        }
        return INSTANCE;
    }
}
```

以上的实现特点是：

* 懒惰实例化
* 首次使用 getInstance() 才使用 synchronized 加锁，后续使用时无需加锁
* 关键的一点：第一个 if 使用了 INSTANCE 变量，是在同步块之外

#### 加 volatile 关键字

```java
public final class Singleton {
    private Singleton() {}
    private static volatile Singleton INSTANCE = null;
    public static Singleton getInstance() {
        // ....
    }
}
```



### Happens-before

Happens-before 规定了对共享变量的写操作对其他线程的读操作可见，它是可见性与有序性的一套规则总结，抛开以下 happens-before 规则 ，JMM 并不能保证一个线程对共享变量的写，对于其他线程对该共享变量的读可见

* 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其他线程对该变量的读可见

  ```java
  static int x;
  static Object m = new Object();
  
  new Thread(()->{
      synchronized(m){
          x = 10;
      }
  },"t1").start();
  
  new Thread(()->{
      synchronized(m){
          System.out.println(x);
      }
  },"t2").start();
  ```

* 线程对 volatile 变量的写，对接下来其他线程对该变量的读可见

  ```java
  volatile static int x;
  
  new Thread(()->{
      x = 10;
  }, "t1").start();
  
  new Thread(()->{
      System.out.println(x);
  }, "t2").start();
  ```

* 线程 start 前对变量的写，对该线程开始后对该变量的读可见

  ```java
  static int x;
  
  x = 10;
  
  new Thread(()->{
      System.out.println(x);
  }, "t2").start();
  ```

* 线程结束前对变量的写，对其他线程得知它结束后的读可见（比如其他线程调用 t1.isAlive() 或 t1.join() 等待它结束）

  ```java
  static int x;
  
  Thread t1 = new Thread(()->{
      x = 10;
  }, "t1");
  t1.start();
  
  t1.join();
  System.out.println(x);
  ```

* 线程 t1 打断 t2 （interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过t2.interrupted 或 t2.isInterrupted）

  ```java
  static int x;
  
  public static void main(String[] args) {
      Thread t2 = new Thread(()->{
          while (true) {
              if (Thread.currentThread().isInterrupted()){
                  System.out.println(x);
                  break;
              }
          }
      }, "t2");
      t2.start();
  
      new Thread(()->{
          try {
              TimeUnit.SECONDS.sleep(1);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          x = 10;
          t2.interrupt();
      }, "t1").start();
  
      while (!t2.isInterrupted())
          Thread.yield();
      System.out.println(x);
  }
  ```

* 对变量默认值（0，false，null）的写，对其他线程对该变量的读可见

* 具有传递性，配合volatile的禁止指令重排

  ```java
  volatile static int x;
  static int y;
  
  new Thread(()->{
      y = 10;
      x = 20;
  }, "t1").start();
  
  new Thread(()->{
      // x=20 对 t2 可见，同时 y=10 也对 t2 可见
      System.out.println(x);
  }, "t2").start();
  ```



### 线程安全单例习题

>饿汉式：类加载就会导致该单实例对象被创建
>
>懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会被创建

实现1：

```java
// 问题1：为什么加final ====> 防止子类覆盖父类中的方法，破坏单例
// 问题2：如果实现了序列化接口，如何防止反序列化破坏单例
public final class Singleton implements Serializable {
    // 问题3：为什么设置为私有？是否能防止反射创建新的实例？ ====> 如果不设置为私有，其他的类都能创建对象，无法保证单例；不能防止
    private Singleton(){}
    // 问题4：这样初始化能不能保证单例对象创建时的线程安全？ ====> 能，静态成员变量的初始化操作在类加载时完成，由JVM保证代码的线程安全性
    private static final Singleton INSTANCE = new Singleton();
    // 问题5：为什么提供静态方法而不是直接将 INSTANCE 设置为 public，说出你知道的理由 ====> 方法能提供更好的封装性，能实现懒惰的初始化；创建单例对象时能提供更多的控制；提供泛型的支持
    public static Singleton getInstance() {
        return INSTANCE;
    }
    // 防止反序列化破坏单例
    public Object readResolve() {
        return INSTANCE;
    }
}
```

实现2：

```java
// 问题1：枚举单例是如何限制实例个数的 ===> INSTANCE 相当于是枚举类的静态成员变量
// 问题2：枚举单例在创建时是否有并发问题 ===> 没有，静态成员变量的初始化操作在类加载时完成，由JVM保证代码的线程安全性
// 问题3：枚举单例能否反射破坏单例 ===> 不能
// 问题4：枚举单例能否被反序列化破坏单例 ===> 不能
// 问题5：枚举单例属于懒汉式还是饿汉式 ===> 饿汉式
// 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做 ===> 用构造方法
enum Singleton {
    INSTANCE;
}
```

实现3：

```java
public final class Singleton {
    private Singleton(){}
    private static Singleton INSTANCE = null;
    // 分析这里的线程安全，并说明有什么缺点 ====> synchronized 加在静态方法上，相当于把锁加在了Singleton.class上，类对象和静态成员变量是对应的，就能提供对静态成员变量的线程安全保护；锁的范围大，每次调用都会加锁，性能低下
    public static synchronized Singleton getInstance() {
        if (INSTANCE != null)
            return INSTANCE;
        INSTANCE = new Singleton();
        return INSTANCE;
    }
}
```

实现4：

```java
public final class Singleton{
    private Singleton(){}
    // 问题1：解释为什么加volatile ===> 防止指令重排序
    private static volatile Singleton INSTANCE = null;
    // 问题2：对比实现3，说出这样的意义 ===> 缩小加锁范围，提升性能
    public static Singleton getInstance(){
        if (INSTANCE != null)
            return INSTANCE;
        synchronized(Singleton.class) {
            // 问题3：为什么还要在这里加非空判断，之前不是判断过了吗 ====> 防止并发情况下，第一次创建的单例对象不会被覆盖
            if (INSTANCE != null) 
                return INSTANCE;
            INSTANCE = new Singleton();
            return INSTANCE;
        }
    }
}
```

实现5：

```java
public final class Singleton{
    private Singleton(){}
    // 问题1：属于饿汉式还是懒汉式 ===> 懒汉式
    private static class LazyHolder {
        static final Singleton INSTANCE = new Singleton();
    }
    // 问题2：在创建时是否有并发问题 ====> 不会，静态成员变量的初始化操作在类加载时完成，由JVM保证代码的线程安全性
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```



### CAS 与 volatile

#### CAS 工作方式

```java
public void withdraw(Integer amount) {
    while (true) {
        int prev = balance.get();
        int next = prev - amount;
        // 比较并设置值
        if (balance.compareAndSet(prev, next)) {
            break;
        }
    }
}
```

CAS 必须借助 volatile 才能读取到共享变量的最新值来实现 **比较并交换** 的结果

#### CAS 的特点

结合 CAS 和 volatile 可以实现无锁并发，适用于线程数少、多核CPU的场景下

* CAS 是基于乐观锁的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，继续重试
* sychronized 是基于悲观锁的思想：最悲观的估计，防着其他线程来修改共享变量，上锁其他线程都不能改，解锁之后其他线程才有机会
* CAS 体现的是无锁并发，无阻塞并发
  * 因为没有使用 synchronized，所以线程不会阻塞，效率提升
  * 但如果竞争激烈，重试必然频繁发生，效率会受到影响

#### ABA问题

主线程仅能判断出共享变量的值与最初值是否相同，不能感知到这种从 A 改为 B 再改回 A 的情况，如果主线程希望：

只要有其他线程 **改动了** 共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要再加一个**版本号**



### 不可变类的设计

#### final 的使用

发现该类、类中所有属性都是 final 的

* 属性用 final 修饰保证了该属性是只读的，不能修改
* 类用 final 修饰保证了该类中的方法不能被覆盖，防止子类无意间破坏不可变性

#### 保护性拷贝

构造新字符串对象时，会生成新的 char[] value，对内容进行复制。通过创建副本对象来避免共享的手段称为 **保护性拷贝**



### 享元模式

当需要重用数量有限的同一类对象时



### 自定义连接池



### 设置 final 变量的原理

final 变量的赋值也会通过 putfield 指令来完成，同样在这条指令之后也会加入写屏障，保证在其他线程读到它的值时不会出现为 0 的情况

### 获取 final 变量的原理





### 自定义线程池

```java
public class TestPool {
    public static void main(String[] args) {
        ThreadPool threadPool = new ThreadPool(1, 1000, TimeUnit.MILLISECONDS
                , 1, (queue, task) -> {
            // 1) 死等
//            queue.put(task);
            // 2) 带超时等待
//            queue.offer(task, 500, TimeUnit.MILLISECONDS);
            // 3) 让调用者放弃任务执行
//            System.out.println("放弃");
            // 4) 让调用者抛出异常
//            throw new RuntimeException("任务执行失败" + task);
            // 5) 让调用者自己执行任务
//            task.run();
        });
    }
}

@FunctionalInterface // 拒绝策略
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}

class ThreadPool {
    // 任务队列
    private BlockingQueue<Runnable> taskQueue;
    // 线程集合
    private HashSet<Worker> workers = new HashSet<>();
    // 核心线程数
    private int coreSize;

    // 获取任务的超时时间
    private long timeout;

    private TimeUnit timeUnit;

    private RejectPolicy<Runnable> rejectPolicy;

    // 执行任务
    public void execute(Runnable task) {
        // 当任务数没有超过 coreSize 时，直接交给 worker 对象执行
        // 如果任务数超过 coreSize 时，加入任务队列暂存
        synchronized (workers) {
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                workers.add(worker);
                worker.start();
            } else {
//                taskQueue.put(task);
                // 1) 死等
                // 2) 带超时等待
                // 3) 让调用者放弃任务执行
                // 4) 让调用者抛出异常
                // 5) 让调用者自己执行任务
                taskQueue.tryPut(rejectPolicy, task);
            }
        }
    }

    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            // 执行任务
            // 1) 当 task 不为空，执行任务
            // 2) 当 task 执行完毕，接着从任务队列获取任务执行
            while (task != null || (task = taskQueue.take()) != null) {
                try {
                    task.run();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }

            synchronized (workers) {
                workers.remove(this);
            }
        }
    }
}

// 阻塞队列
class BlockingQueue<T> {
	// 任务队列
    private Deque<T> queue = new ArrayDeque<>();
	// 锁
    private ReentrantLock lock = new ReentrantLock();

    private Condition fullWaitSet = lock.newCondition();

    private Condition emptyWaitSet = lock.newCondition();

    private int capacity;

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    // 带超时的阻塞获取
    public T pull(long timeout, TimeUnit unit) {
        lock.lock();
        try {
            // 将 timeout 同一转换为 纳秒
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    // 返回的是剩余时间
                    if (nanos <= 0)
                        return null;
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞获取
    public T take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞添加
    public void put(T task) {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                try {
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            queue.addLast(task);
            emptyWaitSet.signal();
        } finally {
            lock.unlock();
        }
    }

    // 带超时的阻塞添加
    public boolean offer(T task, long timeout, TimeUnit unit) {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    if (nanos <= 0)
                        return false;
                    nanos = fullWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            queue.addLast(task);
            emptyWaitSet.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

    public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            // 判断队列是否已满
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                queue.addLast(task);
                emptyWaitSet.signal();
            }
        } finally {
            lock.unlock();
        }
    }
}
```



### ThreadPoolExecutor

#### 线程池状态

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量

| 状态名     | 高 3 位 | 接收新任务 | 处理阻塞队列任务 | 说明                                      |
| ---------- | ------- | ---------- | ---------------- | ----------------------------------------- |
| RUNNING    | 111     | Y          | Y                |                                           |
| SHUTDOWN   | 000     | N          | Y                | 不会接收新任务，但会处理阻塞队列剩余任务  |
| STOP       | 001     | N          | N                | 会中断正在执行的任务，并抛弃阻塞队列任务  |
| TIDYING    | 010     | -          | -                | 任务全执行完毕，活动线程为0，即将进入终结 |
| TERMINATED | 011     | -          | -                | 终结状态                                  |

#### 构造方法

```java
public ThreadPoolExecutor(int corePoolSize, // 核心线程数目（最多保留的线程数）
                         int maximumPoolSize, // 最大线程数目
                         long keepAliveTime, // 生存时间 - 针对救急线程
                         TimeUnit unit, // 时间单位 - 针对救急线程
                         BlockingQueue<Runnable> workQueue, // 阻塞队列
                         ThreadFactory threadFactory, // 线程工厂 - 可以为线程创建时起名字
                         RejectedExecutionHandler handler) // 拒绝策略
```

* 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务
* 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入 workQueue 队列排队，直到有空闲的线程
* 如果队列选择了有界队列，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线程来救急
* 如果线程达到了 maximumPoolSize 仍然有新任务这时会执行拒绝策略。
* 当高峰过去后，超过 corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由 keepAliveTime 和 unit 来控制

#### newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 
                                  0L, TimeUnit.MILLISECONDS, 
                                  new LinkedBlockingQueue<Runnable>())
}
```

特点：

* **核心线程数 == 最大线程数（没有救急线程被创建），因此无需超时时间**
* 阻塞队列是**无界**的，可以放任意数量的任务

> 适用于任务量已知，相对耗时的任务

#### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 
                                  60L, TimeUnit.SECONDS, 
                                  new SynchronousQueue<Runnable>())
}
```

特点：

* 核心线程数是 0，最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着
  * **全是救急线程（60s 后可以回收）**
  * **救急线程可以无限创建**
* 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的

> 整个线程池表现为线程数会根据任务量不断增加，没有上限，当任务执行完毕，空闲 1 分钟后释放线程
>
> 适合任务数比较密集，但每个任务执行时间较短的情况

#### newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService(
    	new ThreadPoolExecutor(1, 1,
                              0L, TimeUnit.MILLISECONDS,
                              new LinkedBlockingQueue<Runnable>())
    );
}
```

使用场景：

希望多个任务排队执行。**线程数固定为 1**，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程也不会被释放

区别：

* 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会创建一个线程，保证池的正常工作
* Executors.newSingleThreadExecutor() 线程个数始终为 1，不能修改
  * FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法
* Executors.newFixedThreadPool(1) 初始时为 1，以后还可以修改
  * 对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改



### AQS原理

#### 概述

全称是 AbstractQueuedSynchronized，是**阻塞式锁和相关的同步器工具的框架**

特点：

* 用 **state 属性**来**表示资源的状态**（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
  * getState - 获取 state 状态
  * setState - 设置 state 状态
  * **compareAndSetState** - 乐观锁机制设置 state 状态
  * **独占模式**是**只有一个线程能够访问资源**，而**共享模式**可以**允许多个线程访问资源**
* 提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList
* 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet

### AQS自定义锁

```java
// 自定义锁（不可重入锁）
class MyLock implements Lock {

    // 独占锁 同步器类
    class MySync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {
            if(compareAndSetState(0, 1)) {
                // 加上了锁，设置 owner 为当前线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override // 是否持有独占锁
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        public Condition newCondition() {
            return new ConditionObject();
        }
    }

    private MySync sync = new MySync();

    @Override // 加锁（不成功会进入等待队列等待）
    public void lock() {
        sync.acquire(1);
    }

    @Override // 加锁，可打断
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override // 尝试加锁（一次）
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override // 尝试加锁，带超时
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override // 解锁
    public void unlock() {
        sync.release(1);
    }

    @Override // 创建条件变量
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```



### ReentrantLock原理

![image-20200718225738652](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200718225738652.png)

#### 非公平锁实现原理

##### 加锁解锁流程

先从构造器开始看，默认为**非公平锁**实现

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

**NonfairSync 继承自 AQS**

没有竞争时

![image-20200719102330812](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719102330812.png)

第一个竞争出现时

![image-20200719103752368](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719103752368.png)

Thread-1 执行了

1. CAS 尝试**将 state 由 0 改为 1**，结果**失败**
2. 进入 **tryAcquire 逻辑**，这时 state 已经是 1，结果仍然**失败**
3. 接下来进入 **addWaiter 逻辑**，**构造 Node 队列**
   * 图中黄色三角表示该 Node 的 **waitStatus 状态**，其中 0 为默认正常状态
   * Node 的创建是懒惰的
   * 其中**第一个 Node 称为 Dummy（哑元）或哨兵**，用来**占位**，并不关联线程

![image-20200719104259547](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719104259547.png)

当前线程进入 **acquireQueued 逻辑**

1. acquireQueued 会在第一个死循环中不断尝试获得锁，失败后进入 park 阻塞
2. 如果自己是紧邻着 head（排第二位），那么再次 tryAcquire  尝试获取锁，当然这时 state 仍为 1，失败
3. 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false

![image-20200719104809793](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719104809793.png)

4. shouldParkAfterFailedAcquire 执行完毕回到 acquireQueued，再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
5. 当再次进入 shouldParkAfterFailedAcquire 时，这时因为其前驱 node 的 waitStatus 已经是 -1，这次返回 true
6. 进入 parkAndCheckInterrupt，Thread-1 park（灰色表示）

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200719105300.png)

再次有多个线程经历上述过程竞争失败，变成这个样子

![image-20200719105535271](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719105535271.png)

Thread-0 释放锁，进入 tryRelease 流程，如果成功

* 设置 exclusiveOwnerThread 为 null
* state = 0

![image-20200719105632121](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719105632121.png)

当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程

找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，本例中即为 Thread-1

回到 Thread-1 的 acquireQueued 流程

![image-20200719110144462](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719110144462.png)

如果加锁成功（没有竞争），会设置

* exclusiveOwnerThread 为 Thread-1，state = 1
* head 指向刚刚 Thread-1 所在的 Node，该 Node 清空 Thread
* 原本的 head 因为从链表断开，而可被垃圾回收

如果这时候有其他线程来竞争（非公平的体现），例如这时有 Thread-4 来了

![image-20200719110515104](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719110515104.png)

如果不巧被 Thread-4 占了先

* Thread-4 被设置为 exclusiveOwnerThread，state = 1
* Thread-1 再次进入 acquireQueued 流程，获取锁失败，重新进入 park 阻塞

#### 可重入实现原理

```java
static final class NonfairSync extends Sync {
    // ...

    // Sync 继承过来的方法，方便阅读，放在此处
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果已经获得了锁，线程还是当前线程，表示发生了锁重入
        else if (current == getExclusiveOwnerThread()) {
            // state++
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    // Sync 继承过来的方法，方便阅读，放在此处
    protected final boolean tryRelease(int releases) {
        // state--
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 支持锁重入，只有 state 减为 0，才释放成功
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
}
```

 #### 条件变量实现原理

**每个条件变量其实就对应着一个等待队列**，其实现类是 ConditionObject

##### await流程

开始 Thread-0 持有锁，调用 await，进入 ConditionObject 的 addConditionWaiter 流程

创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0 ，加入等待队列尾部

![image-20200719114726037](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719114726037.png)

接下来进入 AQS 的 fullyRelease 流程，释放同步器上的锁

![image-20200719115006236](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719115006236.png)

unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功

![image-20200719115113208](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719115113208.png)

park 阻塞 Thread-0

![image-20200719115139561](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719115139561.png)

##### signal 流程

假设 Thread-1 要来唤醒 Thread-0

![image-20200719115233734](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719115233734.png)

进入 ConditionObject 的 doSignal 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node

![image-20200719154101256](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719154101256.png)

执行 transferForSignal 流程，将该 Node 加入 AQS 队列尾部，将 Thread-0 的 waitStatus 改为 0，Thread-3 的waitStatus 改为 -1

![image-20200719154855732](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719154855732.png)

Thread-1 释放锁，进入 unlock 流程



### 读写锁

#### ReentrantReadWriteLock

当读操作远远高于写操作时，这时候使用 **读写锁** 让 **读-读** 可以并发，提高性能

#### StampedLock



### Semaphore

信号量，用来限制能同时访问共享资源的线程上限

#### 加锁解锁流程

刚开始，permits（state）为 3，这时 5 个线程来获取资源

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719165149479.png" alt="image-20200719165149479"  />

假设其中 Thread-1，Thread-2，Thread-4 CAS 竞争成功，而 Thread-0 和 Thread-3 竞争失败，进入 AQS 队列 park 阻塞

![image-20200719165527654](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719165527654.png)

这时 Thread-4 释放了 permits，状态如下

![image-20200719165633672](https://raw.githubusercontent.com/whn961227/images/master/data/image-20200719165633672.png)

接下来 Thread-0 竞争成功，permits 再次设置为 0，设置自己为 head 节点，断开原来的 head 节点，unpark 接下来的 Thread-3 节点，但由于 permits 是 0，因此 Thread-3 在尝试不成功后再次进入 park 状态

![](https://raw.githubusercontent.com/whn961227/images/master/data/20200719170146.png)



### CountdownLatch

用来进行线程同步协作，等待所有线程完成倒计时

其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一



### CyclicBarrier

循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置 **计数个数**，每个线程执行到某个需要 **同步** 的时刻调用 await() 方法进行等待，当等待的线程数满足 **计数个数** 时，继续执行



### ThreadLocal

**作用：**主要是做**数据隔离**，**填充的数据只属于当前线程**，变量的数据对别的线程而言是相对隔离的，在多线程环境下，防止自己的变量被其他线程篡改

**应用场景**

Spring 采用 Threadlocal 的方式，来保证单个线程中的数据库操作使用的是同一个数据库连接，同时，采用这种方式可以使业务层使用事务时不需要感知并管理 connection 对象，通过传播级别，巧妙地管理多个事务配置之间的切换，挂起和恢复

#### 底层原理

**每个线程 Thread 都维护了自己的 threadLocals 变量**，所以在每个线程创建 ThreadLocal 的时候，实际上数据是存在自己线程 Thread 的 threadLocals 变量里面的，其他线程没法拿到，从而实现了隔离

##### ThreadLocalMap 底层结构

既然有个 Map 那他的数据结构其实是很像 HashMap 的，但是看源码可以发现，它并未实现 Map 接口，而且他的 Entry 是继承 WeakReference（弱引用）的，也没有看到 HashMap 中的 next，所以不存在链表

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727101052.png" style="zoom: 25%;" />

**为什么需要数组？没有链表如何解决 Hash 冲突**

用数组是因为我们开发过程中一个线程可以有多个 ThreadLocal 来存放不同类型的对象，但是它们都将放到你当前线程的 ThreadLocalMap 里，所以肯定要数组来存

ThreadLocalMap 在存储的时候会给每一个 ThreadLocal 对象一个 threadLocalHashCode，在插入过程中，根据 ThreadLocal 对象的 hash 值，定位到 table 中的位置 i，`int i = key.threadLocalHashCode & (len-1)`

然后会判断一下：如果当前位置为空，就初始化一个 Entry 对象放在位置 i 上

```java
if (k == null) {
    replaceStaleEntry(key, value, i);
    return;
}
```

如果位置 i 不为空，如果这个 Entry 对象的 key 正好是即将设置的 key，那么就刷新 Entry 中的 value

```java
if (k == key) {
    e.value = value;
    return;
}
```

如果位置 i 不为空，而且 key 不等于 entry，那就找下一个空位置，直到为空为止

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727103238.png" style="zoom: 50%;" />

这样的话，在 get 的时候，也会根据 ThreadLocal 对象的 hash 值，定位到 table 中的位置，然后判断该位置 Entry 对象中的 key 是否和 get 的 key 一致，如果不一致，就判断下一位置，set 和 get 如果冲突严重的话，效率很低

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    // get 的时候一样是根据 ThreadLocal 获取到 table 的 i 值，然后查找数据拿到后会对比 key 是否相等，if (e != null && e.get() == key)
    while (e != null) {
        ThreadLocal<?> k = e.get();
        // 相等就直接返回，不相等就继续查找，找到相等位置
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

**如果想共享线程 ThreadLocal 数据怎么办**

使用 InheritableThreadLocal 可以实现多个线程访问 ThreadLocal 的值，我们在主线程中创建一个 InheritableThreadLocal 的实例，然后在子线程中得到这个 InheritableThreadLocal 实例设置的值

```java
public class Mytest {
    public static void main(String[] args){
        final ThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
        threadLocal.set("test");
        Thread thread = new Thread(() -> {
            System.out.println("子线程：" + threadLocal.get());
        });
        thread.start();
    }
}
```

**如何实现传递**

```java
 if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```

如果线程的 inheritThreadLocals 变量不为空，而且父线程的 inheritThreadLocals 也存在，那么就把父线程的 inheritThreadLocals 给当前线程的 inheritThreadLocals

#### 存在问题

**内存泄露**

ThreadLocal 在保存的时候会把自己当作 key 存在 ThreadLocalMap 中，正常情况应该是 key 和 value 都应该被外界强引用才对，但是现在 key 被设计成 WeakReference 弱引用了

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727115823.png" style="zoom: 33%;" />

**弱引用**

>具有弱引用的对象拥有更短暂的生命周期，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象

这就导致一个问题，ThreadLocal 在没有外部强引用时，发生 GC 时会被回收，如果创建 ThreadLocal 的线程一直持续运行，那么这个 Entry 对象中的 value 就有可能一直得不到回收，发生内存泄漏

就比如线程池里的线程，线程都是复用的，那么之前的线程实例处理完后，出于复用的目的，线程依然存活，所以，ThreadLocal 设定的 value 值被持有，导致内存泄漏

按照道理一个线程使用完，ThreadLocalMap 是应该要被清空的，但是现在线程被复用了

#### 如何解决

在代码的最后使用 **remove**

```java
ThreadLocal<String> localName = new ThreadLocal<>();
try {
    localName.set("张三");
    // ...
} finally {
    localName.remove();
}
```

remove 的源码很简单，找到对应的值全部置空，这样在垃圾回收器回收的时候，会自动把它们回收掉

**为什么 ThreadLocalMap 的 key 要设计成弱引用**

key 不设置成弱引用的话就会造成和 entry 中 value 一样内存泄漏的场景

**示例**

<img src="https://raw.githubusercontent.com/whn961227/images/master/data/20200727163649.png" style="zoom: 33%;" />

**假设**

1. Entry 的 key 使用强引用，即 key 对 ThreadLocal 对象使用强引用，也就是图中 5 使用强引用
2. ThreadLocalMap 中不会对过期的 Entry 进行清理

如果 ThreadLocalMap 的 key 使用强引用，那么即使栈内存的 ThreadLocal 引用被清除，但是堆中的 ThreadLocal 对象并不会被清除，因为 ThreadLocalMap 中的 Entry 的 key 对 ThreadLocal 对象是强引用

如果当前线程不结束，那么堆中的 TheadLocal 对象将会一直存在，对应的内存就不会被回收，与之关联的 Entry 不会被回收（Entry 对应的 value 也不会被回收），当这种情况出现数量比较多的时候，未释放的内存就会上升，就可能出现内存泄露的问题

**若 Entry 使用弱引用**

**假设**

1. ThreadLocalMap 中不会对过期的 Entry 进行清理

按照源码，Entry 继承弱引用，其 key 对 ThreadLocal 是弱引用，也就是图中 5 是弱引用，6 是强引用

当栈中的 ThreadLcoal 引用被清除了；由于堆内存中 ThreadLocalMap 的 Entry Key 弱引用 ThreadLocal 对象，此时 ThreadLocal 对象会被 gc 回收，Entry 的 key 为 null，但是 value 不为 null，且 value 也是强引用，所以 Entry 仍旧不能回收，只能释放 ThreadLocal 的内存，仍旧可能导致内存泄露