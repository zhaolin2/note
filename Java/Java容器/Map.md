### 散列表

数组链表中的每个列表叫做桶，每个对象都有一个散列码，来根据散列码来保存对象

为了避免散列冲突，在元素的比例到达负载因子，需要重新映射到更大的桶中

- 链地址法
- 开放地址法
- 再hash法

## Map

映射

- hashmap  底层hash表 查找O(1)
- linkedhashmap  保证输入和输出的顺序一样
- hashtable
- treemap  底层是树结构，根据键值排序

## HashMap

- 不保证有序
- 线程不安全
- 数组+链表
  - 在put的时候才会去创建数组 在构造方法的时候 数组为空
  - 链表在hash冲突的时候形成
- 允许null

### 参数

- 初始16  装载因子0.75
- put方法之后 会去判断链表的长度  链表->红黑树  8  并且在树化的时候  map最小容量是64(小于64就扩容)
- 红黑树->链表 6

### 注意点

- h = key.hashCode()) ^ (h >>> 16)

### 为什么长度是2的幂

 为了**加快哈希计算**以及**减少哈希冲突** 

异或

 **length 为偶数时**，length-1 为奇数，奇数的二进制最后一位是 1，这样便保证了 hash &(length-1) 的最后一位可能为 0，也可能为 1 

 如果 length 为奇数的话，很明显 length-1 为偶数，它的最后一位是 0，这样 hash & (length-1) 的最后一位肯定为 0，即只能为偶数 

 为了减少Hash碰撞，尽量使Hash算法的结果均匀分布 

### 为什么是8

随机情况下 在桶中的分布遵循泊松分布 

当链表中元素为8的情况下 概率已经很小了

8: 0.00000006

more: less than 1 in ten million

### HashMap为什么线程不安全？有什么影响？

- 值覆盖问题  当两个线程同时对同一个位置进行赋值操作的时候  会出现值丢失的情况

### hashmap的替代方案

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合；
  - 内部维护了一个普通对象map(private final) 还有互斥锁 	假如没传入互斥锁 那么就会把this当作互斥锁

- Hashtable
  - 读操作的时候也会上锁  效率比较低
- ConcurrentHashMap

### put方法

- 1.首先会根据key来计算出来一个int类型的hash值
- 2.首先判断数组是否为空 空的话就使用resize方法来初始化数组
- 3.根据hash值跟数组最大值的与操作来得到索引位置
- 4.索引位置为空则直接新建一个node放入进去
- 5.有元素则判断 key是否完全相同 相同的话 就把保存下原来的node
- 6.判断是红黑树还是链表
- 7.红黑树就使用红黑树的方式插入
- 8.链表则遍历链表来插入到最后一位
- 9.返回被覆盖的旧值
- 10.判断是否要扩容

HashMap只提供了put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用。

**对putVal方法添加元素的分析如下：**

- ①如果定位到的数组位置没有元素 就直接插入。
- ②如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用`e = ((TreeNode)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

ps:下图有一个小问题，来自 [issue#608](https://github.com/Snailclimb/JavaGuide/issues/608)指出：直接覆盖之后应该就会 return，不会有后续操作。参考 JDK8 HashMap.java 658 行。

![put方法](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/put%E6%96%B9%E6%B3%95.png)

```java
      static final int hash(Object key) {
        int h;
        // key.hashCode()：返回散列值也就是hashcode
        // ^ ：按位异或
        // >>>:无符号右移，忽略符号位，空位都以0补齐
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```



```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```

### get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

```

### resize方法

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { 
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                //表示当前桶中没有后续节点
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## Hashtable

- 数组+链表
- 不允许null的key和null的value
- 在计算位置的时候 取模计算

### 参数

- 初始化11 装载因子0.75
- 扩容 2倍+1
- hash的值是直接使用的hashcode

源码

大部分没区别 基本都带的synchronized

## Linkedhashmap

- 继承hashmap
- 底层是序列表+双向链表
- 插入的顺序是有序的

### 注意

- 可以通过这个类来实现LRUMap

### 源码

```java
/**
     * true表示访问顺序  false表示插入顺序
     * 默认是插入顺序
     * 也就是map中的节点是按照插入还是访问的 进行排序
     */
    final boolean accessOrder;
```

插入使用的是hashmap的插入方法

```java
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
    //如果是访问顺序 就把节点移到最后边
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

## LRUMap

```java
public class LRULinkedHashMap<I extends Number, I1 extends Number> extends LinkedHashMap {

    private int capacity;

    LRULinkedHashMap(int initCapacity){
        super(16,0.75f,true);
        this.capacity=initCapacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        System.out.println("remove entry: "+eldest.getKey()+"="+eldest.getValue());
        return size()>capacity;
    }
}

```

