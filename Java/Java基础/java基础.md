# java基础类型

八种基本类型：byte  ,boolean,  char  ,short,  int long   float   double

​	 java中基本类型的**包装类**的大部分都实现了常量池技术，即Byte,Short,Integer,Long,Character,Boolean。这5种包装类默认创建了数值[-128，127]的相应类型的缓存数据，但是超出此范围仍然会去创建新的对象。 

基本类型都是在栈上分配，而引用类型会在堆上来进行分配，所以尽量局部变量使用基础类型

# 基础类型所需内存

| Primitive Type | Memory Required(bytes)   32位操作系统        |
| :------------- | -------------------------------------------- |
| boolean        | 1                                            |
| byte           | 1                                            |
| char           | 2                                            |
| short          | 2                                            |
| int            | 4                                            |
| float          | 4                                            |
| long           | 8                  4                         |
| double         | 8                                            |
| reference      | 8 (开启指针压缩之后 4个)                   4 |

# 值传递

参数传递对于基础来说，就是值传递 每次传入一个对应值

对于引用类型来说，是句柄(直接指针)传递，所以当句柄重新赋值的时候，那么就跟原来的对象没有关系了。

值传递：把实际参数复制一份传入到函数中，函数中的参数改变，不会影响到实际参数

引用传递：调用参数的时候，直接把实际参数的地址传入进去，那么在函数中的改变会影响到实际参数。

## 求值策略

 在计算机科学中，求值策略是确定编程语言中表达式的求值的一组（通常确定性的）规则。求值策略定义何时和以何种顺序求值给函数的实际参数、什么时候把它们代换入函数、和代换以何种形式发生。 

- 值传递 实际参数先求值，然后复制值传递给函数
- 引用传递 直接把引用传递给函数
- 共享对象传递 先获取到实参的地址，复制地址拷贝给函数

官方文档指出：java把对象的引用当作值传递给方法(就是共享对象传递)

# String 

String本身为final，使用char[] 来保存

在Java9之后，使用byte[]来存储，使用coder表示使用的哪种编码

## 不可变的好处

- 缓存hash值
- String Pool
- 安全性  防止内存泄漏
- 线程安全

## 可变String

- StringBuilder
- StringBuffer 线程安全

# 基本操作符

操作符优先级

括号  > ++ > && > 赋值

![](D:\note\images\yunsuanfu.png)

位运算符

 a & b = 0000 1100(与，都为1为true) 

a|b = 0011 1101（或，有1就是true） 

a ^ b = 0011 0001（异或，不同就是true） 

~a  = 1100 0011（取反） 

<< 表示左移位

\>> 表示带符号右移位

\>>> 表示无符号右移

# java标识符

- Java 标识符有如下命名规则： 
  - 由26个英文字母大小写，数字：0-9 符号：_ $ 组成
  - 标识符应以字母、_ 、$开头。
  - 标识符不能是关键字。
- Java中严格区分大小写 

# 修饰符

​	对于外部类来说，只有两种修饰，public和默认（default），因为外部类放在包中，只有两种可能，包可见和包不可见。

​	对于内部类来说，可以有所有的修饰，因为内部类放在外部类中，与成员变量的地位一致，所以有四种可能。

    		作用域       当前类    同一package   子孙类     其他package       
          public        √         √             √           √ 
    
          protected     √          √             √           × 
    
          friendly      √          √             ×           × 
    
          private       √          ×             ×           ×

# 基本包装对象

**PO 是** Persistant Object 的缩写，用于表示数据库中的一条记录映射成的 java 对象。PO 仅仅用于表示数据，没有任何数据操作。通常遵守 Java Bean 的规范，拥有 getter/setter 方法。

**DAO 是** Data Access Object 的缩写，用于表示一个数据访问对象。使用 DAO 访问数据库，包括插入、更新、删除、查询等操作，与 PO 一起使用。DAO 一般在持久层，完全封装数据库操作，对外暴露的方法使得上层应用不需要关注数据库相关的任何信息。

**VO 是** Value Object 的缩写，用于表示一个与前端进行交互的 java 对象。有的朋友也许有疑问，这里可不可以使用 PO 传递数据？实际上，这里的 VO 只包含前端需要展示的数据即可，对于前端不需要的数据，比如数据创建和修改的时间等字段，出于减少传输数据量大小和保护数据库结构不外泄的目的，不应该在 VO 中体现出来。通常遵守 Java Bean 的规范，拥有 getter/setter 方法。

**DTO 是** Data Transfer Object 的缩写，用于表示一个数据传输对象。DTO 通常用于不同服务或服务不同分层之间的数据传输。DTO 与 VO 概念相似，并且通常情况下字段也基本一致。但 DTO 与 VO 又有一些不同，这个不同主要是设计理念上的，比如 API 服务需要使用的 DTO 就可能与 VO 存在差异。通常遵守 Java Bean 的规范，拥有 getter/setter 方法。

