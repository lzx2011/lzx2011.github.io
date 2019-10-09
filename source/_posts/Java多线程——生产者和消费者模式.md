---
title: Java多线程——生产者和消费者模式
date: 2016-08-15 22:23:14
categories: Java
tags: Java多线程
---
生产者和消费者模式是一种并发设计模式，生产者消费者模式解决的是两者速率不一致而产生的阻抗不匹配，该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度。

# 生产者消费者模式

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

# 为什么要使用生产者和消费者模式

在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。


# 优点

* 可以独立地同时编码生产者和消费者，他们只需要知道共享对象即可。

* 生产者不需要知道谁是消费者或有多少消费者，消费者也是如此。

* 生产者和消费者可以以不同的速度工作，消费者没有消费半成品的风险。

* 分离生产者和消费者的功能导致更干净，可读和易于管理的代码。


# 应用

Executor框架本身也实现了生产者和消费者模式，在线程池中，如果任务数多于基本线程数时，会将任务放到阻塞队列中来平衡生产者和消费者的处理能力，关于线程池的介绍可以看我的另一篇文章 <a href="http://blog.csdn.net/revitalizing/article/details/61671858" target="_blank">java——线程池</a>

# 示例代码

## 用阻塞队列实现

先用阻塞队列来实现，BlockingQueue 是个继承Queue接口的接口，该接口有不同的实现，比如ArrayBlockingQueue 和 LinkedBlockingQueue，他们都实现了 FIFO。

用LinkedBlockingQueue实现生产者和消费者模式如下。

```java
public class ProducerConsumerPractice {

    public static void main(String[] args){

        LinkedBlockingDeque<Integer> linkedBlockingDeque = new LinkedBlockingDeque<>(5);
        new Thread(new Producer(linkedBlockingDeque)).start();
        new Thread(new Consumer(linkedBlockingDeque)).start();
    }
}

class Producer implements Runnable{

    private LinkedBlockingDeque<Integer> linkedBlockingDeque;

    public Producer(LinkedBlockingDeque<Integer> linkedBlockingDeque){
        this.linkedBlockingDeque = linkedBlockingDeque;
    }

    public void run(){
        for(int i = 0; i < 10; i++){
            try {
                //Thread.sleep(500);
                linkedBlockingDeque.put(i);
                System.out.println("Producer: " + i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

class Consumer implements Runnable{

    private LinkedBlockingDeque<Integer> linkedBlockingDeque;

    public Consumer(LinkedBlockingDeque<Integer> linkedBlockingDeque){
        this.linkedBlockingDeque = linkedBlockingDeque;
    }

    public void run(){
        while(true){
            try{
                Thread.sleep(500);
                System.out.println("consumer: " + linkedBlockingDeque.take());
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }
}
```
运行结果：

```java
Producer: 0
Producer: 1
Producer: 2
Producer: 3
Producer: 4
consumer: 0
Producer: 5
consumer: 1
Producer: 6
consumer: 2
Producer: 7
consumer: 3
Producer: 8
consumer: 4
Producer: 9
consumer: 5
consumer: 6
consumer: 7
consumer: 8
consumer: 9
```
我设置了阻塞队列的初始长度为5，然后用sleep（500）调慢了消费速度，所以我们在运行结果中可以看到生产0-4后，队列满了，生产者被阻塞了，然后消费者根据FIFO原则先消费了0，所以生产者又可以继续生产了。在ide中运行看的会更清楚些，第二种方式实现打印的结果会更明白。

## 用wait(), notify() 实现

之前写过一篇文章 <a href="http://blog.csdn.net/revitalizing/article/details/61963579" target="_blank">线程间协作——wait、notify、notifyAll</a> 讲了 wait(), notify（），notifyAll()的用法，现在用他们来实现生产者和消费者模式，当做补充例子吧。这里用 Vector 模拟队列，因为这个队列没有阻塞功能，所以要用wait()和 notify（）模拟队列满时生产者和队列为空时消费者的阻塞，以及正常情况下互相通知对方的效果。

代码中同样调慢了消费速度，为了看的更清晰。

```java
public class ProducerConsumerPractice {

    public static void main(String[] args){

        Vector<Integer> vector = new Vector<>(5);
        new Thread(new Producer(vector)).start();
        new Thread(new Consumer(vector)).start();
    }
}


class Producer implements Runnable{

    private Vector<Integer> vector;

    public Producer(Vector vector){
        this.vector = vector;
    }

    public void run(){
        for(int i = 0; i < 10; i++){
            while(vector.size() == vector.capacity()){
                synchronized (vector){
                    System.out.println("Queue is full, Producer  is waiting , size: " + vector.size());
                    try {
                        vector.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

            synchronized (vector){
                vector.add(i);
                System.out.println("Producer: " + i);
                vector.notifyAll();
            }
        }
    }
}

class Consumer implements Runnable{

    private Vector<Integer> vector;

    public Consumer(Vector vector){
        this.vector = vector;
    }

    public void run(){
        while(true){
            while(vector.isEmpty()){
                synchronized (vector){
                    System.out.println("Queue is empty, Consumer is waiting , size: " + vector.size());
                    try {
                        vector.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

            }
            //调慢消费速度
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (vector){
                System.out.println("Consumer: " + vector.remove(0));
                vector.notifyAll();
            }
        }
    }

}

```

运行结果：

```java
Producer: 0
Producer: 1
Producer: 2
Producer: 3
Producer: 4
Queue is full, Producer  is waiting , size: 5
Consumer: 0
Producer: 5
Queue is full, Producer  is waiting , size: 5
Consumer: 1
Producer: 6
Queue is full, Producer  is waiting , size: 5
Consumer: 2
Producer: 7
Queue is full, Producer  is waiting , size: 5
Consumer: 3
Producer: 8
Queue is full, Producer  is waiting , size: 5
Consumer: 4
Producer: 9
Consumer: 5
Consumer: 6
Consumer: 7
Consumer: 8
Consumer: 9
Queue is empty, Consumer is waiting , size: 0
```

# 参考资料
<a href="http://www.infoq.com/cn/articles/producers-and-consumers-mode" target="_blank">聊聊并发——生产者消费者模式</a>
<a href="http://www.java67.com/2012/12/producer-consumer-problem-with-wait-and-notify-example.html#ixzz4bbane200" target="_blank">Producer Consumer Problem with Wait and Notify Example</a>


