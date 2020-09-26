# Volatile

- 可见性/一致性：对一个 volatile 变量的读，总是能看到(任意线程)对这个 volatile 变量**最后的写入**。
-  原子性：对任意**单个 volatile 变量的读/写**具有**原子性**，但类似于 volatile++这种复合操作不具有原子性。
使用场景：
- 写入的值不依赖当前值，如果依赖的话，会分为获取-计算，写入三步操作，volatile无法保证
- 读写变量没有加锁，如果加锁的话，锁本身就保证了可见性，那么不需要声明为volatile


volatile和final都使用了内存屏障，只不过volatile有两个保障点，可见性和有序性，可见性是通过缓存锁以及缓存一致性协议控制，有序性是内存屏障来保证。

- 在每个volatile写操作前插入一个StoreStore屏障；
- 在每个volatile写操作后插入一个StoreLoad屏障；
- 在每个volatile读操作后插入一个LoadLoad屏障；
- 在每个volatile读操作后再插入一个LoadStore屏障。

## 原理

### 可见性

- LOCK#:通过在总线加LOCK#锁的方式
在总线上加锁，会导致其他CPU无法访问主存，效率降低
- 后边都是通过MESI缓存一致性来保证的

### 避免指令重排序

- 通过内存屏障来保证

## 如何保证

只有当线程对变量执行的前一个操作是**load**时，线程才能对变量执行**use**操作；只有线程的后一个操作是use时，线程才能对变量执行load操作。即规定了u**se、load、read**三个操作之间的约束关系，规定这三个操作必须连续的出现，保证了线程每次**读取变量的值前都必须去主存获取最新的值**。

只有当前程对变量执行的前一个操作是**assign**时，线程才能对变量执行store操作；只有线程的后一个操作是store时，线程才能对变量执行assign操作，即规定了assign、store、write三个操作之间的约束关系，规定了这三个操作必须连续的出现，保证线程每次**修改变量后都必须将变量的值写回主存**。



![jmm工作流程](https://images2017.cnblogs.com/blog/352511/201708/352511-20170814091902006-644572602.png)



## 参考

深入浅出多线程   https://redspider.gitbook.io/concurrent/di-er-pian-yuan-li-pian/8 

输入理解Volatile  https://mrbird.cc/volatile.html 