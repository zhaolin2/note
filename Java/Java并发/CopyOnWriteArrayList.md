- Vector几乎每个方法都加了synchronized
-  Collections.synchronizedList   每个方法内部也都是使用synchronized(mutex)

## CopyOnWrite

 写入时复制（英语：Copy-on-write，简称COW）是一种计算机程序设计领域的优化策略。其核心思想是，如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的（transparently）。此作法主要的优点是如果调用者没有修改该资源，就不会有副本（private copy）被建立，因此多个调用者只是读取操作时可以共享同一份资源。 

- 所以希望写入的数据能准确读取，不要使用CopyOnWrite容器

## CopyOnWriteArrayList

## 注意点

- 底层通过复制数组来实现
- 遍历的时候 不用加锁
- 元素可以为null
- 修改的时候 复制出来新数组 操作在新数组中完成  最后把新数组设置为array
- 写加锁 读不加锁

缺点：

- 耗费内存
- 只能保证数据的最终一致性，不能保证实时一致性

## 成员变量

```java
/** 可重入锁对象 */
    final transient ReentrantLock lock = new ReentrantLock();

    /** CopyOnWriteArrayList底层由数组实现，volatile修饰 */
    private transient volatile Object[] array;
```

## add

 在添加的时候就上锁，并**复制一个新数组，增加操作在新数组上完成，将array指向到新数组中**，最后解锁。 

```java
public boolean add(E e) {

        // 加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {

            // 得到原数组的长度和元素
            Object[] elements = getArray();
            int len = elements.length;

            // 复制出一个新数组
            Object[] newElements = Arrays.copyOf(elements, len + 1);

            // 添加时，将新元素添加到新数组中
            newElements[len] = e;

            // 将volatile Object[] array 的指向替换成新数组
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

## set

```java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {

        // 得到原数组的旧值
        Object[] elements = getArray();
        E oldValue = get(elements, index);

        // 判断新值和旧值是否相等
        if (oldValue != element) {

            // 复制新数组，新值在新数组中完成
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len);
            newElements[index] = element;

            // 将array引用指向新数组
            setArray(newElements);
        } else {
            // Not quite a no-op; enssures volatile write semantics
            setArray(elements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

## 迭代器

每次迭代器都是重新创建一个新的迭代器，其中，迭代器中使用的是旧数组

```java
// 1. 返回的迭代器是COWIterator
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }


    // 2. 迭代器的成员属性
    private final Object[] snapshot;
    private int cursor;

    // 3. 迭代器的构造方法
    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    // 4. 迭代器的方法...
    public E next() {
        if (! hasNext())
            throw new NoSuchElementException();
        return (E) snapshot[cursor++];
    }

    //.... 可以发现的是，迭代器所有的操作都基于snapshot数组，而snapshot是传递进来的array数组
```



# CopyOnWriteSet

- 使用CopyOnWriteArrayList来实现

## 成员变量

```java
private final CopyOnWriteArrayList<E> al;
```

### add

```java
//不存在就添加
public boolean add(E e) {   
    return al.addIfAbsent(e);
}
```

## CopyOnWrite使用

```java
import java.util.Collection;
import java.util.Map;
import java.util.Set;

public class CopyOnWriteMap<K, V> implements Map<K, V>, Cloneable {
    private volatile Map<K, V> internalMap;

    public CopyOnWriteMap() {
        internalMap = new HashMap<K, V>();
    }

    public V put(K key, V value) {
        synchronized (this) {
            Map<K, V> newMap = new HashMap<K, V>(internalMap);
            V val = newMap.put(key, value);
            internalMap = newMap;
            return val;
        }
    }

    public V get(Object key) {
        return internalMap.get(key);
    }

    public void putAll(Map<? extends K, ? extends V> newData) {
        synchronized (this) {
            Map<K, V> newMap = new HashMap<K, V>(internalMap);
            newMap.putAll(newData);
            internalMap = newMap;
        }
    }
}
```

### 黑名单服务

```java
// 黑名单服务
public class BlackListServiceImpl {
    //　减少扩容开销。根据实际需要，初始化CopyOnWriteMap的大小，避免写时CopyOnWriteMap扩容的开销。
    private static CopyOnWriteMap<String, Boolean> blackListMap = 
        new CopyOnWriteMap<String, Boolean>(1000);

    public static boolean isBlackList(String id) {
        return blackListMap.get(id) == null ? false : true;
    }

    public static void addBlackList(String id) {
        blackListMap.put(id, Boolean.TRUE);
    }

    /**
     * 批量添加黑名单
     * (使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。
     * 如使用上面代码里的addBlackList方法)
     * @param ids
     */
    public static void addBlackList(Map<String,Boolean> ids) {
        blackListMap.putAll(ids);
    }

}
```

