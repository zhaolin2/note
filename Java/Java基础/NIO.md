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

## IO模型

- 阻塞I/O
- 非阻塞I/O
- I/O多路复用
- 信号驱动I/O
- 异步I/O

## 阻塞I/O

 在进程(用户)空间中调用`recvfrom`，其系统调用直到数据包到达且**被复制到应用进程的缓冲区中或者发生错误时才返回**，在此期间**一直等待**。 

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQxJcyco4G7ibEK6n9QXJTbB8ib5AFteaCYJkKTOksDzcPypY1wJAm2oNoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 非阻塞I/O

 `recvfrom`从应用层到内核的时候，如果没有数据就**直接返回**一个EWOULDBLOCK错误，一般都对非阻塞I/O模型**进行轮询检查这个状态**，看内核是不是有数据到来。 

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQxX1ic8anBfCswIhRXVHP6cvJ2IicKfaYicoyGTianx8nOpcHBfS0AaRfBCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## I/O复用模型

在Linux下它是这样子实现I/O复用模型的：

- 调用`select/poll/epoll/pselect`其中一个函数，**传入多个文件描述符**，如果有一个文件描述符**就绪，则返回**，否则阻塞直到超时。

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQxEQiaOqKnvic0u2jNcSMoI7uGPD1wRXhzIKGvQZqPaZLBlnO5FuAPY2Lg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 1）当用户进程调用了select，那么整个进程会被block；
- （2）而同时，kernel会“监视”所有select负责的socket；
- （3）当任何一个socket中的数据准备好了，select就会返回；
- （4）这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程(空间)。
- 所以，I/O 多路复用的特点是**通过一种机制一个进程能同时等待多个文件描述符**，而这些文件描述符**其中的任意一个进入读就绪状态**，select()函数**就可以返回**。

select/epoll的优势并不是对于单个连接能处理得更快，而是**在于能处理更多的连接**。