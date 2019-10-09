---
title: 线程执行顺序——CountDownLatch、CyclicBarrier 、join()、线程池
date: 2017-01-25 15:10:14
categories: Java
tags: Java
---

本文主要围绕一个问题展开：线程执行顺序，比如某个线程在其他线程并发执行完毕后最后执行，分别用CountDownLatch、CyclicBarrier 、join()、线程池 来实现。

# CyclicBarrier

CyclicBarrier 的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它可以让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。线程进入屏障通过CyclicBarrier的await()方法。

CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。
CyclicBarrier还提供一个构造函数CyclicBarrier(int parties, Runnable barrierAction)，用于在线程到达屏障时，优先执行barrierAction这个Runnable对象，方便处理更复杂的业务场景。

## 实现原理

在CyclicBarrier的内部定义了一个Lock对象，每当一个线程调用CyclicBarrier的await方法时，将剩余拦截的线程数减1，然后判断剩余拦截数是否为0，如果不是，进入Lock对象的条件队列等待。如果是则执行barrierAction对象的Runnable方法，然后将锁的条件队列中的所有线程放入锁等待队列中，这些线程会依次的获取锁、释放锁，接着先从await方法返回，再从CyclicBarrier的await方法中返回。

## 使用场景

CyclicBarrier主要用于一组线程之间的相互等待，而CountDownLatch一般用于一组线程等待另一组些线程。实际上可以通过CountDownLatch的countDown()和await()来实现CyclicBarrier的功能。即 CountDownLatch中的countDown()+await() = CyclicBarrier中的await()。注意：在一个线程中先调用countDown()，然后调用await()。

## 示例

```java
public class CyclicBarrierPractice {

    static class Worker implements Runnable{
        private String name;
        private CyclicBarrier cyclicBarrier;
        public Worker(String name, CyclicBarrier cyclicBarrier){
            this.name = name;
            this.cyclicBarrier = cyclicBarrier;
        }

        public void run(){
            System.out.println(name + " is working");
            try {
                Thread.sleep(1000);
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }

    static class Boss implements Runnable{
        private String name;

        public Boss(String name){
            this.name = name;
        }

        public void run(){
            System.out.println(name + " checking work");

        }
    }

    public static void main(String[] args){
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Boss("boss"));
        for(int i=0; i<3; i++){
            new Thread(new Worker("worker"+i, cyclicBarrier)).start();
        }
    }
}

```

## 运行结果

```java
worker0 is working
worker1 is working
worker2 is working
boss checking work
```

# join
join()是Thread类的一个方法，join()方法的作用是等待这个线程结束。t.join()方法阻塞调用此方法的线程(calling thread)，直到线程t完成，此线程再继续；通常用于在main()主线程内，等待其它线程完成再结束main()主线程。

## join实现
Join方法实现是通过wait（Object 提供的方法）。 比如当main线程调用t.join时候，main线程会获得线程对象t的锁（wait 意味着拿到该对象的锁),调用该对象的wait(等待时间)，直到该对象唤醒main线程，比如退出后。这就意味着main 线程调用t.join时，必须能够拿到线程t对象的锁。

## 示例
用join方式实现问题如下，在代码中main线程被阻塞直到 thread1，thread2执行完，主线程才会顺序的执行thread3.

```java

public class OrderThreadExecute {

	class OrderThread implements Runnable{
        private String name;

        public OrderThread(String name){
            this.name = name;
        }

        public void run(){
            System.out.println(name + " is working");
        }
    }

    public static void main(String[] args) {
        
        //使用join方法顺序执行
        OrderThread worker1 = orderThread.new OrderThread("worker1");
        OrderThread worker2 = orderThread.new OrderThread("worker2");
        OrderThread boss = orderThread.new OrderThread("boss");

        Thread thread1 = new Thread(worker1);
        Thread thread2 = new Thread(worker2);
        Thread thread3 = new Thread(boss);

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread3.start();

    }

}

```