**BO 是** Business Object 的缩写，用于表示一个业务对象。BO 包括了业务逻辑，常常封装了对 DAO、RPC 等的调用，可以进行 PO 与 VO/DTO 之间的转换。BO 通常位于业务层，要区别于直接对外提供服务的服务层：BO 提供了基本业务单元的基本业务操作，在设计上属于被服务层业务流程调用的对象，一个业务流程可能需要调用多个 BO 来完成。

**POJO 是** Plain Ordinary Java Object 的缩写，表示一个简单 java 对象。上面说的 PO、VO、DTO 都是典型的 POJO。而 DAO、BO 一般都不是 POJO，只提供一些调用方法。



##  **实例**

以一个实例来探讨下 POJO 的使用。假设我们有一个面试系统，数据库中存储了很多面试题，通过 web 和 API 提供服务。可能会做如下的设计：

数据表：表中的面试题包括编号、题目、选项、答案、创建时间、修改时间；

PO：包括题目、选项、答案、创建时间、修改时间；

VO：题目、选项、答案、上一题URL、下一题URL；

DTO：编号、题目、选项、答案、上一题编号、下一题编号；

DAO：数据库增删改查方法；

BO：业务基本操作。

## equals

 不能使用一个值为null的引用类型变量来调用非静态方法，否则会抛出异常 

```java
Objects#equals

public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```

浮点数使用BigDecimal来进行比较

 `a.compareTo(b)` : 返回 -1 表示小于，0 表示 等于， 1表示 大于。 

数组转换成List

 List list = new ArrayList<>(Arrays.asList("a", "b", "c")) 

#  代码执行顺序

首先加载类的时候 会执行**静态代码块** 只执行一次 先执行父类 在执行子类

进行new的时候 首先调用父类的**构造块** 然后**构造方法** 然后才是子类的构造块 构造方法

初始化过程是这样的： 

1.首先，初始化父类中的静态成员变量和静态代码块，按照在程序中出现的顺序初始化； 

2.然后，初始化子类中的静态成员变量和静态代码块，按照在程序中出现的顺序初始化； 

3.其次，初始化父类的普通成员变量和代码块，在执行父类的构造方法；

4.最后，初始化子类的普通成员变量和代码块，在执行子类的构造方法； 


#  创建对象方法

1、使用 new 关键字（最常用）： ObjectName obj = new ObjectName(); 

2、使用反射的Class类的newInstance()方法： ObjectName obj = ObjectName.class.newInstance(); 

3、使用反射的**Constructor**类的newInstance()方法： ObjectName obj = ObjectName.class.getConstructor.newInstance(); 

4、使用对象克隆**clone()**方法： ObjectName obj = obj.clone(); 

5、使用**反序列化**（ObjectInputStream）的readObject()方法： 



```java
try {
	ObjectInputStream ois = new ObjectInputStream(new FileInputStream(FILE_NAME))
	ObjectName obj = ois.readObject();
}
```

##  switch

```
JDK1.0-1.4 数据类型接受 byte short int char

JDK1.5       数据类型接受 byte short int char enum(枚举)

JDK1.7       数据类型接受 byte short int char enum(枚举)，String 六种类型
```

#  抽象类和接口

类： 抽象类可以实现一部分操作，剩下的留给子类完成  只能单继承
	接口     一般指一种规定 不提供实现  			可以多继承
字段：
	抽象类中 就是一般的成员变量
	接口中的变量默认是public static final修饰
方法：
	可以有抽象方法 可以有实现方法    不能有private abstract
	只能有抽象方法 或者default默认实现   默认为public abstract

抽象类和接口都无法被实例化

#  &和&&

​    （1）、&逻辑运算符称为逻辑与运算符，&&逻辑运算符称为短路与运算符，也可叫逻辑与运算符。

​    对于&：无论任何情况，&两边的操作数或表达式都会参与计算。

​    对于&&：当&&左边的操作数为false或左边表达式结果为false时，&&右边的操作数或表达式将不参与计算，此时最终结果都为false。

​    综上所述，如果逻辑与运算的第一个操作数是false或第一个表达式的结果为false时，对于第二个操作数或表达式是否进行运算，对最终的结果没有影响，结果肯定是false。推介平时多使用&&，因为它效率更高些。

# 运算符

1）不论有什么运算，小括号的优先级都是最高的，先计算小括号中的运算，得到x+y +""+25+y

2）任何字符与字符串相加都是字符串，但是是有顺序的，字符串前面的按原来的格式相加，字符串后面的都按字符串相加，得到25+“”+25+5

3）上面的结果按字符串相加得到25255

# Exception

Throwable
    Exception
    	RuntimeException
    		ClassCastException
    		NullPointerException
    		IndexOutOfBoundsException
    	IOException
    		FileNotFoundException
    	ReflectiveOperationException
   			NoSuchMethodException
    		InvocationTargetException
    	

    Error
    	VirtualMachineError
    		StackOverflowError
    		OutOfMemoryError
IOError

