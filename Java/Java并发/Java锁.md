[toc]

## 锁
美团技术团队：https://tech.meituan.com/2018/11/15/java-lock.html
![锁的分类](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/7f749fc8.png)
## aqs
AbstractQueuedSynchronizer

**源码解析**：
[aqs解析](https://juejin.im/post/5a4a4530518825697078553e#heading-9)



## ReentrantLock

默认是非公平锁
```

public class ReentrantLock implements Lock, java.io.Serializable {

    private static final long serialVersionUID = 7373984872572414699L;
    
    //通过同步器来提供锁 其中提供了两个实现 公平和非公平锁
    private final Sync sync;
    
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```

### NonfairSync


```
static final class NonfairSync extends Sync {

        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
            //java.util.concurrent.locks.AbstractQueuedSynchronizer#acquire
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    
    //非公平锁的加锁
    final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
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

### fairSync


```
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
         //
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //如果前边没有节点 就尝试获取锁
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
    }
    
    public final boolean hasQueuedPredecessors() {
        //第一个节点为虚节点，只是占位
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
        //h==t 队列没有进行初始化
        // h!=t时 (s = h.next) == null  待队列正在有线程进行初始化，但只是进行到了Tail指向Head，没有将Head指向Tail，此时队列中有元素，需要返回True
        //(s = h.next) != null，说明此时队列中至少有一个有效节点 
        // s.thread == Thread.currentThread()，说明等待队列的第一个有效节点中的线程与当前线程相同，那么当前线程是可以获取资源的
        //s.thread != Thread.currentThread()，说明等待队列的第一个有效节点线程与当前线程不同，当前线程必须加入进等待队列
    }
```