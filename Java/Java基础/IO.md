# IO

Bit是最小的二进制单位，计算的操作部分，为0或者1；

Byte是计算机操作数据的最小单位，8bit组成 -128-127

char是用户所读写的最小单位，在java中由16位bit组成

## IO分类

具体使用见JavaSEDemo中的io包

- 字节读取 InputStream
  - 节点流  FileInPutStream
  - ByteArrInputStream
  - PipInputStream
  - 处理流  BufferedInputStream
  - DataInputStream
  - ObjectInputStream
  - SequenceInputStream

- 字节写出   跟字节读取相对应

- 字符读取 Reader
  - 节点流 FileReader
  - CharArrayReader
  - PipedReader
  - 处理流  BufferedReader
  - InputStreamReader

## 按操作方式（类结构）

- **字节流和字符流：**

- - 字节流：以字节为单位，每次次读入或读出是8位数据。可以读任何类型数据。
  - 字符流：以字符为单位，每次次读入或读出是16位数据。其只能读取字符类型数据。

- **输出流和输入流：**

- - 输出流：从内存读出到文件。只能进行写操作。
  - 输入流：从文件读入到内存。只能进行读操作。

```text
注意：这里的出和入，都是相对于系统内存而言的。
Input用于读
Output用于写
OutputStreamWriter  字符流->字节流
InputStreamReader 字节流->字符流
```

- **节点流和处理流：**

- - 节点流：直接与数据源相连，读入或读出。
  - 处理流：与节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流。

```text
为什么要有处理流？直接使用节点流，读写不方便，为了更快的读写文件，才有了处理流。
```

![](https://pic3.zhimg.com/80/v2-6a68758ec960e05fd07ae9438ea1b832_720w.png)

## 分类说明

- **1. 输入字节流InputStream**：
  输入字节流的继承图可见上图，可以看出：

- - FileInputStream： 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。
  - ByteArrayInputStream：从Byte数组中读取流
  - PipedInputStream： 是从与其它线程共用的管道中读取数据。PipedInputStream的一个实例要和PipedOutputStream的一个实例共同使用，共同完成管道的读取写入操作。主要用于线程操作。
  - ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）

- **2. 输出字节流OutputStream：
  **输出字节流的继承图可见上图，可以看出：

- - FIleOutputStream：是两种基本的介质流  本地文件中写入数据
  - ByteArrayOutputStream： 是两种基本的介质流，它们分别向Byte 数组。
  - PipedOutputStream：是向与其它线程共用的管道中写入数据。
  - ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流。

- **3. 字符输入流Reader：
  **在上面的继承关系图中可以看出：

- - FileReader：
  - PipedReader：是从与其它线程共用的管道中读取数据
  - CharArrayReader：
  - CharReader、StringReader 是两种基本的介质流，它们分别将Char 数组、String中读取数据。
  - BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。
  - FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。
  - InputStreamReader： 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader 可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。我们可以从这个类中得到一定的技巧。Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致。后面会有Reader 与InputStream 的对应关系。



- **4. 字符输出流Writer：
  **在上面的关系图中可以看出：

- - FileWriter:
  - PipedWriter:是向与其它线程共用的管道中写入数据
  - CharArrayWriter:
  - CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。
  - BufferedWriter 是一个装饰器，为Writer 提供缓冲功能。
  - PrintWriter 和PrintStream 极其类似，功能和使用也非常相似。
  - OutputStreamWriter： 是OutputStream 到Writer 转换的桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。功能和使用和OutputStream 极其类似，后面会有它们的对应图。

## 同步 阻塞

同步是对于被调用者来说的。

假如A->B

- 同步就是B接收到A的调用之后，立刻执行，返回结果
- 异步是指不会保证立马执行，但是保证会做，B在执行完成之后会通知A。

阻塞和非阻塞是描述调用方的。

比如A->B

- 阻塞是指等到结果的返回，一直等待，等着B返回结果。
- 非阻塞是指在发出调用之后不需要等待，去做自己的事情。

# 五种IO模型

## 阻塞IO

 当用户线程发出IO请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU。

当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。 

```java
data = socket.read();
```

## 非阻塞IO

 当用户线程发起一个read操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。 

 所以事实上，在非阻塞IO模型中，用户线程需要不断地询问内核数据是否就绪，也就说非阻塞IO不会交出CPU，而会一直占用CPU。 

```java
while(true){
    data = socket.read();
    if(data!= error){
        处理数据
        break;
    }
}

```

## IO复用

 在多路复用IO模型中，会有一个线程不断去轮询多个socket的状态，只有当socket真正有读写事件时，才真正调用实际的IO读写操作。因为在多路复用IO模型中，只需要使用一个线程就可以管理多个socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。 

## 信号驱动IO

 在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。 

## 异步IO

 异步IO模型是比较理想的IO模型，在异步IO模型中，当用户线程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从内核的角度，当它受到一个asynchronous read之后，它会立刻返回，说明read请求已经成功发起了，因此不会对用户线程产生任何block。然后，内核会等待数据准备完成，然后将数据拷贝到用户线程，当这一切都完成之后，内核会给用户线程发送一个信号，告诉它read操作完成了。也就说用户线程完全不需要实际的整个IO操作是如何进行的，只需要先发起一个请求，当接收内核返回的成功信号时表示IO操作已经完成，可以直接去使用数据了。 

# 零拷贝



# NIO

主要包括三个组成

- Bufffer       数据
- Channel管道    用来运送数据
- Selector选择器

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQx2PBicyCHBys3VYmichRh8SKX7DwOqs9UnhTbIfTT0a0e215shnr6m6uA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 直接缓冲区与非直接缓冲区

- 非直接缓冲区是**需要**经过一个：copy的阶段的(从内核空间copy到用户空间)
- 直接缓冲区**不需要**经过copy阶段，也可以理解成--->**内存映射文件**，(上面的图片也有过例子)。



使用直接缓冲区有两种方式：

- 缓冲区创建的时候分配的是直接缓冲区
- 在FileChannel上调用`map()`方法，将文件直接映射到内存中创建



这是非直接缓冲区

当我们的程序想要从硬盘中读取数据 需要

1.先从物理硬盘把数据读取到物理内存中

2再将内容复制到JVM的内存中

3然后读取应用程序才可以读取到内容



 直接缓冲区的是图中红线所标识的 直接在应用程序和物理磁盘中直接在内存中建立一个缓冲区在物理内存中,这样省略了复制的步骤 效率由此提高. 

![](https://img-blog.csdn.net/20180322202832624)