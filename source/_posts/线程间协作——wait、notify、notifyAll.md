---
title: 线程间协作——wait、notify、notifyAll
date: 2016-07-08 21:32:26
categories: Java
tags: Java多线程
---
在 Java 中，可以通过配合调用 Object 对象的 wait() 方法和 notify()方法或 notifyAll() 方法来实现线程间的通信。在线程中调用 wait() 方法，将阻塞等待其他线程的通知（其他线程调用 notify() 方法或 notifyAll() 方法），在线程中调用 notify() 方法或 notifyAll() 方法，将通知其他线程从 wait() 方法处返回。

# wait()
该方法用来将当前线程置入休眠状态，直到接到通知或被中断为止。在调用 wait()之前，线程必须要获得该对象的对象级别锁，即只能在同步方法或同步块中调用 wait()方法。进入 wait()方法后，当前线程释放锁。

# notify()
该方法也要在同步方法或同步块中调用，即在调用前，线程也必须要获得该对象的对象级别锁。
该方法用来通知那些可能等待该对象的对象锁的其他线程。如果有多个线程等待，则线程规划器任意挑选出其中一个 wait()状态的线程来发出通知，并使它等待获取该对象的对象锁（notify 后，当前线程不会马上释放该对象锁，wait 所在的线程并不能马上获取该对象锁，要等到程序退出 synchronized 代码块后，当前线程才会释放锁，wait所在的线程也才可以获取该对象锁），但不惊动其他同样在等待被该对象notify的线程们

# notifyAll()
该方法与 notify ()方法的工作方式相同，重要的一点差异是：
notifyAll 使所有原来在该对象上 wait 的线程统统退出 wait 的状态（即全部被唤醒，不再等待 notify 或 notifyAll，但由于此时还没有获取到该对象锁，因此还不能继续往下执行），变成等待获取该对象上的锁，一旦该对象锁被释放（notifyAll 线程退出调用了 notifyAll 的 synchronized 代码块的时候），他们就会去竞争。如果其中一个线程获得了该对象锁，它就会继续往下执行，在它退出 synchronized 代码块，释放锁后，其他的已经被唤醒的线程将会继续竞争获取该锁，一直进行下去，直到所有被唤醒的线程都执行完毕。

应该在while循环，而不是if语句中调用wait。if语句存在一些微妙的小问题，导致即使条件没被满足，你的线程你也有可能被错误地唤醒。所以如果你不在线程被唤醒后再次使用while循环检查唤醒条件是否被满足，你的程序就有可能会出错。在while循环里使用wait的目的，是在线程被唤醒的前后都持续检查条件是否被满足。如果条件并未改变，wait被调用之前notify的唤醒通知就来了，那么这个线程并不能保证被唤醒，有可能会导致死锁问题。参考《Effective Java》

# 示例
我之前写过一篇文章 <a href="http://blog.csdn.net/revitalizing/article/details/61008956" target="_blank">线程执行顺序——CountDownLatch、join()、线程池</a> 讨论的是让一个线程晚于其他线程最后执行。我想用wait和notify写个例子让一个线程先于其他线程运行。

代码场景：Worker类和Boss类都实现Runnable接口，但是老板先安排完工作后，工人才能开始工作。

代码实现：

```java
public class WaitAndNotify {

    public static void main(String[] args) throws Exception{

        Object object = new Object();

        new Thread(new Worker("work1", object)).start();
        new Thread(new Worker("work2", object)).start();
        new Thread(new Worker("work3", object)).start();

        new Thread(new Boss("boss", object)).start();


    }
}

class Worker implements Runnable{

    private String name;
    private Object object;

    public Worker(String name,Object object){
        this.object = object;
        this.name = name;
    }

    public void run(){
        synchronized(object){
            try {
                object.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name + " is working");
        }
    }
}

class Boss implements Runnable{

    private String name;
    private Object object;

    public Boss(String name, Object object){
        this.name = name;
        this.object = object;
    }

    public void run(){
        synchronized(object){
            System.out.println(name + " has arranged the work");
            object.notifyAll();
            //object.notify();
        }
    }
} 
```

运行结果：

```java
boss has arranged the work
work3 is working
work2 is working
work1 is working
```

从上面的代码中可以看到，Boss类使用notifyAll（）方法，3个worker线程都会执行，如果换成 notify（）方法，则只有一个worker线程会执行。

# 应用
比如可以用wait和notify实现生产者和消费者，这里不细说了，有空再写下生产者和消费者吧。

# 总结
* 可以使用wait和notify函数来实现线程间通信。
* 在synchronized的函数或对象里使用wait、notify和notifyAll，否则Java虚拟机会生成 IllegalMonitorStateException。
* 在while循环里而不是if语句下使用wait。这样，循环会在线程睡眠前后都检查wait的条件，并在条件实际上并未改变的情况下处理唤醒通知。
* 在多线程间共享的对象（在生产者消费者模型里即缓冲区队列）上使用wait。


# 参考资料
<a href="http://wiki.jikexueyuan.com/project/java-concurrency/collaboration-between-threads.html" target="_blank">线程间协作：wait、notify、notifyAll</a>
<a href="http://www.importnew.com/16453.html" target="_blank">如何在 Java 中正确使用 wait, notify 和 notifyAll – 以生产者消费者模型为例</a>



