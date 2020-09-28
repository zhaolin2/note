# 整体划分

整个JVM大概可以分为以下几个区域

- 类装载子系统
- 运行时数据区
  - 堆
  - 方法区(元空间)
  - 线程栈
  - 本地方法栈
  - 程序计数器
- 字节码执行引擎

## 对象在内存中的存储布局 jol

普通对象

- markword 8个字节
- class pointer  类型指针  4字节
- 实例数据
- padding   前边不能被8整除 补齐

数组对象

- markword
- class pointer
- 数组长度 4个字节
- 实例数据
- padding

## 对象头

markword

class point 

synchroize 加锁

new ->偏向锁->自旋锁(无锁 lock free 轻量级锁)->重量级锁

年龄就4位 最多15  

 ps+po  15

cms 6

```java


        if(url == null) {
            throw new SQLException("The url cannot be null", "08001");
        }

        println("DriverManager.getConnection(\"" + url + "\")");

        // Walk through the loaded registeredDrivers attempting to make a connection.
        // Remember the first exception that gets raised so we can reraise it.
        SQLException reason = null;

        for(DriverInfo aDriver : registeredDrivers) {
            // If the caller does not have permission to load the driver then
            // skip it.
            if(isDriverAllowed(aDriver.driver, callerCL)) {
                try {
                    println("    trying " + aDriver.driver.getClass().getName());
                    // 调用com.mysql.jdbc.Driver.connect方法获取连接
                    Connection con = aDriver.driver.connect(url, info);
                    if (con != null) {
                        // Success!
                        println("getConnection returning " + aDriver.driver.getClass().getName());
                        return (con);
                    }
                } catch (SQLException ex) {
                    if (reason == null) {
                        reason = ex;
                    }
                }

            } else {
                println("    skipping: " + aDriver.getClass().getName());
            }

        }

        // if we got here nobody could connect.
        if (reason != null)    {
            println("getConnection failed: " + reason);
            throw reason;
        }

        println("getConnection: no suitable driver found for "+ url);
        throw new SQLException("No suitable driver found for "+ url, "08001");
    }
```

## 栈帧

 存储局部变量表、操作数栈、动态链接、方法出口等信息 

 **局部变量表**的容量以变量槽（Variable Slot，下称 Slot）为最小单位 

一般第一个0为this对象

**操作数栈**   而是通过标准的入栈和出栈操作来完成一次数据访问 

 操作数栈   基于栈的执行引擎 

1. iload_0  // push the int in local variable 0 onto the stack 
2. iload_1  // push the int in local variable 1 onto the stack 
3. iadd    // pop two ints, add them, push result 
4. istore_2  // pop int, store into local variable 2 

动态链接  就是指向在运行时常量池中该栈帧所属方法的引用

 符号引用和直接引用在运行时进行**解析和链接的过程**，

 当一个方法被执行后，有两种方式退出这个方法。第一种方式是执行引擎遇到任意一个方法返回的字节码指令，这时候可能会有返回值传递给上层的方法调用者(调 用当前方法的的方法称为调用者)，是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法方式称为正常完成出口(Normal Method Invocation Completion)。 

栈帧：

- 函数的返回地址和参数

- 临时变量: 包括函数的非静态局部变量以及编译器自动生成的其他临时变量

- 函数调用的上下文

  ​	 一个线程中方法的调用链可能会很长，很多方法都同时处于执行状态。对于JVM执行引擎来说，在在活动线程中，只有位于JVM虚拟机栈栈顶的元素才是有效的，即称为**当前栈帧**，与这个栈帧相关连的方法称为**当前方法，**定义这个方法的类叫做**当前类**。 





A答案：FullGC 是老年代内存空间不足的时候，才会触发的，老年代一般是生命周期较长的对象或者大对象，频繁的 FullGC 不会可能会影响程序性能（因为内存回收需要消耗CPU等资源），但是并不会直接导致内存泄漏。

B 答案：JVM奔溃的可能是内存溢出引起的，也可能是其他导致 JVM崩溃的操作，例如设置了错误的JVM参数等。

C 答案：内存异常，最常见的 就是 StackOverFlow 了把，内存溢出，其实内存泄漏的最终结果就是内存溢出。所以，基本上C是对的答案。

## jvm运行时数据区域

### 程序计数器

​	当前线程所执行字节码的行号计数器，分支，循环，跳转，异常处理，线程恢复都要依赖这个计数器

​		为线程私有的内存

### java虚拟机栈

​	描述的是java方法执行的内存模型，每个方法执行的过程，对应一个栈帧在虚拟机内存从入栈到出栈的过程

局部方法表：

### 本地方法栈：

​	是为执行到的Native方法服务的

### Java堆

​	被所有线程共享，在虚拟机启动的时候创建，在此存放对象实例。

survivor中相同年龄大小的总和在survivor的一半以上  那么>=年龄可以进入老年区

大对象和存活时间长(15)的放入老年代 

​		从内存回收的角度来看 分为新生代 老年代

 Eden：Survivor from：Survivor to = 8:1:1 

​			Eden,

​		From Survivor空间，

​		To Survivor 

 **新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2** 

- Class - 由系统类加载器(system class loader)加载的对象，这些类是不能够被回收的，他们可以以静态字段的方式保存持有其它对象。我们需要注意的一点就是，通过用户自定义的类加载器加载的类，除非相应的Java.lang.Class实例以其它的某种（或多种）方式成为roots，否则它们并不是roots，.
- Thread - 活着的线程
- Stack Local - Java方法的local变量或参数
- JNI Local - JNI方法的local变量或参数
- JNI Global - 全局JNI引用
- Monitor Used - 用于同步的监控对象
- Held by JVM - 用于JVM特殊目的由GC保留的对象，但实际上这个与JVM的实现是有关的。可能已知的一些类型是：系统类加载器、一些JVM知道的重要的异常类、一些用于处理异常的预分配对象以及一些自定义的类加载器等。然而，JVM并没有为这些对象提供其它的信息，因此需要去确定哪些是属于"JVM持有"的了。

 Eden区满时会进行一次Minor GC操作，将Eden区进行回收，此时判断存活的对象会被复制进入Survivor from区（年龄加1） 

