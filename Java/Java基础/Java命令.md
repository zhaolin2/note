# Jps

Java Virtual Machine Process Status Tool

java程序在启动之后，会在java.io.tempdir中生成临时文件夹,默认是/tmp/hsperfate_{username}/{pid}

主要命令包括下边的：

- -l 获取完成包名或者jar名
- -v 获取传递给JVM的参数名

问题：

- 因为jps获取线程号主要是通过临时文件夹来获取的，所以只能获取当前用户开启的java线程，完整的java线程还是需要ps -ef
- 假如用户没有/tmp的读写权限的话，可能会获取不到
- 临时文件丢失的时候，则无法使用jps获取进程
- jps只会在默认文件夹下进行读取，如果使用参数-Djava.io.tempdir来进行设置，则无法读取

# Jstack

jstack主要用来定位线程出现长时间停顿的原因，比如死锁，死循环，请求外部资源的长时间等待等。

 ![img](file:///G:\QQ\1548021143\Image\C2C\{5F32EEB8-BFCC-871B-43C0-14F885F418BA}.jpg) 

尝试获取锁(临界区)的都会进入到entry set，状态为“waiting for monitor entry”

waiting set的状态则为“in Object.wait”

使用命令能看到线程得几种状态：

- new 未启动
- runnable 正在运行
- blocked 阻塞并等待监视器锁(也就是synchronized 没有获取到锁 在代码块等待) 
  - wait to lock
- waiting 无限制的等待另一个线程执行特定操作
  - park a wait for address(parking)
  - locked adress(on object monitor)
  - waiting on adress ? 申请对象锁成功 在wait set等待
  - waiting to lock address (申请对象锁未成功 在entry set等待)
- timed_wait 有限的等待一个线程执行特定操作
- terminated 退出的

建议入手点：

- waiting in monty entry 被阻塞的 有问题
- runnable  注意IO线程
- in Object.wait

## 常用参数

- -l 长列表 打印锁的附加信息
- -F 强制打印栈信息

# Jmap

## 堆Dump

堆Dump是反应Java堆的使用情况的内存镜像，包括系统信息，虚拟机属性，完整的线程Dump，所有类和对象的状态等。

## 常用参数

- -heap pid 查看堆使用情况
- -histo 3331  查看堆内存(histogram)中的对象数量及大小
-  **-histo:live**  先gc在统计信息
-  jmap -dump:format=b,file=heapDump pid 把内存的详细情况输出到文件中
  -  `jhat -port 5000 heapDump` 在浏览器中访问：`http://localhost:5000/` 查看详细信息 

# Jstat

jstat始对Java应用程序的资源和性能能进行实时的基于命令行的监控，包括了对Heap Size和垃圾回收状态多的监控，只针对jvm

jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

- option
  - gc  监视堆状况
  - class 监视类状态

https://mp.weixin.qq.com/s/pp0hrFPguptR4Q68wQRzDQ 具体使用

# Jhat



# Javap

javap主要用来查看编译之后的class文件的结构

```java
-help 帮助
-l 输出行和变量的表
-public 只输出public方法和域
-protected 只输出public和protected类和成员
-package 只输出包，public和protected类和成员，这是默认的
-p -private 输出所有类和成员
-s 输出内部类型签名
-c 输出分解后的代码，例如，类中每一个方法内，包含java字节码的指令，
-verbose 输出栈大小，方法参数的个数
-constants 输出静态final常量
```

# 查看gc

jmap -heap 6683 | head -n50

jmap -histo 6683 | head -n20

jmap -dump:format=b,file=heap 6683