# CountDownLatch
Java的util.concurrent包里面的CountDownLatch其实可以把它看作一个计数器（倒计时锁），只不过这个计数器的操作是原子操作，同时只能有一个线程去操作这个计数器，也就是同时只能有一个线程去减这个计数器里面的值。

你可以向CountDownLatch对象设置一个初始的数字作为计数值，任何调用这个对象上的await()方法都会阻塞，直到这个计数器的计数值被其他的线程减为0为止。

## 使用场景

CountDownLatch的一个非常典型的应用场景是：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。假如我们这个想要继续往下执行的任务调用一个CountDownLatch对象的await()方法，其他的任务执行完自己的任务后调用同一个CountDownLatch对象上的countDown()方法，这个调用await()方法的任务将一直阻塞等待，直到这个CountDownLatch对象的计数值减到0为止。

## 实例

举个例子，有三个工人在为老板干活，这个老板有一个习惯，就是当三个工人把一天的活都干完了的时候，他就来检查所有工人所干的活。记住这个条件：三个工人先全部干完活，老板才检查。所以在这里用Java代码设计两个类，Worker代表工人，Boss代表老板，代码使用了内部类实现。

```java

public class OrderThreadExecute {

    class Worker implements Runnable {
        private CountDownLatch downLatch;
        private String name;

        public Worker(CountDownLatch downLatch, String name) {
            this.downLatch = downLatch;
            this.name = name;
        }

        @Override
        public void run() {
            this.doWork();
            try {
                TimeUnit.SECONDS.sleep(new Random().nextInt(10));
            } catch (InterruptedException ie) {
            }
            System.out.println(this.name + "活干完了！");
            this.downLatch.countDown();
        }

        private void doWork() {
            System.out.println(this.name + "正在干活...");
        }

    }

    class Boss implements Runnable {
        private CountDownLatch downLatch;

        public Boss(CountDownLatch downLatch) {
            this.downLatch = downLatch;
        }

        @Override
        public void run() {
            System.out.println("老板正在等所有的工人干完活......");
            try {
                this.downLatch.await();
            } catch (InterruptedException e) {
            }
            System.out.println("工人活都干完了，老板开始检查了！");
        }

    }

    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        CountDownLatch latch = new CountDownLatch(3);

        OrderThreadExecute orderThread = new OrderThreadExecute();

        Worker w1 = orderThread.new Worker(latch, "张三");
        Worker w2 = orderThread.new Worker(latch, "李四");
        Worker w3 = orderThread.new Worker(latch, "王二");

        Boss boss = orderThread.new Boss(latch);

        executor.execute(boss);
        executor.execute(w3);
        executor.execute(w2);
        executor.execute(w1);

        executor.shutdown();

    }

}

```

## CountDownLatch与join比较

调用thread.join() 方法必须等thread 执行完毕，当前线程才能继续往下执行，而CountDownLatch通过计数器提供了更灵活的控制，只要检测到计数器为0当前线程就可以往下执行而不用管相应的thread是否执行完毕。

>具体比较见文章：http://blog.csdn.net/nyistzp/article/details/51444487

# 使用线程池

当线程池的线程全部执行完毕后执行，勉强也算吧，示例代码如下。

```java
public class ExecuteOrderPractice {

    public void orderPractice(){
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for(int i = 0; i < 5; i++){
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try{
                        Thread.sleep(1000);
                        System.out.println(Thread.currentThread().getName() + " do something");
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            });
        }

        executorService.shutdown();

        while(true){
            if(executorService.isTerminated()){
                System.out.println("Finally do something ");
                break;
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        new ExecuteOrderPractice().orderPractice();

    }
}

```

# 引申
如何每个线程都顺序执行，听起来好像为什么还要用多线程呢，有空再看吧


# 参考资料
<a href="http://blog.csdn.net/nyistzp/article/details/51444487" target="_blank">java 多线程 CountDownLatch与join()方法区别</a>
<a href="http://www.aichengxu.com/java/2129819.htm" target="_blank">CountDownLatch使用实例</a>
<a href="http://blog.csdn.net/truong/article/details/40227435" target="_blank">Java如何判断线程池所有任务是否执行完毕</a>
   

