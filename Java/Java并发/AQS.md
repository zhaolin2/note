# AQS

## 思想

aqs 思想：被请求的共享资源空闲 就把当前线程设置为有效线程  并且把资源设置为锁定* 如果资源被锁定 那么就加入到CLH队列中* 

- CLH队列是一个虚拟的双向队列(不存在实例 只存在节点之间的关系)-
-  把请求的线程封装为队列中的一个节点来进行锁的分配


![1593169571450](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/Java%20%E7%A8%8B%E5%BA%8F%E5%91%98%E5%BF%85%E5%A4%87%EF%BC%9A%E5%B9%B6%E5%8F%91%E7%9F%A5%E8%AF%86%E7%B3%BB%E7%BB%9F%E6%80%BB%E7%BB%93/CLH.png)


## AQS资源共享方式

- Exclusive(独占)  只有一个线程能获取
  - 公平锁 按照排队顺序
  - 非公平锁 不按照排队顺序
- Share 多个线程能同时执行  Semaphore/CountDownLatch

## 需要重写的方法

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。

```

## 源码

主要是对CLH的操作，没有实际队列，只有双端节点

### Node

```java
static final class Node {
        /** waitStatus值，表示线程已被取消（等待超时或者被中断）*/
        static final int CANCELLED =  1;
        /** waitStatus值，表示后继线程需要被唤醒（unpaking）*/
        static final int SIGNAL    = -1;
        /**waitStatus值，表示结点线程等待在condition上，当被signal后，会从等待队列转移到同步到队列中 */
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
       /** waitStatus值，表示下一次共享式同步状态会被无条件地传播下去
        static final int PROPAGATE = -3;
        /** 等待状态，初始为0 */
        volatile int waitStatus;
        /**当前结点的前驱结点 */
        volatile Node prev;
        /** 当前结点的后继结点 */
        volatile Node next;
        /** 与当前结点关联的排队中的线程 */
        volatile Thread thread;
        /** ...... */
    }
