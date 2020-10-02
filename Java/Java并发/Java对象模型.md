# oop-klass Model

Orinary Object Pointer指的是普通对象指针，

 `Klass`用来描述对象实例的具体类型 

这样分开的目的是为了不让每个对象中都含有虚函数表。

## oop

 **在Java程序运行过程中，每创建一个新的对象，在JVM内部就会相应地创建一个对应类型的OOP对象。**在HotSpot中，根据JVM内部使用的对象业务类型，具有多种`oopDesc`的子类。除了`oppDesc`类型外，opp体系中还有很多`instanceOopDesc`、`arrayOopDesc` 等类型的实例，他们都是`oopDesc`的子类。 

 ![OOP结构](http://47.103.216.138/wp-content/uploads/2017/12/OOP%E7%BB%93%E6%9E%84.png) 

 这些OOPS在JVM内部有着不同的用途，例如**，`instanceOopDesc`表示类实例，`arrayOopDesc`表示数组。**也就是说，**当我们使用`new`创建一个Java对象实例的时候，JVM会创建一个`instanceOopDesc`对象来表示这个Java对象。同理，当我们使用`new`创建一个Java数组实例的时候，JVM会创建一个`arrayOopDesc`对象来表示这个数组对象。** 

- 对象头
  - _mark
  - _metadata    union _metadata
    - _klass
    - _compressed_klass
- 实例数据
- 对齐填充

## Klass

 ![klass](http://47.103.216.138/wp-content/uploads/2017/12/klass.png) 

Klass向JVM提供两个功能：

- 实现语言层面的Java类（在Klass基类中已经实现）
- 实现Java对象的分发功能（由Klass的子类提供虚函数实现）

 `_metadata`是一个共用体，其中`_klass`是普通指针，`_compressed_klass`是压缩类指针。

这两个指针都指向`instanceKlass`对象，它用来描述对象的具体类型。 

 ![2579123-5b117a7c06e83d84](http://47.103.216.138/wp-content/uploads/2017/12/2579123-5b117a7c06e83d84.png) 

 对象的实例（instantOopDesc)保存在堆上，对象的元数据（instantKlass）保存在方法区，对象的引用保存在栈上。 

# 对象头

```c++
enum { 
    age_bits                 = 4,
      lock_bits                = 2,
      biased_lock_bits         = 1,
      max_hash_bits            = BitsPerWord - age_bits - lock_bits - biased_lock_bits,
      hash_bits                = max_hash_bits > 31 ? 31 : max_hash_bits,
      cms_bits                 = LP64_ONLY(1) NOT_LP64(0),
      epoch_bits               = 2
};

 enum { locked_value             = 0,
         unlocked_value           = 1,
         monitor_value            = 2,
         marked_value             = 3,
         biased_lock_pattern      = 5
  };
```

