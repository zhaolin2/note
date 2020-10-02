# synchronized

对于同步方法，JVM采用 ACC_SYNCHRONIZED 标记同步(隐式同步)，对于同步代码块，JVM采用monitorenter,monitorexit两个指令来进行同步。

- 原子性
  - 使用montorenter和exit来保证在同一个时间只能被一个线程访问
- 可见性
  - 被syncnronized修饰的代码，在结束的时候会解锁，根据JMM，在对一个变量解锁的时候，需要把变量同步回主内存，从而保证可见性
- 有序性
  - 因为monitor和as-if-serial的保证，单个代码块只有单个线程访问，并且在单个线程的视角中，可以保证有序性

## 锁优化



## 偏向锁

加锁：

1. 访问markword 看当前对象的是否是偏向锁

2. 如果是偏向锁 那么测试当前线程ID是否指向当前线程，是的话进入5，否则进入3

3. 没有指向当前线程，那么通过cas来竞争操作锁(把mark word的线程ID设置为当前线程ID，成功的话，执行5，否则执行4

4. 如果cas获取偏向锁失败，表示有竞争，当到达全局安全点的时候，获取偏向锁的线程被挂起，进行锁升级，然后被阻塞在安全点的线程继续执行

5. 执行同步代码块

释放锁：

1. 新来的线程尝试cas操作，成功则获得偏向锁
2. 失败，那么等待持有的线程达到安全点
3. 暂停持有偏向锁的线程，如果已经退出，或者未活动，就把对象头设置为无锁状态
4. 如果没有退出 则升级到轻量级锁

安全点：

- 循环的末尾

- 方法临返回前

- 调用方法之后

- 抛异常的位置

## 轻量级锁(自旋锁)

使用monitor需要从用户态切换到内核态来进行操作，会花费很多处理器时间。所以在JDK1.4引入自旋锁，在JDK1.6默认开启，不再每次进入等到waitset。

加锁过程：

1. 分配空间并复制mark word到栈，尝试使用cas获取锁，成功的话，会把对象头的markword替换为指向所记录的指针，失败进行自旋
2. 执行完开始解锁

解锁：

cas把栈中的 displaced mark word 替换会对象头，成功表示无竞争，失败则表示有锁竞争，

 <img src="https://pic1.zhimg.com/v2-9db4211af1be81785f6cc51a58ae6054_r.jpg" alt="preview" style="zoom:200%;" /> 

 

# monitor

类似操作系统中管程的概念，每次只会有一个线程来进行访问。

 管程 (英语：Monitors，也称为监视器) 是一种程序结构，结构内的多个子程序（对象或模块）形成的多个工作线程互斥访问共享资源。这些共享资源一般是硬件设备或一群变量。管程实现了在一个时间点，最多只有一个线程在执行管程的某个子程序。与那些通过修改数据结构实现互斥访问的并发程序设计相比，管程实现很大程度上简化了程序设计。 管程提供了一种机制，线程可以临时放弃互斥访问，等待某些条件得到满足后，重新获得执行权恢复它的互斥访问。 

## 监视器的实现

 objectMonitor.hpp 

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //用来记录该线程获取锁的次数
    _waiters      = 0,
    _recursions   = 0; //锁的重入次数
    _object       = NULL;
    _owner        = NULL; //指向持有ObjectMonitor对象的线程
    _WaitSet      = NULL; //存放处于wait状态的线程队列
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //存放处于等待锁block状态的线程队列
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

当多个线程同时访问一段同步代码时，首先会进入`_EntryList`队列中，当某个线程获取到对象的monitor后进入`_Owner`区域并把monitor中的`_owner`变量设置为当前线程，同时monitor中的计数器`_count`加1。即获得对象锁。

若持有monitor的线程调用`wait()`方法，将释放当前持有的monitor，`_owner`变量恢复为`null`，`_count`自减1，同时该线程进入`_WaitSet`集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。

 ![img](https://mmbiz.qpic.cn/mmbiz_png/6fuT3emWI5IxkkunUZsCtBZ1aD6w3x1cGolhT461rBeRIwQQarKeEI0eG5oUWLiawJmbDCFSdibDF3QKqNWhn18g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

具体过程：https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650120784&idx=1&sn=3436c978f0d7ab3bb672d03689518902&chksm=f36bbf71c41c36672a3a6a7edebe0b913f2f5cf33d75d594d228f086ec7bdb9e253ab0beeae4&scene=21#wechat_redirect