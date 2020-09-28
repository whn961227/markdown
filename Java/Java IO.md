## IO

### 分类

#### 按操作方式（类结构）

##### 字节流和字符流

字节流：以字节为单位，每次读入或读出是 8 位数据，可以读任何类型数据

字符流：以字符为单位，每次读入或读出是 16 位数据，只能读取字符类型的数据

##### 输出流和输入流

输出流：从内存读出到文件，只能进行写操作

输入流：从文件读入到内存，只能进行读操作

##### 节点流和处理流

节点流：直接与数据源相连，读入或读出

处理流：与节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流

**为什么要有处理流：** 直接使用节点流，读写不方便，为了更快的读写文件，才有了处理流

<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/raw/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/NIO%E6%A6%82%E8%A7%88.resources/E97A1DBA-0CC4-4679-A081-B164B1645040.jpg" alt="08a43f0086bd0b2f2c6adbe12ba53203" style="zoom: 80%;" />

**字节的输入和输出类结构图**

<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/raw/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/NIO%E6%A6%82%E8%A7%88.resources/D96C7B52-7E5A-44FA-9EB3-6D146ADE7EEF.png" alt="ad1daa76924b325f7f5a5b580c5d5872" style="zoom: 67%;" />

**字符流的输入和输出类结构图**

<img src="https://github.com/wangzhiwubigdata/God-Of-BigData/raw/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/NIO%E6%A6%82%E8%A7%88.resources/CA9A534F-8DEF-448B-A946-3ADE41538F9D.png" alt="952c1fdeadfaeb2ed13a785208e0aea2" style="zoom:67%;" />

#### 按操作对象

