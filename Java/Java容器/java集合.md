## 集合类型关系

- Collection:

  - List

    - LinkedList 双向链表  list queue  允许null
    - ArrayList   默认 10   扩容1.5倍 允许null
    - Vector 初始化为10   每次扩大为2倍   允许null
      - stack  继承自Vector 同步使用synchronized
  - Set

    -  hashSet  底层使用hashmap来进行存储
       -  LinkedHashSet
       -  TreeSet
  - Queue
    - PriorityQueue 优先级队列  8   小于64 每次+2  大于64 每次2倍
    - ArrayDeque 双端队列 16  扩容为2倍 不允许null
  
- Map

  - HashMap     开始长度16 负载因子0.75  扩容2 当前节点的链表长度大于8之后 变为红黑树
  - hashMap.put(null,null);  key=null 为hash出来为0       扩容  *2+1
  - HashTable     初始化11 负载因子0.75    key和value都不能为null 
  - TreeMap  按照key的自然排序 经典红黑树 没有容量限制
  
  

## Collection

Collection主要是集合类的公共接口，定义了集合类的所需要的基本操作

- 长度可变
- 可以存储不同类型元素(一般不这么干)
- 只能存储引用类型

![3y Collection](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2CZJ5rW93t1ZycoBZQ0PXqib4Tq597KFtdMoTOuCsiabhO8nPiax3510KJKhswVoAPwEbjIn54Er4Sg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 迭代器

迭代器作为一个设计模式，在java中作为一个功能接口，被大部分的容器所实现

- hasNext
- next
- remove

基本都由具体的集合类型来用内部类的方式来进行实现，

来根据自己的数据结构和遍历方式来具体实现

### ListIterator

作为迭代器的子类，添加了list中可以使用到的方法

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2CZJ5rW93t1ZycoBZQ0PXq6o0Y7Pdn7QHSeYSyrRDhbylYRZgx0bToQ1FlR4r1dyLpoy2GCfxFcg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用list的专用的迭代器，可以进行更加灵活的操作

## List

- 有序  存入和取出顺序一致
- 可重复

## Set

- 元素不可重复

## Queue

Queue主要用在生产者消费者模型中

- BlockingQueue
  - 在获取元素时(或者达到队列最大容量)，队列为空则阻塞
- Deque  （Double ended Queue）
  - 可以在头部和尾部进行操作，FIFO的时候就是就是Queue，LIFO就是Stack
- TransferQueue
  - 跟BlockingQueue很像，不过生产者会阻塞到有消费者来消费

## Map

map也叫映射

![](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3lPrpniaBWDf3s5O0JcibBDIDFbv4yQsialSxSUFhExoxrZFR6AibswibTXJMsDtbaE1YQibtAGK2xcoTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为不在乎是否有序，为了实现快速查找，所以一般使用散列表(hashcode)来进行查找

## 是否允许null值

### 允许null值的

- List
  - arraylist
  - linkedlist
- Queue
  - 

- Map
  - hashmap

## 扩容倍数

一般都为2倍 arraylist 1.5


## fail-fast

**快速失败（fail—fast）**是java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。 

也就是modCount==exceptedCount

 **java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。** 

## fail-safe

采用安全失败机制的集合容器,在遍历时不是直接在集合内容上访问的,而是先copy原有集合内容,在拷贝的集合上进行遍历.

原理:

由于迭代时是对原集合的拷贝的值进行遍历,所以在遍历过程中对原集合所作的修改并不能被迭代器检测到,所以不会出发ConcurrentModificationException
缺点:

基于拷贝内容的优点是避免了ConcurrentModificationException,但同样地, 迭代器并不能访问到修改后的内容 (简单来说就是, 迭代器遍历的是开始遍历那一刻拿到的集合拷贝,在遍历期间原集合发生的修改迭代器是不知道的)

 **java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。** 

## hashtable和hashmap区别

hashtable不允许键或者值为空	hashmap的键和值都可以为空

因为hashmap对null做了处理，还有是因为hashtable采用的是安全失败(**fail-safe**)机制

这种机制会使你此次读到的数据不一定是最新的数据。

如果你使用null值，就会使得其无法判断对应的key是不存在还是为空，因为你无法再调用一次contain(key）来对key是否存在进行判断，ConcurrentHashMap同理。




实现方式：hashtable继承的是 Dictionary (jdk 1.0)，hashmap继承的是 AbstractMap  

初始化容量：hashmap初始化16   hashtable默认11    负载因子都是0.75

扩容机制：hashmap到达最大容量*负载因子的时候  扩容为2倍    hashtable则为2倍+1

迭代器不同: hashmap的iterator迭代器是**fail-fast**  hashtable的enumerator不是**fail-fas**t的

所以，其他线程改变了hashmap的结构，会抛出ConcurrentModificationException,hashtable则不会



java.util   下的集合类都是快速失败的，不能在多线程下做并发修改 (迭代修改)

```java
class Properties extends Hashtable<Object,Object> {
```

## listIterator

add方法   随机添加的话linkedlist快 但是正常add的话arraylist快

hashmap和hashtable

  --hashMap去掉了HashTable 的contains方法，但是加上了containsValue（）和containsKey（）方法。 

  --hashTable同步的，而HashMap是非同步的，效率上比hashTable要高。 

  --hashMap允许空键值，而hashTable不允许



## 如何实现数组和List 之间的转换？

--List转换成为数组：调用ArrayList的toArray方法。

--数组转换成为List：调用Arrays的asList方法。

  


  poll() 和 remove() 都是从队列中取出一个元素，但是 poll() 在获取元素失败的时候会返回空，但是 remove() 失败的时候会抛出异常。



  --enumeration：枚举，相当于迭代器。 



 ## 迭代器Iterator 是什么？

迭代器是一种设计模式，它是一个对象，它可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构。迭代器通常被称为“轻量级”对象，因为创建它的代价小。

  


  Java中的Iterator功能比较简单，并且只能单向移动： 

  (1) 使用方法iterator()要求容器返回一个Iterator。第一次调用Iterator的next()方法时，它返回序列的第一个元素。注意：iterator()方法是java.lang.Iterable接口,被Collection继承。 

  (2) 使用next()获得序列中的下一个元素。 

  (3) 使用hasNext()检查序列中是否还有元素。 

  (4) 使用remove()将迭代器新返回的元素删除。 

  Iterator是Java迭代器最简单的实现，为List设计的ListIterator具有更多的功能，它可以从两个方向遍历List，也可以从List中插入和删除元素。



  --Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。 

  --Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。 

  --ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。



##  工具类

数组  sysytem.arraycopy()

```java
 	/* @param      src      the source array.
     * @param      srcPos   starting position in the source array.
     * @param      dest     the destination array.
     * @param      destPos  starting position in the destination data.
     * @param      length   the number of array elements to be copied.
     */
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

collections

```java
private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
    二分搜索
        同步list
        返回不可修改的list
```




  通过Collections工具类提供的方法 

  --Collections.unmodifiableMap(map) 

  --Collections.unmodifiableList(list) 

  --Collections..unmodifiableSet(set) 

  来创建一个只读集合，这样改变集合的任何操作都会抛出Java. lang. UnsupportedOperationException 异常。