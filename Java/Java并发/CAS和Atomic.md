[toc]

# CAS

- 主要是通过Unsafe类来实现的
- 乐观锁

CAS的全称是：比较并交换（Compare And Swap）。在CAS中，有这样三个值：

- V：要更新的变量(var)
- E：预期值(expected)
- N：新值(new)

比较并交换的过程如下：

判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。

## Unsafe

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);
boolean compareAndSwapInt(Object o, long offset,int expected,int x);
boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

## 实现

compareset 底层通过这个来保证

```shell
lock cmpxchg
```

## 缺点

### ABA

当出现ABA问题的时候，是没办法解决的，需要追加版本号或者时间戳

-  AtomicStampedReference 

### 循环时间浪费CPU

CAS一般跟自旋来配合，当长时间不成功，会浪费CPU资源

解决思路是让JVM支持处理器提供的**pause指令**。

pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。

### 只能保证一个变量的原子操作

- 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；
- 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

# Atomic

### AtomicInteger
Integer原子类

```java
//偏移量
 private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    //int值
    private volatile int value;
    
    /**
     * Atomically sets to the given value and returns the old value.
     *
     * @param newValue the new value
     * @return the previous value
     */
    public final int getAndSet(int newValue) {
        return unsafe.getAndSetInt(this, valueOffset, newValue);
    }
    
    /**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

### AtomicIntegerArray
Integer数组原子类

``` java
    //第一个元素地址相对于数组起始地址的偏移值
    private static final int base = unsafe.arrayBaseOffset(int[].class);
    private static final int shift;
    private final int[] array;

    //base + i * sacle = 索引i的元素在数组中的内存起始地址
    static {
        //返回指定类型数组的元素所占用的字节数
        int scale = unsafe.arrayIndexScale(int[].class);
        if ((scale & (scale - 1)) != 0)
            throw new Error("data type scale not a power of two");
        //将scale转换为2进制，然后从左往右数连续0的个数
        //shift=scale = 2 ^ shift
        shift = 31 - Integer.numberOfLeadingZeros(scale);
    }
    
    
    private long checkedByteOffset(int i) {
        if (i < 0 || i >= array.length)
            throw new IndexOutOfBoundsException("index " + i);

        return byteOffset(i);
    }
    
    //返回索引i在数组中的内存偏移量
    private static long byteOffset(int i) {
        return ((long) i << shift) + base;
    }
    
    public final int get(int i) {
        return getRaw(checkedByteOffset(i));
    }
    
    //知道偏移量之后 直接取值
    private int getRaw(long offset) {
        return unsafe.getIntVolatile(array, offset);
    }
    
    /**
    原子的设置值
     * Sets the element at position {@code i} to the given value.
     *
     * @param i the index
     * @param newValue the new value
     */
    public final void set(int i, int newValue) {
        unsafe.putIntVolatile(array, checkedByteOffset(i), newValue);
    }
    
    /**
     * Atomically sets the element at position {@code i} to the given
     * value and returns the old value.
     *
     * @param i the index
     * @param newValue the new value
     * @return the previous value
     */
    public final int getAndSet(int i, int newValue) {
        return unsafe.getAndSetInt(array, checkedByteOffset(i), newValue);
    }
```

### AtomicIntegerFieldUpdater
单个字段的原子更新类
#### 适用范围
一般用于读多写少的属性，可以用这个来节省空间，更新的时候使用这个类来更新，访问就使用Volatile
```java
//得到一个属性修改器
public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass,
                                                              String fieldName) {
        return new AtomicIntegerFieldUpdaterImpl<U>
            (tclass, fieldName, Reflection.getCallerClass());
    }
    
    private static final class AtomicIntegerFieldUpdaterImpl<T>
        extends AtomicIntegerFieldUpdater<T> {
        private static final sun.misc.Unsafe U = sun.misc.Unsafe.getUnsafe();
        //要得到这个属性相对于起始地址的偏移量
        private final long offset;
        // 如果字段受保护，cclass为调用者类的class对象，否则cclass为tclass
        private final Class<?> cclass;
        
        //targetclass
        private final Class<T> tclass;
        
        //使用unsafe的cas来确保修改
        public final boolean compareAndSet(T obj, int expect, int update) {
            accessCheck(obj);
            return U.compareAndSwapInt(obj, offset, expect, update);
        }
        
        /**
         * Checks that target argument is instance of cclass.  On
         * failure, throws cause.
         */
        private final void accessCheck(T obj) {
            if (!cclass.isInstance(obj))
                throwAccessCheckException(obj);
        }

```

### AtomicStampedReference

加时间戳的原子类
```java
public class AtomicStampedReference<V> {

    //真正存储的数据
    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;
    
    //返回当前的包装类型 0下标显示现在的版本
    public V get(int[] stampHolder) {
        Pair<V> pair = this.pair;
        stampHolder[0] = pair.stamp;
        return pair.reference;
    }
    
     public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
    
    private boolean casPair(Pair<V> cmp, Pair<V> val) {
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
    }
```

