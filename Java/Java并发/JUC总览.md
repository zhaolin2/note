# 主要包括

- 并发容器
  - copyonwritearraylist 初始为1  需要就加1 ReentrantLock
  - ArrayBlockingQueue 自定义 初始化之后无法改变
  - ConcurrentLinkedQueue
  - concurrentHashMap  16 0.75 2
  - ConcurrentSkipListMap
- 原子类
  - AtomicInteger
  - AtomicReference
- 锁
  - ReentrantLock
  - ReentrantReadWriteLock
- 线程通信类
  - Semaphore 限制线程数量
  - Exchanger 两个线程交换数据
  - CountDownLatch  等待计数器为0 开始工作
  - CyclibBarrier  作用跟CountDownLatch ，可以重复使用
  - Phaser 增强的Phaser

通信类可以参考：通信工具类   https://redspider.gitbook.io/concurrent/di-san-pian-jdk-gong-ju-pian/17 

## 容器总览

