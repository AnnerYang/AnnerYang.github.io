---
layout: post
title: Java核心-理解BIO、NIO、AIO
categories: Java
description: Java核心-理解BIO、NIO、AIO
keywords: BIO, NIO, AIO
---


理解BIO、NIO、AIO的区别，NIO中的核心与使用




# 1 Java中的IO原理

首先Java中的IO都是依赖操作系统内核进行的，我们程序中的IO读写其实调用的是操作系统内核中的read&write两大系统调用。

## 1.1 基础概念

在介绍I/O原理之前，先重温几个基础概念：

- 操作系统与内核

![image-20200504000543303](/images/posts/java/image-20200504000543303.png)

**操作系统**：管理计算机硬件与软件资源的系统软件**内核**：操作系统的核心软件，负责管理系统的进程、内存、设备驱动程序、文件和网络系统等等，为应用程序提供对计算机硬件的安全访问服务

-  内核空间和用户空间

为了避免用户进程直接操作内核，保证内核安全，操作系统将内存寻址空间划分为两部分：**内核空间（Kernel-space）**，供内核程序使用**用户空间（User-space）**，供用户进程使用 为了安全，内核空间和用户空间是隔离的，即使用户的程序崩溃了，内核也不受影响

- 数据流

![image-20200504000823922](/images/posts/java/image-20200504000823922.png)

计算机中的数据是基于随着时间变换高低电压信号传输的，这些数据信号连续不断，有着固定的传输方向，类似水管中水的流动，因此抽象数据流(I/O流)的概念：指一组有顺序的、有起点和终点的字节集合.

抽象出数据流的作用：实现程序逻辑与底层硬件解耦，通过引入数据流作为程序与硬件设备之间的抽象层，面向通用的数据流输入输出接口编程，而不是具体硬件特性，程序和底层硬件可以独立灵活替换和扩展.

![image-20200504000934921](/images/posts/java/image-20200504000934921.png)

## 1.2 内核是如何进行IO交互的

### 1.2.1 磁盘I/O

典型I/O读写磁盘工作原理如下：

![image-20200504001330989](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504001330989.png)

> **tips**: DMA：全称叫直接内存存取（Direct Memory Access），是一种允许外围设备（硬件子系统）直接访问系统主内存的机制。基于 DMA 访问方式，系统主内存与硬件设备的数据传输可以省去CPU 的全程调度.

值得注意的是：

- 读写操作基于系统调用实现
- 读写操作经过用户缓冲区，内核缓冲区，应用进程并不能直接操作磁盘
- 应用进程读操作时需阻塞直到读取到数据

### 1.2.2  网络I/O

这里先以最经典的阻塞式I/O模型介绍：

![image-20200504002100156](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504002100156.png)

![image-20200504170358909](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504170358909.png)

> **tips**:recvfrom，经socket接收数据的函数

值得注意的是：

- 网络I/O读写操作经过用户缓冲区，Sokcet缓冲区
- 服务端线程在从调用recvfrom开始到它返回有数据报准备好这段时间是阻塞的，recvfrom返回成功后，线程开始处理数据报

# 2  Java I/O设计

## 2.1 I/O分类

Java中对数据流进行具体化和实现，关于Java数据流一般关注以下几个点：

- **(1) 流的方向** 从外部到程序，称为输入流；从程序到外部，称为输出流
- **(2) 流的数据单位** 程序以字节作为最小读写数据单元，称为字节流，以字符作为最小读写数据单元，称为字符流
- **(3) 流的功能角色**

![image-20200504171642324](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504171642324.png)

从/向一个特定的IO设备（如磁盘，网络）或者存储对象(如内存数组)读/写数据的流，称为**节点流**； 对一个已有流进行连接和封装，通过封装后的流来实现数据的读/写功能，称为**处理流**(或称为过滤流)；

## 2.2 I/O操作接口

java.io包下有一堆I/O操作类，这些I/O操作类都是在继承4个基本抽象流的基础上，要么是节点流，要么是处理流

### 2.2.1 四个基本抽象流

java.io包中包含了流式I/O所需要的所有类，java.io包中有四个基本抽象流，分别处理字节流和字符流：

