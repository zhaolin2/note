## 内存结构

### 堆

分配实例和数组都在这分配，也被叫做GC堆

进一步可以细分为Eden，s0，s1，tentired    一次gc+1，从eden到s区，从s到tentired

### 方法区

用来存储已经被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码

也叫永久代和元空间，方法区是虚拟机规范是规定的(类似接口),永久代是1.8之前的实现，在堆中，有大小的限制，元空间是使用直接内存，只受到本机可用内存的限制。

### 虚拟机栈

虚拟机栈是一个个的栈帧构成的，每个栈帧又包括局部变量表，操作数栈，动态链接，方法出口信息

局部变量表主要存放了各种已知的数据类型(基础数据类型)，对象引用

方法出口，包括正常返回和抛出异常，每一个都会导致栈帧被弹出。

### 本地方法栈

执行native方法，本地方法被执行的时候，在本地方法栈中也会创建一个栈帧，包括局部变量表，操作数栈，动态链接，方法出口等信息。

### 程序计数器

当前线程所执行字节码的行号指示器

- 可以依次执行指令，来实现代码的流程控制，顺序执行，选择，循环，异常处理
- 在多线程情况下，在切换回当前线程的时候，可以直到上次线程运行到哪了

## 常量池

### Class文件中的常量池(静态常量池)

- 字面量  文本字符串
- 符号引用
  - 类和接口的全限定名
  - 字段的名称和描述符
  - 方法的名称和描述符

### 运行时常量池

方法区的一部分，在加载类的时候，会把类的版本，字段，方法，接口等信息，字面量，符号引用，都进入**方法区**的运行时常量池。

### 字符串常量池

只存储引用，不存储内容

**只有“”的，才会存储进常量池**

### String.intern

- **如果常量池中存在当前字符串，那么直接返回常量池中它的引用**。
- **如果常量池中没有此字符串, 会将此字符串引用保存到常量池中后, 再直接返回该字符串的引用**！



```java
		//并没有存入hello
String s1=new String("he")+new String("llo"); 
        //将 堆中新建的对象"hello" 存入字符串常量池 返回当前字符串的引用 也就是s1
        s1.intern();
        //因为直接就有 所以返回的其实是s1
        String s2="hello";
        //输出是true
        System.out.println(s1==s2);

		//其实已经在常量池中创建了一个常量1 和一个堆中的对象s  这俩不一样
		  String s = new String("1");
        s.intern();
		//返回的是常量池中的对象
        String s2 = "1";
        System.out.println(s == s2);
```



### 基本类型

 java中基本类型的包装类的大部分都实现了常量池技术,即除了两种**浮点数类型**外的其余六种：Character,Byte,Short,Integer,Long,Boolean.

但是需要注意，除了Boolean之外的五种封装类只有在**[-128,127]**范围内才在常量池内有对象。 

