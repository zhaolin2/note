# Queue

队列一般可以分为以下几种，其中有些可以搭配来形成不同的队列

- Dqueue 双端队列
- BlockingQueue 阻塞队列
- BlockingDqueue
- TransferQueue 使命必达接口 
- PriorityQueue 优先队列

## ArrayDeque

- 环形队列(循环数组)
- 初始16 每次扩大为2 
- 可以代替Stack

注意点

- tail = (tail + 1) &(elements.length - 1)) == head  判断头尾是否相等

- head = (head - 1) & (elements.length - 1)   头部添加

- (tail - 1) & (elements.length - 1)  获取尾部元素


### 源码

**属性**

```java
//用数组存储元素
transient Object[] elements; // non-private to simplify nested class access
//头部元素的索引
transient int head;
//尾部下一个将要被加入的元素的索引
transient int tail;
//最小容量，必须为2的幂次方
private static final int MIN_INITIAL_CAPACITY = 8;
```
**初始化**

```java
//默认容量16
public ArrayDeque(){
    elements = new Object[16];
}
//指定容量
public ArrayDeque(int numElements){
    allocateElements(numElements);
}
//依据给定的集合中的元素进行创建
public ArrayDeque(Collection<? extends E> c){
    allocateElements(c.size());
    addAll(c);
}

```

扩容

```java
private void doubleCapacity(){
    assert head == tail;//扩容时头部索引和尾部索引肯定相等
    int p = head;
    int n = elements.length;
    //头部索引到数组末端(length-1处)共有多少元素
    int r = n - p;
    //容量翻倍
    int newCapacity = n << 1;
    //容量过大，溢出了
    if(newCapacity < 0){
        throw new IllegalStateException("Sorry, deque too big");
    }
    Object[] a = new Object[newCapacity];
    // 既然是head和tail已经重合了，说明tail是在head的左边。
    System.arraycopy(elements, p, a, 0, r);// 拷贝原数组从head位置到结束的数据  
    System.arraycopy(elements, 0, a, r, p);// 拷贝原数组从开始到head的数据  
    elements = a; // 重置head和tail为数据的开始和结束索引  
    head = 0;
    tail = n;
}

// 拷贝该数组的所有元素到目标数组  
private <T> T[] copyElements(T[] a) {  
    if (head < tail) { // 开始索引大于结束索引，一次拷贝  
        System.arraycopy(elements, head, a, 0, size());  
    } else if (head > tail) { // 开始索引在结束索引的右边，分两段拷贝  
        int headPortionLen = elements.length - head;  
        System.arraycopy(elements, head, a, 0, headPortionLen);  
        System.arraycopy(elements, 0, a, headPortionLen, tail);  
    }  
    return a;  
}
```

## PriorityQueue

- 二叉堆  使用数组实现二叉堆

### 注意点

- 默认小顶堆  通过自定义来比较函数来实现小顶堆

### 源码

#### 成员变量

```java
// 默认初始化大小
private static final int DEFAULT_INITIAL_CAPACITY = 11;

// 用数组实现的二叉堆，下面的英文注释确认了我们前面的说法。 
/**
 * Priority queue represented as a balanced binary heap: the two
 * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
 * priority queue is ordered by comparator, or by the elements'
 * natural ordering, if comparator is null: For each node n in the
 * heap and each descendant d of n, n <= d.  The element with the
 * lowest value is in queue[0], assuming the queue is nonempty.
 */
private transient Object[] queue;
// 队列的元素数量 
private int size = 0;
// 比较器,用于指定优先级
private final Comparator<? super E> comparator;
// 修改版本
private transient int modCount = 0;
```

#### 初始化

```java
/**
 * 默认构造方法，使用默认的初始大小来构造一个优先队列，比较器comparator为空，这里要求入队的元素必须实现Comparator接口
 */
public PriorityQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

/**
 * 使用指定的初始大小来构造一个优先队列，比较器comparator为空，这里要求入队的元素必须实现Comparator接口
 */
public PriorityQueue( int initialCapacity) {
    this(initialCapacity, null);
}

/**
 * 使用指定的初始大小和比较器来构造一个优先队列
 */
public PriorityQueue( int initialCapacity,
                     Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    // 初始大小不允许小于1
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    // 使用指定初始大小创建数组
    this.queue = new Object[initialCapacity];
    // 初始化比较器
    this.comparator = comparator;
}

/**
 * 构造一个指定Collection集合参数的优先队列
 */
public PriorityQueue(Collection<? extends E> c) {
    // 从集合c中初始化数据到队列
    initFromCollection(c);
    // 如果集合c是包含比较器Comparator的(SortedSet/PriorityQueue)，则使用集合c的比较器来初始化队列的Comparator
    if (c instanceof SortedSet)
        comparator = (Comparator<? super E>)
            ((SortedSet<? extends E>)c).comparator();
    else if (c instanceof PriorityQueue)
        comparator = (Comparator<? super E>)
            ((PriorityQueue<? extends E>)c).comparator();
    //  如果集合c没有包含比较器，则默认比较器Comparator为空
    else {
        comparator = null;
        // 调用heapify方法重新将数据调整为一个二叉堆
        heapify();
    }
}

/**
 * 构造一个指定PriorityQueue参数的优先队列
 */
public PriorityQueue(PriorityQueue<? extends E> c) {
    comparator = (Comparator<? super E>)c.comparator();
    initFromCollection(c);
}

/**
 * 构造一个指定SortedSet参数的优先队列
 */
public PriorityQueue(SortedSet<? extends E> c) {
    comparator = (Comparator<? super E>)c.comparator();
    initFromCollection(c);
}

/**
 * 从集合中初始化数据到队列
 */
private void initFromCollection(Collection<? extends E> c) {
    // 将集合Collection转换为数组a
    Object[] a = c.toArray();
    // If c.toArray incorrectly doesn't return Object[], copy it.
    // 如果转换后的数组a类型不是Object数组，则转换为Object数组
    if (a.getClass() != Object[].class)
        a = Arrays. copyOf(a, a.length, Object[]. class);
    // 将数组a赋值给队列的底层数组queue
    queue = a;
    // 将队列的元素个数设置为数组a的长度
    size = a.length ;
}
```