- InputStream
- OutputStream
- Reader
- Writer

![image-20200504171904129](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504171904129.png)

### 2.2.2 节点流

![image-20200504171950092](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504171950092.png)

节点流I/O类名由节点流类型 + 抽象流类型组成，常见节点类型有：

- File文件
- Piped 进程内线程通信管道
- ByteArray / CharArray (字节数组 / 字符数组)
- StringBuffer / String (字符串缓冲区 / 字符串)

节点流的创建通常是在构造函数传入数据源，例如：
```JAVA
FileReader reader = new FileReader(new File("file.txt"));
FileWriter writer = new FileWriter(new File("file.txt"));
```

### 2.2.3 处理流

![image-20200504172116897](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504172116897.png)

处理流I/O类名由对已有流封装的功能 + 抽象流类型组成，常见功能有：

- 缓冲：对节点流读写的数据提供了缓冲的功能，数据可以基于缓冲批量读写，提高效率。常见有BufferedInputStream、BufferedOutputStream
- 字节流转换为字符流：由InputStreamReader、OutputStreamWriter实现
- 字节流与基本类型数据相互转换：这里基本数据类型数据如int、long、short，由DataInputStream、DataOutputStream实现
- 字节流与对象实例相互转换：用于实现对象序列化，由ObjectInputStream、ObjectOutputStream实现

处理流的应用了适配器/装饰模式，转换/扩展已有流，处理流的创建通常是在构造函数传入已有的节点流或处理流：

```JAVA
FileOutputStream fileOutputStream = new FileOutputStream("file.txt");
// 扩展提供缓冲写
BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
 // 扩展提供提供基本数据类型写
DataOutputStream out = new DataOutputStream(bufferedOutputStream);
```




# 3 Java BIO

BIO 就是传统的 java.io 包，我们通常所说的 BIO 是相对于 NIO 来说的，BIO 是 BlockingIO 的缩写，它是基于流模型实现的，交互的方式是同步、阻塞方式。在JDK1.4出来之前，我们建立网络连接的时候采用BIO模式，需要先在服务端启动一个ServerSocket，然后在客户端启动Socket来对服务端进行通信，默认情况下服务端需要对每个请求建立一堆线程等待请求，而客户端发送请求后，先咨询服务端是否有线程相应，如果没有则会一直等待或者遭到拒绝请求，如果有的话，客户端会线程会等待请求结束后才继续执行。

## 3.1 标准I/O存在问题

### 3.1.1 数据多次拷贝

- 标准I/O处理，完成一次完整的数据读写，至少需要从底层硬件读到内核空间，再读到用户文件，又从用户空间写入内核空间，再写入底层硬件

- 此外，底层通过write、read等函数进行I/O系统调用时，需要传入数据所在缓冲区起始地址和长度
由于JVM GC的存在，导致对象在堆中的位置往往会发生移动，移动后传入系统函数的地址参数就不是真正的缓冲区地址了

- 可能导致读写出错，为了解决上面的问题，使用标准I/O进行系统调用时，还会额外导致一次数据拷贝：把数据从JVM的堆内拷贝到堆外的连续空间内存(堆外内存)

所以总共经历6次数据拷贝，执行效率较低

![image-20200504175314780](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504175314780.png)

### 3.1.2 操作阻塞

传统的网络I/O处理中，由于请求建立连接(connect)，读取网络I/O数据(read)，发送数据(send)等操作是线程阻塞的，对应抽象到java的socket代码简单示例如下：

```java
public class SocketServer {
  public static void main(String[] args) throws Exception {
    // 监听指定的端口
    int port = 8080;
    ServerSocket server = new ServerSocket(port);
    // server将一直等待连接的到来--阻塞
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    InputStream inputStream = socket.getInputStream();
    byte[] bytes = new byte[1024];
    int len;
    while ((len = inputStream.read(bytes)) != -1) {
      //获取数据进行处理--阻塞
      String message = new String(bytes, 0, len,"UTF-8");
    }
    // socket、server，流关闭操作，省略不表
  }
}
```

