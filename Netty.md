# IO 模型

读写一个socket会返回相应的文件描述符，描述符就是一个数字，指向内核的一个结构体(文件路径、数据区等)。

1. 阻塞与非阻塞：用于描述调用者在等待返回结果时的状态。
* 阻塞：调用者发起请求后，会一直等待返回结果，这期间当前线程会被挂起（阻塞）。
* 非阻塞：调用者发起请求后，会立刻返回，当前线程也不会阻塞。该调用不会立刻得到结果，调用者需要定时轮询查看处理状态。

2. 同步与异步：用于描述调用结果的返回机制（或者叫通信机制）。
* 同步：调用者发起请求后，会一直等待返回结果，即由调用者主动等待这个调用结果。
* 异步：调用者发起请求后，会立刻返回，但不会立刻得到这个结果，而是由被调者在执行结束后主动通知（如 Callback）调用者。

五种 IO 模型：

1. 阻塞式 IO 模型：默认情况下，所有文件操作都是阻塞的，应用进程从发起 IO 系统调用，至内核返回，这整个期间是处于阻塞状态的，可能数据包被正确复制到应用进程缓存区成功返回，也可能发生错误返回。

  ![233](assets/233.png)

2. 非阻塞式 IO 模型：应用进程可以将 Socket 设置为非阻塞，在发起 IO 系统调用后，如果缓冲区没有数据，就直接返回 EWOULDBLOCK 错误标识，否则返回成功标识，应用进程需要轮询调用 recvfrom 系统调用。

  ![234](assets/234.png)

3. IO 多路复用模型：Linux 提供 select/poll 系统调用，进程通过将多个socket 的多个 fd 传递给 select/poll 调用，调用 select/poll 的进程会阻塞等待 select/poll 顺序扫描 fd 是否就绪，当就绪时，立即返回回调函数，再调用 recvfrom 系统调用完成数据读取。出于效率的考量，单进程打开的fd 受到限制，默认 1024 。另外 linux 还提供 epoll 系统调用使用事件驱动代替顺序扫描，性能更高。

  epoll 优势：

  * 支持一个进程打开的 socket描述符 不受限制。
  * IO效率不会随fd 数目增加而线性下降。
  * eppol 使用 mmap 加速内核和用户空间的消息传递：内核需要报 fd 消息通知到用户空间。

  ![235](assets/235.png)

4. 信号驱动 IO 模型： 可以为 Socket 开启信号驱动 IO 功能，应用进程需向内核注册一个信号处理程序（通过sigaction 系统调用），该操作并立即返回。当内核中有数据准备好，会发送一个信号给应用进程，应用进程便可以在信号处理程序中发起 IO 系统调用，来完成数据读取。主要用于UDP套接字，TCP信号产生过于频繁，并且信号没有告诉发送了什么事情。

  ![236](assets/236.png)

5. 异步 IO 模型：应用进程发起 IO 系统调用后，会立即返回。当内核中数据完全准备后，并且也复制到了用户空间，会产生一个信号来通知应用进程。

  ![237](assets/237.png)

应用进程对内核发起 IO 系统调用后，内核会经过两个阶段来完成数据的传输：
* 第一阶段：等待数据。即应用进程发起 IO 系统调用后，会一直等待数据；当有数据传入服务器，会将数据放入内核空间，此时数据准备好。
* 第二阶段：将数据从内核空间复制到用户空间，并返回给应用程序成功标识。

  ![238](assets/238.png)

# JAVA NIO

传统IO基于字节流和字符流操作，NIO 基于 Channel 和 Buffer 操作，数据总是从 Channel 读取到 Buffer 中，或者从  Buffer 写入到 Channel 中。

## Channel

Channel 是 IO操作 的载体，全双工的，即可以读取也可以写入，流的读写只能是单向的，例如 InputStream 、 OutputStream 。

主要分为两类：

* FileChannel
* SocketChannel、ServerSocketChannel、DatagramChannel

都继承于 SeekableByteChannel（负责在任意位置进行读写），GatheringByteChannel （负责顺序将多个缓存区写入Channel，即一个缓冲区数组），ScatteringByteChannel （负责从Channel 读取数据依次填满多个缓存区，即一个缓冲区数组），例如

```java
// SeekableByteChannel

Path filePath = Paths.get("D://error.log");
ByteBuffer buffer = ByteBuffer.allocate(1024);
buffer.put("111111".getBytes());
buffer.flip();
try {
    FileChannel channel = FileChannel.open(filePath, StandardOpenOption.WRITE);
    channel.position(80);    // 覆盖此处的内容为111111
    channel.write(buffer);
    channel.close();

    buffer = ByteBuffer.allocate(1024);
    channel = FileChannel.open(filePath, StandardOpenOption.READ);
    channel.position(100);    // 读取此处的内容
    channel.read(buffer, 10);
    buffer.flip();
    System.out.println(buffer);

    Charset charset = Charset.forName("utf-8");
    CharsetDecoder decoder = charset.newDecoder();
    CharBuffer charBuffer = decoder.decode(buffer);
    System.out.println(charBuffer);
} catch (IOException e) {
    e.printStackTrace();
}
```

```java
// ScatteringByteChannel

ByteBuffer header = ByteBuffer.allocateDirect (10);
ByteBuffer body = ByteBuffer.allocateDirect (80);
ByteBuffer [] buffers = { header, body };
int bytesRead = channel.read (buffers);
```

```java
// GatheringByteChannel

ByteBuffer header = ByteBuffer.allocateDirect (10);
ByteBuffer body = ByteBuffer.allocateDirect (80);
ByteBuffer [] buffers = { header, body };
channel.write(bufferArray);
```

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

### FileChannel

创建：