```

### acquire()

- 1.首先调用被重写的tryAcquire()

  - 成功则直接返回

  - 失败的话，构造一个独占式的节点来通过cas添加在同步队列尾部

- 3.cas失败则死循环设置尾节点

- 4.尝试添加进队列    

  - 如果前置节点是head 尝试获取锁   
  
- 5.检查自己前置节点状态

  - Singal 直接阻塞
  
  - cancel  从后往前找到一个非cancel的节点 来把它的状态设置为Singal
  
- 6.使用unsafe来把当前阻塞，直到被唤醒或者被中断
```java
public final void acquire(int arg) {
         if (!tryAcquire(arg) &&
             acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
     }
```

### addWaiter

```java
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);//构造结点
        //指向尾结点tail
        Node pred = tail;
        //如果尾结点不为空，CAS快速尝试在尾部添加，若CAS设置成功，返回；否则，enq。
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
```

### enq 

乐观的并发策略

```java
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { //如果队列为空，创建结点，同时被head和tail引用
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {//cas设置尾结点，不成功就一直重试
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

### acquireQueued

只有前继节点式head才有机会尝试获取锁，获取失败则判断是否该进行阻塞

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {//死循环
                final Node p = node.predecessor();//找到当前结点的前驱结点
                if (p == head && tryAcquire(arg)) {//如果前驱结点是头结点，才tryAcquire，其他结点是没有机会tryAcquire的。
                    setHead(node);//获取同步状态成功，将当前结点设置为头结点。
                    p.next = null; // 方便GC
                    failed = false;
                    return interrupted;
                }
                // 如果没有获取到同步状态，通过shouldParkAfterFailedAcquire判断是否应该阻塞，parkAndCheckInterrupt用来阻塞线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

### shouldParkAfterFailedAcquire

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取前驱结点的wait值 
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)//若前驱结点的状态是SIGNAL，意味着当前结点可以被安全地park
            return true;
        if (ws > 0) {
        // ws>0，只有CANCEL状态ws才大于0。若前驱结点处于CANCEL状态，也就是此结点线程已经无效，从后往前遍历，找到一个非CANCEL状态的结点，将自己设置为它的后继结点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {  
            // 若前驱结点为其他状态，将其设置为SIGNAL状态
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

### parkAndCheckInterrupt

阻塞线程并处理中断

```java
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);//使用LockSupport使线程进入阻塞状态
        return Thread.interrupted();// 线程是否被中断过
    }
```

###  Release

- 1.调用使用者重写的tryRelease方法  

  - 成功则唤醒后继节点

  - 失败则返回false

- 2.把node状态设置为0 后续向后遍历找到一个阻塞的节点进行唤醒

```java
public final boolean release(int arg) {
        if (tryRelease(arg)) {//调用使用者重写的tryRelease方法，若成功，唤醒其后继结点，失败则返回false
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒后继结点
            return true;
        }
        return false;
    }
```

### unparkSuccessor

```java
private void unparkSuccessor(Node node) {
        //获取wait状态
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);// 将等待状态waitStatus设置为初始值0
        Node s = node.next;//后继结点
        if (s == null || s.waitStatus > 0) {//若后继结点为空，或状态为CANCEL（已失效），则从后尾部往前遍历找到一个处于正常阻塞状态的结点　　　　　进行唤醒
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);//使用LockSupprot唤醒结点对应的线程
    }
```

###  **acquireShared** 

- 1.调用tryAcquireShared

  - 成功则返回

  - 失败则排队
- 2.排队的时候 如果前置节点是head 尝试进行获取共享锁
  - 成功则把继续传播
  - 失败则进行阻塞
- 3.如果是唤醒状态，则直接设置为默认值 并且持续唤醒

```java
public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)//返回值小于0，获取同步状态失败，排队去；获取同步状态成功，直接返回去干自己的事儿。
            doAcquireShared(arg);
    }
```

### doAcquireShared

释放共享状态

```java
private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);//构造一个共享结点，添加到同步队列尾部。若队列初始为空，先添加一个无意义的傀儡结点，再将新节点添加到队列尾部。
        boolean failed = true;//是否获取成功
        try {
            boolean interrupted = false;//线程parking过程中是否被中断过
            for (;;) {//死循环
                final Node p = node.predecessor();//找到前驱结点
                if (p == head) {//头结点持有同步状态，只有前驱是头结点，才有机会尝试获取同步状态
                    int r = tryAcquireShared(arg);//尝试获取同步装填
                    if (r >= 0) {//r>=0,获取成功
                        setHeadAndPropagate(node, r);//获取成功就将当前结点设置为头结点，若还有可用资源，传播下去，也就是继续唤醒后继结点
                        p.next = null; // 方便GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&//是否能安心进入parking状态
                    parkAndCheckInterrupt())//阻塞线程
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

### setHeadAndPropagate

看是否能继续传播下去  如果是共享锁 则继续释放下去

```java
private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

### doReleaseShared

 

```java
private void doReleaseShared() {
        for (;;) {//死循环，共享模式，持有同步状态的线程可能有多个，采用循环CAS保证线程安全
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;          
                    unparkSuccessor(h);//唤醒后继结点
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                
            }
            if (h == head)              
                break;
        }
    }
```



# JDK实现

## ReentrantLock

总结：公平锁和非公平锁只有两处不同：

1. 非公平锁在调用 lock 后，首先就会调用 **CAS** 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
2. 非公平锁在 CAS 失败后，和公平锁一样都会进入到 tryAcquire 方法，在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断**等待队列是否有线程处于等待状态**，如果有则不去抢锁，乖乖排到后面。

### ReentrantLock 中公平锁的 `lock` 方法

```java
static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }
    // AbstractQueuedSynchronizer.acquire(int arg)
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 1. 和非公平锁相比，这里多了一个判断：是否有线程在等待
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}Copy to clipboardErrorCopied
```

### 非公平锁的 lock 方法：

```java
static final class NonfairSync extends Sync {
    final void lock() {
        // 2. 和公平锁相比，这里会直接先进行一次CAS，成功就返回了
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
    // AbstractQueuedSynchronizer.acquire(int arg)
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 这里没有对阻塞队列进行判断
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

## Semaphore

- 允许多个线程同时访问

### 构造方法

```java
   public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```

### 公平锁

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = 2014338818796000944L;

        FairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
            for (;;) {
                //如果队列中还有等待的节点  有的话 直接排队
                if (hasQueuedPredecessors())
                    return -1;
                //没有才会进行减去资源
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }
```



### 非公平锁

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;

        NonfairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
```

#### 重写的方法

```java
final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                //获取锁 并把资源cas减去
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    //设置state设置为remaining
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
```

## CountDownLatch

- 计数是一次性的

## CycliBarrier

##  ReentrantReadWriteLock  