以上面服务端程序为例，当请求连接已建立，读取请求消息，服务端调用read方法时，客户端数据可能还没就绪(例如客户端数据还在写入中或者传输中)，线程需要在read方法阻塞等待直到数据就绪。而BIO、NIO、AIO之间的区别就在于这些操作是同步还是异步，阻塞还是非阻塞。所以我们引出同步异步，阻塞与非阻塞的概念。

> 同步与异步
>
>>同步和异步指的是一个执行流程中每个方法是否必须依赖前一个方法完成后才可以继续执行。假设我们的执行流程中：依次是方法一和方法二。
>>
>>同步指的是调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。即方法二一定要等到方法一执行完成后才可以执行。
>>
>>异步指的是调用立刻返回，调用者不必等待方法内的代码执行结束，就可以继续后续的行为。（具体方法内的代码交由另外的线程执行完成后，可能会进行回调）。即执行方法一的时候，直接交给其他线程执行，不由主线程执行，也就不会阻塞主线程，所以方法二不必等到方法一完成即可开始执行。
>>
>>同步与异步关注的是方法的执行方是主线程还是其他线程，主线程的话需要等待方法执行完成，其他线程的话无需等待立刻返回方法调用，主线程可以直接执行接下来的代码。
>>
>>同步与异步是从多个线程之间的协调来实现效率差异。

-----------------------------------------------------------------------------------------------

>阻塞与非阻塞
>>阻塞与非阻塞指的是单个线程内遇到同步等待时，是否在原地不做任何操作。
>>阻塞指的是遇到同步等待后，一直在原地等待同步方法处理完成。
>>非阻塞指的是遇到同步等待，不在原地等待，先去做其他的操作，隔断时间再来观察同步方法是否完成。
>>阻塞与非阻塞关注的是线程是否在原地等待。

### 3.1.3 性能问题

为了实现服务端并发响应，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理，如下图:

![image-20200504180643048](E:\Git\AnnerYang.github.io\images\posts\java\image-20200504180643048.png)

按照上面这种处理方式，虽然可以达到我们的要求，但同时又会带来另外一个问题。由于每创建一个线程，就要为这个线程分配一定的内存空间（也叫工作存储器），而且操作系统本身也对线程的总数有一定的限制。如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，甚至服务器可能会因此而瘫痪。 

# 4 Java NIO

## 4.1 Java NIO概述

NIO也叫Non-Blocking IO 是同步非阻塞的IO模型。线程发起io请求后，立即返回（非阻塞io）。同步指的是必须等待IO缓冲区内的数据就绪，而非阻塞指的是，用户线程不原地等待IO缓冲区，可以先做一些其他操作，但是要定时轮询检查IO缓冲区数据是否就绪。Java中的NIO 是new IO的意思。其实是NIO加上IO多路复用技术。普通的NIO是线程轮询查看一个IO缓冲区是否就绪，而Java中的new IO指的是线程轮询地去查看一堆IO缓冲区中哪些就绪，这是一种IO多路复用的思想。IO多路复用模型中，将检查IO数据是否就绪的任务，交给系统级别的select或epoll模型，由系统进行监控，减轻用户线程负担。

>select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

>epoll的设计和实现与select完全不同。epoll通过在Linux内核中申请一个简易的文件系统(文件系统一般用什么数据结构实现？B+树)。把原先的select/poll调用分成了3个部分：
>>1）调用epoll_create()建立一个epoll对象(在epoll文件系统中为这个句柄对象分配资源)
>>2）调用epoll_ctl向epoll对象中添加这100万个连接的套接字
>>3）调用epoll_wait收集发生的事件的连接


## 4.2 Java  NIO Buffer

Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。  

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

### 4.2.1 Buffer的基本用法

使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
2. 调用flip()方法
3. 从Buffer中读取数据
4. 调用clear()方法或者compact()方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。  

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

