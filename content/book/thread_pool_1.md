---
title: "Java线程池 - ThreadPoolExecutor"
date: 2019-11-12T14:48:28+08:00
draft: false
categories: [java]
tags: [java,线程池,并发]
---

# 线程池
* java中的线程池是运用场景最多的并发框架。在开发过程中，合理的使用线程池能够带来下面的一些好处：
1. 降低资源的消耗。
2. 提高响应速度。
3. 提高线程的可管理性。

## 线程池ThreadPoolExecutor工作原理
![threadpool.jpg](https://pic1.zhimg.com/80/v2-0c6eec6cd6711e9dd765027d8585f358_hd.jpg)

ThreadPoolExecutor执行execute方法有4种情况：
1）如果当前运行的线程少于corePoolSize，则创建新的线程来执行任务。
2）如果运行的线程等于或者多余corePoolSize，则将任务加入BlockingQueue中，在等待队列中，等待有新的线程可以运行。
3）如果BlockingQueue队列满了，且没有超过maxPoolSize，则创建新的线程来处理任务。
4）如果创建的线程超过maxPoolSize，任务会拒绝，并调用RejectExecutionHandler.rejectedExecution()方法。

## 线程池的使用
### 线程池的创建
一般我们可以通过ThreadPoolExecutor来创建一个线程池。
在ThreadPoolExecutor类中提供了四个构造方法：
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```
通过
```java
new ThreadPoolExecutor(corePoolSize,maximumPoolSize,keepAliveTime,unit,workQueue,threadFactory,handler);
```
创建一个新的线程池。