![2539ba1fc433a54b14cebfc79019c2ba](https://github.com/wangzhiwubigdata/God-Of-BigData/raw/master/Java%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%A2%9E%E5%BC%BA/NIO%E6%A6%82%E8%A7%88.resources/8F7AD527-634A-4D4E-B31B-6E1FB35BB4EC.jpg)

## NIO

### 核心组件

* Channels
* Buffers
* Selectors

#### Channels 和 Buffers

![Java NIO: Channels and Buffers](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

通道将数据读取到缓冲区中，缓冲区将数据写入到通道中

几种 NIO 主要实现的 Channel 和 Buffer 类型

* FileChannel
* DatagramChanel
* SocketChannel
* ServerSocketChannel

这些通道涵盖了 UDP + TCP 的网络 IO 和文件 IO

NIO 主要实现的 Buffer 类型

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

这些 Buffer 涵盖了可以通过 IO 发送的基本数据类型

#### Selectors

Selector 允许单个线程处理多个 Channel。如果应用程序打开了许多连接（通道），但每个连接的流量很少，这将很方便

![Java NIO: Selectors](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

要使用 Selector，先向 Selector 注册 Channel，然后，然后调用 `select()` 方法，这个方法会一直阻塞直到有一个事件可用于已注册的通道之一。方法返回后，线程即可处理这些事件。

### Channel

Channel 类似于 Stream，但有一些区别：

* 可以读取和写入通道，流通常是单向的（读或写）
* 通道可以异步读写
* 通道始终读取或写入缓冲区

![Java NIO: Channels and Buffers](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

#### Channel 实现类

* FileChannel：从文件中读取或写入数据
* DatagramChannel：通过 UDP 网络读取或写入数据
* SocketChannel：通过 TCP 网络读取或写入数据
* ServerSocketChannel：监听进入的 TCP 连接，例如 Web 服务器，为每个传入的连接创建一个 SocketChannel

#### Channel 实例

```java
RandomAccessFile aFile = new RandomAccessFile("input/words", "rw");
FileChannel inChannel = aFile.getChannel();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf); // 从 Channel 中读入字节到 Buffer 中
while (bytesRead != -1) {
    System.out.println("read " + bytesRead);
    buf.flip(); // 进行读写转换
    while (buf.hasRemaining()) {
        System.out.print((char)buf.get()); // 从 Buffer 中读出字节
    }
    buf.clear();
    bytesRead = inChannel.read(buf);
}
aFile.close();
```

### Buffer

与 NIO Channel 交互时，将使用 NIO Buffer。数据从 Channel 读到 Buffer，从 Buffer 中写入 Channel

Buffer 本质是一个内存块，可以在其中写入数据，然后可以在以后再次读取

#### Buffer 的基本使用

使用一个 Buffer 来读写数据通常遵循以下四个步骤：

1. 将数据写入 Buffer
2. 调用 `buffer.flip()`
3. 从 Buffer 中读取数据
4. 调用 `buffer.clear()`或 `buffer.compact()`

当将数据写入 Buffer 时，Buffer 会跟踪你已写入多少数据，一旦需要读取数据，就需要调用 `flip()`方法调用将 Buffer 从写入模式切换到读取模式。在读取模式下，你可以读取写入 Buffer 的所有数据

读取所有数据后，需要清除 Buffer，使其准备好再次写入。你可以通过两种方式执行此操作：通过调用 `clear()`或者 `compact()`。`clear()`方法清除整个 Buffer，`compact()`仅清除读取的数据。任何未读取的数据都将移至 Buffer 的开头，并且写入 Buffer 的数据将放在未读的数据之后

```java
RandomAccessFile aFile = new RandomAccessFile("input/words", "rw");
FileChannel inChannel = aFile.getChannel();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf); // 从 Channel 中读入字节到 Buffer 中
while (bytesRead != -1) {
    System.out.println("read " + bytesRead);
    buf.flip(); // 进行读写转换
    while (buf.hasRemaining()) {
        System.out.print((char)buf.get()); // 从 Buffer 中读出字节
    }
    buf.clear(); // 清空 Buffer
    bytesRead = inChannel.read(buf);
}
aFile.close();
```

#### Buffer 容量，位置和限制

Buffer 的三个属性：

* capacity
* position
* limit

##### 容量

作为内存存储块，Buffer 具有固定的大小，称之为容量，你只能将 capacity 大小的 byte，long，char 等写入 Buffer。当 Buffer 装满后，需要先清空（读数据或清除数据），然后才能向其中写入更多数据

##### 位置

当数据写入 Buffer 时，你需要在特定位置进行。最初位置为 0 ，当已将 byte，long等写入 Buffer 位置时，该位置将前进以指向 Buffer 中的下一个单元以将数据插入其中。位置可以最大化为 `capacity-1`

##### 限制

在写入模式下，Buffer 限制是可以写入 Buffer 数据量的限制。在写模式下，限制等于 Buffer 的容量

当切换到 Buffer 的读模式，限制意味着可以读取多少数据的限制。因此，当切换 Buffer 到读模式，将限制设置为写模式的写入位置，换句话说，你可以读取与写入的字节一样多的字节数（限制设置为写入的字节数，该字节数由位置标记）

#### Buffer 类型

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

#### 分配 Buffer

```java
// 分配的一个容量为 48 的 ByteBuffer
ByteBuffer buf = ByteBuffer.allocate(48);
// 分配的一个容量为 1024 个字符的 CharBuffer
CharBuffer buf = CharBuffer.allocate(1024);
```

#### 将数据写入 Buffer

可以通过两种方式写入 Buffer

1. 将数据从 Channel 中写入 Buffer
2. Buffer 通过 `put()`方法写入

```java
int bytesRead = inChannel.read(buf); // 读入 Buffer
buf.put(127);
```

#### flip()

`flip()`切换 Buffer 的读写模式，调用 `flip()`会将 position 设置为 0，并将 limit 设置为 position 最后的位置

#### 从 Buffer 读取数据

可以通过两种方式读取 Buffer

1. 从 Buffer 读取数据到 Channel
2. 使用 `get()` 方法从 Buffer 读取数据

```java
int bytesWritten = inChannel.write(buf);
byte aByte = buf.get();
```

#### rewind()

`Buffer.rewind()`设置 position 为 0，因此可以重新读取 Buffer 中的所有数据，limit 保持不变，仍然标记可以从 Buffer 读取的数据量

#### clear() 和 compact()

如果调用 `clear()`，position 将被置为 0，limit 将被置为 capacity。换句话说，Buffer 被清除了，未读取的数据将丢失。

如果想读取未读数据，需要调用 `compact()`

`compact()`将所有未读取的数据复制到 Buffer 开头，然后设置 position 在最后一个未读元素之后，limit 被置为 capacity 的位置。现在 Buffer 可以进行数据写入，但是不会覆盖未读取的数据

#### mark() 和 reset()

通过调用 `Buffer.mark()`标记给定的位置，然后通过 `Buffer.reset()`方法将位置重置回标记的位置

```java
buffer.mark();
buffer.reset();
```

#### equals() 和 compareTo()

可以通过这两个方法比较两个 Buffer

##### equals()

如果满足以下条件，两个 Buffer 相等：

1. 具有相同的类型
2. 具有相同数量
3. 所有剩余的字节，字符等都是相等的

##### compareTo()

用于比较两个 Buffer 的剩余元素，在以下情况下，一个 Buffer 被认为比另一个 Buffer 小：

1. 第一个元素小于另一个 Buffer 的第一个元素
2. 所有元素都相等，但是第一个 Buffer 的元素个数比第二个 Buffer 少

### Scatter / Gather

NIO 内置分散/收集支持

#### Scattering Read

分散读取将数据从单个通道读取到多个缓冲区中

![Java NIO: Scattering Read](http://tutorials.jenkov.com/images/java-nio/scatter.png)

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = {header, body};
inChannel.read(bufferArray);
```

首先将 Buffers 插入数组，然后将数组作为参数传递给 `channel.read()` 方法，该方法按照 Buffer 在数组中出现的顺序从 Channel 中写入数据，一旦 Buffer 已满，Channel 继续填充下一个 Buffer

#### Gathering Write

聚集写入将来自多个 Buffer 的数据写入单个 Channel

![Java NIO: Gathering Write](http://tutorials.jenkov.com/images/java-nio/gather.png)

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = {header, body};
inChannel.write(bufferArray);
```

Buffer 数组被传递到 `write()`方法中，该方法按照 Buffer 在数组中的顺序写入。仅写入 position 和 limit 之间的数据。因此，与分散读取相比，聚集写入可以很好的适应动态大小的消息部分

### Selector

NIO Selector 是一个组件，可以检查一个或多个 Channel 实例，并确定准备好进行读取或写入的通道。这样，单个线程可以管理多个 Channel，从而可以管理多个网络连接

#### 为什么要使用选择器

仅使用单个线程来处理多个通道的优点是需要更少的线程来处理通道，实际上，你可以只使用一个线程来处理所有的 Channel。线程间的切换对于操作系统而言是很大代价的，并且每个线程也占用操作系统的一些资源。因此，使用的线程越少越好

![Java NIO: Selectors](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

#### Creating a Selector

```java
Selector selector = Selector.open();
```

#### Registering Channels with the Selector

```java
// 向 selector 注册 Channel
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

Channel 被 Selector 使用必须是非阻塞模式，这意味着不能将 FileChannel 搭配使用，因为 FileChannel 无法切换到非阻塞模式

注意 `register()`方法的第二个参数，是一个 interest set，意味着你在 Channel 中对什么事件感兴趣，通过 Selector，可以监听四种不同的事件：

1. 连接
2. 接受
3. 读
4. 写

Channel 触发事件，因此，成功连接另一台服务器的 Channel 就是“连接就绪”，接受传入连接的服务器套接字通道就是“接受就绪”，准备读取数据的通道就是“读取就绪”，准备好写入数据的通道就是“写入就绪”

这四个事件由四个 SelectionKey 常量表示：

1. `SelectionKey.OP_CONNECT`
2. `SelectionKey.OP_ACCEPT`
3. `SelectionKey.OP_READ`
4. `SelectionKey.OP_WRITE`

```java
// 如果对多个事件感兴趣
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