下面是一个使用buffer的例子:
```java
package com.company;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;
import java.nio.charset.StandardCharsets;

/**
 * Buffer 和 channel 测试
 */
public class NIOTest {

    public static void main(String[] args) throws IOException {
        // 返回JDK定义的UTF-8字符集 charset
        Charset charset = StandardCharsets.UTF_8;
        // 定义字符集解码器
        CharsetDecoder decoder = charset.newDecoder();

        //创意一个可以随机访问文件流,mode:rw表示可读取和写入
        RandomAccessFile aFile = new RandomAccessFile("E:/TeduProject/testFile/nio-data.txt", "rw");

        //定义一个上面文件关联的唯一 FileChannel 对象
        FileChannel inChannel = aFile.getChannel();

        //创建容量为48字节的缓冲区
        ByteBuffer bBuf = ByteBuffer.allocate(48);
        //创建容量为48字符的缓冲区
        CharBuffer cBuf = CharBuffer.allocate(48);

        int bytesRead = inChannel.read(bBuf);//将字节序列从此文件通道读入给定的缓冲区。
        char[] tmp = null; // 定义临时存放转码后的字符数组
        byte[] remainByte = null;// 定义存放解码后未处理完的字节,decode仅仅转码尽可能多的字节，此次转码不了的字节需要缓存，下次再转
        int leftNum = 0; // 定义未转码的字节数

        while (bytesRead != -1) {

            System.out.println("Read " + bytesRead);

            bBuf.flip();// 切换buffer从写模式到读模式
            decoder.decode(bBuf, cBuf, true); // 以utf8编码转换ByteBuffer到CharBuffer
            cBuf.flip(); // 切换buffer从写模式到读模式

            remainByte = null;
            leftNum = bBuf.limit() - bBuf.position();
            if (leftNum > 0) { // 记录未转换完的字节
                remainByte = new byte[leftNum];
                bBuf.get(remainByte, 0, leftNum);
            }

            // 输出已转换的字符
            tmp = new char[cBuf.length()];
            while (cBuf.hasRemaining()) {
                cBuf.get(tmp);
                System.out.println(new String(tmp));
            }

            bBuf.clear(); // 切换buffer从读模式到写模式
            cBuf.clear(); // 切换buffer从读模式到写模式
            if (remainByte != null) {
                bBuf.put(remainByte); // 将未转换完的字节写入bBuf，与下次读取的byte一起转换
            }

            bytesRead = inChannel.read(bBuf);
        }
        aFile.close();
    }
}
```

### 4.2.2 Buffer的capacity,position和limit

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

为了理解Buffer的工作原理，需要熟悉它的三个属性：

- capacity
- position
- limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在插图后面。

![image-20200505175856481](E:\Git\AnnerYang.github.io\images\posts\java\image-20200505175856481.png)

**capacity**

作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

**position**

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

**limit**

在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position

### 4.2.3 Buffer的类型

Java NIO 有以下Buffer类型

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float 或 double类型来操作缓冲区中的字节。

### 4.2.4 Buffer的分配

要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配48字节capacity的ByteBuffer的例子。

```java
ByteBuffer buf = ByteBuffer.allocate(48);
```

这是分配一个可存储1024个字符的CharBuffer：

```java
CharBuffer buf = CharBuffer.allocate(1024);
```

### 4.2.5 向Buffer中写数据

写数据到Buffer有两种方式：

- 从Channel写到Buffer。
- 通过Buffer的put()方法写到Buffer里。

从Channel写到Buffer的例子:

```java
int bytesRead = inChannel.read(buf); //read into buffer.
```

通过put方法写Buffer的例子：

```java
buf.put(127);
```

put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如， 写到一个指定的位置，或者把一个字节数组写入到Buffer。 更多Buffer实现的细节参考JavaDoc。

### 4.2.6 flip()方法

flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。

换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

### 4.2.7 从Buffer中读取数据

从Buffer中读取数据有两种方式：

1. 从Buffer读取数据到Channel。
2. 使用get()方法从Buffer中读取数据。

从Buffer读取数据到Channel的例子：

```java
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```

使用get()方法从Buffer中读取数据的例子:

```java
byte aByte = buf.get();
```

get方法有很多版本，允许你以不同的方式从Buffer中读取数据。例如，从指定position读取，或者从Buffer中读取数据到字节数组。更多Buffer实现的细节参考JavaDoc。

