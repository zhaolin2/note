# IO流

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