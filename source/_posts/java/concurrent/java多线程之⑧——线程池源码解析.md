title: java多线程之⑧——线程池源码解析
date: 2017-12-25
categories:
  - java多线程
tags:
  - 多线程
  - 并发
  - 线程池
  
---

Java中我们使用`new Thread()`就能很轻松的创建并使用一个线程，但是创建并销毁一个线程是一个非常耗费计算机资源和时间的事情，如果同一时间线程并发的数量非常多，并且每个线程都执行一个非常短的任务就结束，这样会大大降低程序的执行效率。

这个时候线程池的概念就产生了，他使得线程可以复用，再执行完一个任务之后，不直接销毁，进而后续可以用他来执行后续的其他任务。

java中，`ThreadPoolExecutor`类是实现线程池的核心类。我们通过分析源码来了解java线程池的运行机制。


# 源码解析

## 构造函数

首先看一下`ThreadPoolExecutor`的构造方法：

````
public class ThreadPoolExecutor extends AbstractExecutorService {

	    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
	            BlockingQueue<Runnable> workQueue);
	 
	    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
	            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
	 
	    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
	            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
	 
	    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
	        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
````
`ThreadPoolExecutor`通过了四个构造方法，翻看源码可以看到，前三个构造方法都是调用的最后一个7参数的构造方法，我们看一下几个参数的具体含义：
*  `corePoolSize`：核心池的大小。在创建线程池之后，提交一个任务到线程池的时候，核心线程池中并没有任何线程创建（当然，你可以调用prestartAllCoreThreads提前创建所有核心线程），核心池用来存储用来执行任务的线程，当并发进来的线程达到corePoolSize之后，后面进来的线程将进入缓存队列中；
* `maximumPoolSize`：线程池中线程的最大数量包含上面讲过的corePoolSize，如果缓存队列满了，并且已创建的线程数小于最大线程数，则线程池会继续创建新的线程执行任务；
* `keepAliveTime`：线程的超时时间，表示线程没有任务执行时，超过一定时间将要终止。默认情况下，这个时间只针对于超过了核心线程数量`corePoolSize`的线程；如果设置了`allowCoreThreadTimeOut(boolean)`方法，则也对核心池的线程起作用，直到线程池中的线程数量变为0；
* `unit`：keepAliveTime的时间单位；
* `workQueue`：阻塞队列，用来存储等待中需要执行的任务，可以选择以下几种阻塞队列：
  1.  `ArrayBlockingQueue`：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。
  2. `LinkedBlockingQueue`：一个基于链表结构的阻塞队列，此队列按FIFO排序元素。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
  3. `SynchronousQueue`：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法`Executors.newCachedThreadPool`使用了这个队列；
* `threadFactory`：线程工厂，主要用来创建线程；
* `handler`：饱和策略。当队列和线程吃都满了，即线程池处于饱和状态，需要有一种策略来处理新提交的任务。默认情况下是抛出异常（AbortPolicy策略）。可以用策略如下：
  1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
  2. ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
  3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
  4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务;