### 4.2.8 rewind()方法

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

### 4.2.9 clear()与compact()方法

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

### 4.2.10 mark()与reset()方法

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如：

```java
buffer.mark();
//call buffer.get() a couple of times, e.g. during parsing.
buffer.reset();  //set position back to mark.
```

### 4.2.11 equals()与compareTo()方法

可以使用equals()和compareTo()方法两个Buffer。

#### 4.2.11.1 equals()

当满足下列条件时，表示两个Buffer相等：

1. 有相同的类型（byte、char、int等）。
2. Buffer中剩余的byte、char等的个数相等。
3. Buffer中所有剩余的byte、char等都相同。

如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。

#### 4.2.11.2 compareTo()方法

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：

1. 第一个不相等的元素小于另一个Buffer中对应的元素 。
2. 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

## 4.3 Java  NIO Channel

### 4.3.1 Channel是什么

Java NIO的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

正如上面所说，从通道读取数据到缓冲区，从缓冲区写入数据到通道。如下图所示：

![image-20200505004256954](E:\Git\AnnerYang.github.io\images\posts\java\image-20200505004256954.png)

下面这些是Java NIO中最重要的通道的实现：

- FileChannel			
    - 从文件中读写数据
- DatagramChannel		
	- 能通过UDP读写网络中的数据
- SocketChannel			
	- 能通过TCP读写网络中的数据
- ServerSocketChannel
	- 可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel

### 4.3.2 FileChannel的实现
```java
package com.company;

import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;
import java.nio.charset.StandardCharsets;

public class NIOChannelTest {

    public static void main(String[] args) throws IOException {
        // 文件编码是utf8,需要用utf8解码
        Charset charset = StandardCharsets.UTF_8;
        CharsetDecoder decoder = charset.newDecoder();

        RandomAccessFile aFile = new RandomAccessFile("E:/TeduProject/testFile/nio-data.txt", "rw");
        FileChannel inChannel = aFile.getChannel();

        ByteBuffer bBuf = ByteBuffer.allocate(48);//缓存大小设置为48个字节
        CharBuffer cBuf = CharBuffer.allocate(48);

        int bytesRead = inChannel.read(bBuf);//从文件通道读取字节到buffer
        char[] tmp = null; // 临时存放转码后的字符
        byte[] remainByte = null;// 存放decode操作后未处理完的字节。decode仅仅转码尽可能多的字节，此次转码不了的字节需要缓存，下次再转
        int leftNum = 0; // 未转码的字节数

        while (bytesRead != -1) {

            System.out.println("Read " + bytesRead);

            bBuf.flip();// 切换buffer从写模式到读模式
            decoder.decode(bBuf, cBuf, true); // 以utf8编码转换ByteBuffer到CharBuffer
            cBuf.flip(); // 切换buffer从写模式到读模式

            remainByte = null;
            leftNum = bBuf.limit() - bBuf.position();
            if (leftNum > 0) { // 记录未转换完的字节
                remainByte = new byte[leftNum];
                bBuf.get(remainByte, 0, leftNum);
            }

            // 输出已转换的字符
            tmp = new char[cBuf.length()];
            while (cBuf.hasRemaining()) {
                cBuf.get(tmp);
                System.out.println(new String(tmp));
            }

            bBuf.clear(); // 切换buffer从读模式到写模式
            cBuf.clear(); // 切换buffer从读模式到写模式
            if (remainByte != null) {
                bBuf.put(remainByte); // 将未转换完的字节写入bBuf，与下次读取的byte一起转换
            }

            bytesRead = inChannel.read(bBuf);
        }
        aFile.close();
    }
}
```

注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据。下一节会深入讲解Buffer的更多细节。











参考: 

- [如何理解BIO、NIO、AIO的区别?][1]
- [Java I/O体系从原理到应用，这一篇全说清楚了][2]
- [Java NIO 系列教程][3]

[1]:https://juejin.im/post/5dbba5df6fb9a0204a08ae55#heading-1
[2]:https://juejin.im/post/5dcbefb45188250d194507b7#heading-12
[3]:http://ifeve.com/java-nio-all/