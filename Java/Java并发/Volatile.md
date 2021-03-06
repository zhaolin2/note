# Volatile

- 可见性/一致性：对一个 volatile 变量的读，总是能看到(任意线程)对这个 volatile 变量**最后的写入**。
- 原子性：对任意**单个 volatile 变量的读/写**具有**原子性**，但类似于 volatile++这种复合操作不具有原子性。
  使用场景：

  - 写入的值不依赖当前值，如果依赖的话，会分为获取-计算，写入三步操作，volatile无法保证

  - 读写变量没有加锁，如果加锁的话，锁本身就保证了可见性，那么不需要声明为volatile


volatile和final都使用了内存屏障，只不过volatile有两个保障点，可见性和有序性，可见性是通过缓存锁以及缓存一致性协议控制，有序性是内存屏障来保证。

- 在每个volatile写操作前插入一个StoreStore屏障；
- 在每个volatile写操作后插入一个StoreLoad屏障；
- 在每个volatile读操作后插入一个LoadLoad屏障；
- 在每个volatile读操作后再插入一个LoadStore屏障。

### 避免指令重排序

- 通过内存屏障来保证

## 如何保证

只有当线程对变量执行的前一个操作是**load**时，线程才能对变量执行**use**操作；只有线程的后一个操作是use时，线程才能对变量执行load操作。即规定了u**se、load、read**三个操作之间的约束关系，规定这三个操作必须连续的出现，保证了线程每次**读取变量的值前都必须去主存获取最新的值**。

只有当前程对变量执行的前一个操作是**assign**时，线程才能对变量执行store操作；只有线程的后一个操作是store时，线程才能对变量执行assign操作，即规定了assign、store、write三个操作之间的约束关系，规定了这三个操作必须连续的出现，保证线程每次**修改变量后都必须将变量的值写回主存**。



![jmm工作流程](https://images2017.cnblogs.com/blog/352511/201708/352511-20170814091902006-644572602.png)



## 参考

深入浅出多线程   https://redspider.gitbook.io/concurrent/di-er-pian-yuan-li-pian/8 

输入理解Volatile  https://mrbird.cc/volatile.html 

# 缓存一致性

- LOCK#:通过在总线加LOCK#锁的方式
  在总线上加锁，会导致其他CPU无法访问主存，效率降低
- 后边都是通过MESI缓存一致性来保证的

MESI的核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

在MESI协议中，每个缓存可能有有4个状态，它们分别是：

> **M(Modified)**：这行数据有效，数据被修改了，和内存中的数据不一致，数据只存在于本Cache中。
>
> **E(Exclusive)**：这行数据有效，数据和内存中的数据一致，数据只存在于本Cache中。
>
> **S(Shared)**：这行数据有效，数据和内存中的数据一致，数据存在于很多Cache中。
>
> **I(Invalid)**：这行数据无效。

传统的MESI的两个行为执行成本较大：

-  一个是将某个Cache Line标记为Invalid状态
- 另一个是当某Cache Line当前状态为Invalid时写入新的数据。所以CPU通过Store Buffer和Invalidate Queue组件来降低这类操作的延时。 

 ![cache_sync](http://47.103.216.138/wp-content/uploads/2018/08/cache_sync.png) 

所以，为了解决缓存的一致性问题，比较典型的方案是MESI缓存一致性协议。

**MESI协议，可以保证缓存的一致性，但是无法保证实时性。**

 **缓存一致性协议**：每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。 

# 内存模型

缓存一致性（Cache Coherence），解决是多个缓存副本之间的数据的一致性问题。

内存一致性（Memory Consistency），保证的是多线程程序访问内存时可以读到什么值。

 内存一致性，就是保证并发场景下的程序运行结果和程序员预期是一样的（当然，要通过加锁等方式），包括的就是并发编程中的原子性、有序性和可见性。

而缓存一致性说的就是并发编程中的可见性。 

# 已经有了缓存一致性协议，为什么还需要volatile？

这个问题的答案可以从多个方面来回答：

> 1、并不是所有的硬件架构都提供了相同的一致性保证，Java作为一门跨平台语言，JVM需要提供一个统一的语义。
>
> 2、操作系统中的缓存和JVM中线程的本地内存并不是一回事，通常我们可以认为：MESI可以解决缓存层面的可见性问题。使用volatile关键字，可以解决JVM层面的可见性问题。
>
> 3、缓存可见性问题的延伸：由于传统的MESI协议的执行成本比较大。所以CPU通过Store Buffer和Invalidate Queue组件来解决，但是由于这两个组件的引入，也导致缓存和主存之间的通信并不是实时的。也就是说，**缓存一致性模型只能保证缓存变更可以保证其他缓存也跟着改变，但是不能保证立刻、马上执行。**