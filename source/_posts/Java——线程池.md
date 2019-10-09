---
title: Java——线程池
date: 2016-11-12 19:20:14
categories: Java
tags: Java
---
# Java 线程池
# 线程池
线程池负责管理工作线程，包含一个等待执行的任务队列。
线程池的任务队列是一个 Runnable集合，工作线程负责从任务队列中取出并执行Runnable对象。Executor 框架便是 Java 5 中引入的，其内部使用了线程池机制，Executor 框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable 等。
# 单线程的弊端

* 每次new Thread新建对象性能差。
* 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或 oom。
* 缺乏更多功能，如定时执行、定期执行、线程中断。

> 不控制线程数量，不断创建新线程，很快会导致oom，线程还是很占用资源的，线程栈的大小，JDK5.0以后每个线程堆栈大小默认为1M,以前每个线程堆栈大小为256K；可以通过jvm参数-Xss来设置；注意-Xss是jvm的非标准参数，不强制所有平台的jvm都支持。

# 线程池的优点

1. 降低资源消耗。使用线程池的好处是重用存在的线程，减少在创建和销毁线程上所花的时间以及系统资源的开销，如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者 “过度切换”的问题。 

2. 提高响应速度。重用存在的线程，任务可以不需要等到线程创建就能立即执行。

3. 提高线程的可管理性。提供定时执行、定期执行、单线程、并发数控制等功能，线程池可以进行统一的分配，调优和监控。

# 线程池分类

Executors 提供了一系列工厂方法用于创先线程池，返回的线程池都实现了 ExecutorService 接口。

* newCachedThreadPool:创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。缓存型池子通常用于执行一些生存期很短的异步型任务 因此在一些面向连接的 daemon 型 SERVER 中用得不多。但对于生存期短的异步任务，它是 Executor 的首选。不限制线程数，可能会导致oom。
 
* newFixedThreadPool: 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
 
* newScheduledThreadPool: 创建一个定长线程池，支持定时及周期性任务执行，多数情况下可用来替代Timer类。

* newSingleThreadExecutor: 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

# 核心类ThreadPoolExecutor

`java.uitl.concurrent.ThreadPoolExecutor`类是线程池中最核心的一个类，有四个构造方法，拿一个构造方法举例。

```java
 public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
```

* corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

* keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

* workQueue：一个阻塞队列，用来存储等待执行的任务。ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和Synchronous。线程池的排队策略与BlockingQueue有关。

* threadFactory：线程工厂，主要用来创建线程；

* handler：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。有以下四种取值：
 
```java
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常(默认采取的策略)。 
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```

# 线程池执行任务方法

## execute()
方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。但是execute方法没有返回值，所以无法判断任务是否被线程池执行成功。

## submit()
方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，submit()执行 Callable 任务，会发现它实际上还是调用的execute()方法，利用了Future来获取任务执行结果。（代码示例参见：http://wiki.jikexueyuan.com/project/java-concurrency/executor.html）

# 线程池关闭

我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。

shutdown()方法使线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；

shutdownNow()方法使线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；

当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

> 通过类ThreadPoolExecutor线程执行的源码分析可以参考文章：http://www.cnblogs.com/dolphin0520/p/3932921.html


# 线程池监控

ThreadPoolExecutor 提供了一些方法，可以查看执行状态、线程池大小、活动线程数和任务数。

```java
 getPoolSize()  线程池的线程数量。
 getCorePoolSize() 线程池基本线程数。
 getActiveCount() 活跃线程数
 getCompletedTaskCount() 获取完成任务数
 getTaskCount()   计划要执行任务数，不一定准确
 isShutdown()  线程池的状态是否是shutdown
 isTerminated()  所有任务是否都执行完毕
```

通过扩展线程池进行监控。通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。

# 线程池执行任务流程

* 如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建一个线程去执行这个任务；

* 如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；
 
* 如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；
 
* 如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被终止。

# 如何合理配置线程池的大小

获取cpu核数：`Runtime.getRuntime().availableProcessors()`;

一般需要根据任务的类型来配置线程池大小：

如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1
如果是IO密集型任务，参考值可以设置为2*NCPU

当然，这只是一个参考值，具体的设置还需要根据实际情况进行调整，比如可以先将线程池大小设置为参考值，再观察任务运行情况和系统负载、资源利用率来进行适当调整。

# 可能出现的问题

下面这段引用来自阿里的java开发规范

> 线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样 的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。Executors 返回的线程池对象的弊端如下,FixedThreadPool和SingleThreadPool允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。CachedThreadPool和ScheduledThreadPool允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。

# 参考资料
<a href="http://stormzhang.com/java/2013/11/08/java-thread-pool/" target="_blank">JAVA THREAD POOL</a>
<a href="http://www.cnblogs.com/dolphin0520/p/3932921.html" target="_blank">Java并发编程：线程池的使用（很细致的好文章）</a>
<a href="http://www.infoq.com/cn/articles/java-threadPool" target="_blank">聊聊并发（三）——JAVA线程池的分析和使用</a>