### 方法区

​	用来存储已经被加载的类的信息，常量，静态变量，即时编译器编译后的代码等数据

​	![1583219067887](D:\note\images\1583219067887.png)

### 运行时常量池

​	Class文件除了有类的版本，字段，方法，接口等描述信息外，还有常量池，用来存放编译器生成的字面量和符号引用

 **运行时常量池**(Runtime Constant Pool)是.class文件中每一个类或接口的常量池表(constant pool table)的运行时表示形式，**属于方法区的一部分**。每一个运行时常量池都在Java虚拟机的方法区中分配，在加载类和接口道虚拟机后，就创建对应的运行时常量池。 

**符号引用(Symbolic References)**则是属于编译原理中的概念，包括了下面三类常量：

1.类和接口的全限定名

2.字段的名称和描述符

3.方法的名称和描述符

Constant pool:
   #1 = Methodref          #6.#24         // java/lang/Object."<init>":()V
   #2 = Methodref          #5.#25         // com/test/ResumeTest.test:(II)V
   #3 = Fieldref           #26.#27        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #28.#29        // java/io/PrintStream.println:(I)V
   #5 = Class              #30            // com/test/ResumeTest
   #6 = Class              #31            // java/lang/Object
   #7 = Class              #32            // java/lang/Cloneable
   #8 = Class              #33            // java/io/Serializable
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               Exceptions
  #16 = Class              #34            // java/lang/InterruptedException
  #17 = Class              #35            // java/lang/ClassNotFoundException
  #18 = Class              #36            // java/lang/CloneNotSupportedException
  #19 = Class              #37            // java/io/IOException
  #20 = Utf8               test
  #21 = Utf8               (II)V
  #22 = Utf8               SourceFile
  #23 = Utf8               ResumeTest.java
  #24 = NameAndType        #9:#10         // "<init>":()V
  #25 = NameAndType        #20:#21        // test:(II)V
  #26 = Class              #38            // java/lang/System
  #27 = NameAndType        #39:#40        // out:Ljava/io/PrintStream;
  #28 = Class              #41            // java/io/PrintStream
  #29 = NameAndType        #42:#43        // println:(I)V
  #30 = Utf8               com/test/ResumeTest
  #31 = Utf8               java/lang/Object
  #32 = Utf8               java/lang/Cloneable
  #33 = Utf8               java/io/Serializable
  #34 = Utf8               java/lang/InterruptedException
  #35 = Utf8               java/lang/ClassNotFoundException
  #36 = Utf8               java/lang/CloneNotSupportedException
  #37 = Utf8               java/io/IOException
  #38 = Utf8               java/lang/System
  #39 = Utf8               out
  #40 = Utf8               Ljava/io/PrintStream;
  #41 = Utf8               java/io/PrintStream
  #42 = Utf8               println
  #43 = Utf8               (I)V

## 基本类型和包装类

 **Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。** 

 两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。** 

# JVM参数

```java
-Xms　　// 设置初始堆内存
-Xmx　　//设置最大堆内存
-Xmn　　//设置年轻代的大小
-XX:NewRatio=n　　//设置年轻代与年老代的比例为“n”
-XX:NewSize=n　　//设置年轻代大小为“n” 
```

# 初始化问题

属于被动引用不会出发子类初始化 

 1.子类引用父类的静态字段，只会触发子类的加载、父类的初始化，不会导致子类初始化 

 2.通过数组定义来引用类，不会触发此类的初始化 

 3.常量在编译阶段会进行常量优化，将常量存入调用类的常量池中， 本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。 



虚拟机规范严格规定了有且只有五种情况必须立即对类进行“初始化”：

1. 使用new关键字实例化对象的时候、读取或设置一个类的静态字段的时候，已经调用一个类的静态方法的时候。

2. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有初始化，则需要先触发其初始化。

3. 当初始化一个类的时候，如果发现其父类没有被初始化就会先初始化它的父类。

4. 当虚拟机启动的时候，用户需要指定一个要执行的主类（就是包含main()方法的那个类），虚拟机会先初始化这个类；

5. 使用Jdk1.7动态语言支持的时候的一些情况。

 

除了这五种之外，其他的所有引用类的方式都不会触发初始化，称为被动引用。下面是被动引用的三个例子：

1. 通过子类引用父类的的静态字段，不会导致子类初始化。

2. 通过数组定义来引用类，不会触发此类的初始化。

3. 常量在编译阶段会存入调用类的常量池中，本质上没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

public class ConstClass { 

  static { 

​    System.out.println("ConstClass init!"); 

  } 

  public static final int value = 123; 

} 

public class NotInitialization{ 

  public static void main(String[] args) { 

​    int x = ConstClass.value; 

  } 

} 

上述代码运行之后，也没有输出“ConstClass init！”，这是因为虽然在Java源码中引用了ConstClass类中的常量HELLOWORLD，但其实在编译阶段通过常量传播优化，已经将此常量的值“hello world”存储到了NotInitialization类的常量池中，以后NotInitialization对常量ConstClass.HELLOWORLD的引用实际都被转化为NotInitialization类对自身常量池的引用了。也就是说，实际上NotInitialization的Class文件之中并没有ConstClass类的符号引用入口，这两个类在编译成Class之后就不存在任何联系了。