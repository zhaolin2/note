# Object

所有的java对象都默认的继承了Object对象，所以所有java对象都含有Object的方法

# 方法

##  registerNatives 

- 注册这个类中包含的本地方法(出了registerNatives之外的 本地方法)

- 加载的时候 直接链接到调用方

##  **getClass** 

获取到对象的运行时的class类型

##  hashCode

默认是通过对象的地址转化来的

## equals

默认是==来进行比较

在进行两个对象的比较的时候，首先需要比较**hashcode**，然后才是进行**equals**比较

所以**equal相等**，**hashcode一定要相等** 

hashcode就像是一个**大的方向**，equal相当于精确的**点对点比较**

##  clone 

虽然object有clone方法，所以自类可以在他自己的方法中调用，但是实际上是没办法通过**实例对象来直接调用**

所以假如想实现clone，还是需要实现clone方法，自己添加实现

### 浅拷贝

浅复制就仅仅拷贝了引用类型的引用，并没有复制引用的值，在原引用发生改变时，拷贝的对象也会发生变化(也就是**两个直接指针 指向 同一个堆中的对象**)

### 深拷贝

深拷贝就是在实现中，每个引用也都进行clone，也就是两个直接指针**指向两个不同的堆中的对象**

## wait

释放共享锁，从运行状态中退出，进入等待队列，直到被再次唤醒

**会释放掉锁**

sleep方法仅仅是释放cpu，并没有释放锁

## notify

随机唤醒一个在监视器的队列中等待的线程 (由监听器的拥有者来调用) 



参考 java多线程通信  https://redspider.gitbook.io/concurrent/di-yi-pian-ji-chu-pian/5 

wait和notify都需要在同步代码块(**同步方法 同步代码块 同步静态代码块**)中来进行调用

wait有4种情况来唤醒：

- 中断
- wait时间到了
- 被notify唤醒
- 被notifyall唤醒

线程之间的通信示例：

```java
public class WaitAndNotify {
    private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadA: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }

    static class ThreadB implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadB: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //防止主程序不结束
                lock.notify();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        Thread.sleep(1000);
        new Thread(new ThreadB()).start();
    }
}

// 输出：
ThreadA: 0
ThreadB: 0
ThreadA: 1
ThreadB: 1
ThreadA: 2
ThreadB: 2
ThreadA: 3
ThreadB: 3
ThreadA: 4
ThreadB: 4
```



##  **finalize** 

gc的时候启动，在被回收的时候调用，只会被调用一次