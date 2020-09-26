# Set

- hashset
- treeset
- linkedhashset

## HashSet

- 允许null
- 底层是一个HashMap实例
- 初始容量影响迭代性能

### 注意点

- 增删改查都是使用hashmap来实现

### 源码

#### 成员变量

```java
static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    // map对应的value
    private static final Object PRESENT = new Object();
```

#### 初始化

```java
    //16 0.75
public HashSet() {
        map = new HashMap<>();
    }


    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

## TreeSet

- 底层是TreeMap( Navigablemap )
- 实现排序

### 源码

### 成员变量

```java

    private transient NavigableMap<E,Object> m;

    //对应的value
    private static final Object PRESENT = new Object();
```

## LinkedHashSet

- 迭代有序
- 允许null
- linkedhashMap

实现就像是linkedhashmap和hashmap的实现一样