1. FileChannel#open ：

  ```java
  public static FileChannel open(Path path, OpenOption... options) throws IOException

  public static FileChannel open(Path path, Set<? extends OpenOption> options, FileAttribute<?>... attrs) throws IOException
  ```

2. 通过FileInputStream/FileOutputStream 获取

  ```java
  FileInputStream inputStream = new FileInputStream("D:/test.txt");
  FileChannel channel = inputStream.getChannel();

  FileOutputStream outputStream = new FileOutputStream("D:/test.txt");
  FileChannel channel1 = outputStream.getChannel();
  ```

3. 通过RandomAccessFile获取

  ```java
  RandomAccessFile randomAccessFile = new RandomAccessFile("./test.txt", "rw");
  FileChannel channel2 = randomAccessFile.getChannel();
  ```

api：

| 接口                    | 描述                                                   |
| ----------------------- | ------------------------------------------------------ |
| open                    | 创建FileChannel                                        |
| read/write              | 基于FileChannel读写                                    |
| force                   | 强制将FileChannel中的数据刷入文件中                    |
| map                     | 使用 mmap 技术直接映射内核内存到用户内存，返回  MappedByteBuffer |
| transferTo/transferFrom | 将通道中的字节传输到其他通道                           |
| lock/tryLock            | 获取文件锁                                             |

read/write：

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(16);
// 如果返回-1，表示已经读到文件尾，无内容可读。
int count = channel2.read(byteBuffer);

ByteBuffer byteBuffer = ByteBuffer.allocate(16);
byte[] bs = "s".getBytes();
byteBuffer.put(bs);
byteBuffer.flip();
channel2.write(byteBuffer);
```

force：强制将 Channel 中的内容写入文件，参数false 表示将文件内容写入文件，true 表示将 文件内容 和 元数据（比如权限信息，文件修改时间）一起写入到文件，可能需要至少一次IO操作，取决于底层操作系统。不能保证对 MappedByteBuffer 的修改也写入，可以调用 MappedByteBuffer#force 来保证。

map：使用 mmap 技术直接映射内核内存，返回  MappedByteBuffer。

* READ_ONLY：以只读的方式映射，如果发生修改，则抛出ReadOnlyBufferException
* READ_WRITE：读写方式
* PRIVATE：对这个MappedByteBuffer的修改不写入文件，且其他程序是不可见的。

一旦经过map映射后，将于用于映射的FileChannel没有联系，即使Channel关闭，也对MappedByteBuffer没有影响。

```java
Path path = FileSystems.getDefault().getPath("./test.txt");
FileChannel channel = FileChannel.open(path, StandardOpenOption.READ, StandardOpenOption.WRITE);
MappedByteBuffer mappedByteBuffer = channel.map(MapMode.READ_ONLY, 0, 100000);
byte[] bs = new byte[100];
mappedByteBuffer.get(bs);
```

transferTo/transferFrom：底层调用 sendfile系统调用(windows 不支持) 或者使用 mmap技术写入，实现零拷贝。

```java
// FileChannel
public long transferTo(long position, long count,
                        WritableByteChannel target)
     throws IOException
 {
     //omit..
     long n;
     //尝试不同方式
     // Attempt a direct transfer, if the kernel supports it 使用 sendfile
     if ((n = transferToDirectly(position, icount, target)) >= 0)
         return n;
     // Attempt a mapped transfer, but only to trusted channel types 使用mmap
     if ((n = transferToTrustedChannel(position, icount, target)) >= 0)
         return n;
     // Slow path for untrusted targets
     return transferToArbitraryChannel(position, icount, target);
 }
```

```java
srcChannel.transferTo(0, Integer.MAX_VALUE, dstChannel);
srcChannel.transferFrom(fromChannel, 0, Integer.MAX_VALUE);
```

lock/tryLock：
* 被JVM持有，进程级别，不可用于多线程安全控制同步工具。如果统一进程内，线程1获取了文件锁FileLock（共享或者独占），线程2再来请求获取该文件的文件锁，则会抛出OverlappingFileLockException 。
* 一个程序获取到FileLock后，是否会阻止另一个程序访问相同文件具重叠内容的部分取决于操作系统的实现，具有不确定性。FileLock的实现依赖于底层操作系统实现的本地文件锁设施。
* ![239](assets/239.png)

### SocketChannel & ServerSocketChannel & DatagramChannel

SocketOption：配置 Socket 连接，定义在 StandardSocketOptions 中。

* SO_BROADCAST：允许使用IPv4的广播地址发送数据报，默认禁用
* SO_KEEPALIVE：在连接空闲时操作系统定期探测连接的另一端，一般时空闲2小时后，发送第一个探测分组，如果没收到回应每隔75秒发送一个探测分组，最多重复发送9次。
* SO_SNDBUF：套接字发送缓冲区的大小，单位是字节，如果是使用UDP，发送缓冲区可能会限制UDP的发送的数据报大小。
* SO_RCVBUF：套接字接收缓冲区的大小，单位是字节，如果是使用UDP，接收缓冲区可能会限制UDP的接收的数据报大小。
* TCP_NODELAY：禁用Nagle算法，当我们只要发送1字节的数据，却需要40字节的TCP/IP头部时，浪费会非常大，Nagle算法是通过合并短段并提高网络效率。

Nagle算法的规则：
* 如果包长度达到MSS(Maximum Segment Size 1460字节)，则允许发送
* 如果该包含有FIN，则允许发送
* 设置了TCP_NODELAY选项，则允许发送
* 未设置TCP_CORK选项时，若所有发出去的小数据包（包长度小于MSS）均被确认，则允许发送；
* 上述条件都未满足，但发生了超时（一般为200ms），则立即发送

## Buffer

Buffer 本质是一块可以写入数据可以读取数据的内存。
