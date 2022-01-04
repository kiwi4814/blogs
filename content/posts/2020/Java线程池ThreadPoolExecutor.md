+++
title = "Java线程池ThreadPoolExecutor"
date = 2020-04-27T20:35:47+08:00
draft = false
slug = "/ThreadPoolExecutor"
tags = ["Java"]
categories = ["技术"]
+++

# 一、相关概念

**核心线程（CorePool）：**线程池最终执行任务的角色肯定还是线程，同时我们也会限制线程的数量，所以我们可以这样理解核心线程，有新任务提交时，首先检查核心线程数，如果核心线程都在工作，而且数量也已经达到最大核心线程数，则不会继续新建核心线程，而会将任务放入等待队列。

**等待队列（WorkQueue）：**等待队列用于存储当核心线程都在忙时，继续新增的任务，核心线程在执行完当前任务后，也会去等待队列拉取任务继续执行，这个队列一般是一个线程安全的阻塞队列，它的容量也可以由开发者根据业务来定制。

**非核心线程**：当等待队列满了，如果当前线程数没有超过最大线程数，则会新建线程执行任务，那么核心线程和非核心线程到底有什么区别呢？本质上它们没什么区别。

**线程活动保持时间（keepAliveTime）：**线程空闲下来之后，保持存活的持续时间，超过这个时间还没有任务执行，该工作线程结束。

**饱和策略（RejectedExcutionHandler）：**当等待队列已满，线程数也达到最大线程数时，线程池会根据饱和策略来执行后续操作，默认的策略是抛弃要加入的任务。

**线程池的状态：**

- RUNNING：运行状态，值是最小的，刚创建的线程池就是此状态
- SHUTDOWN：停工状态，不再接受新任务，已经接收的会继续执行
- STOP：停止状态，不再接收新任务，已经接收正在执行的也会中断
- 清空状态：所有任务都停止了，工作的线程也全部都结束了
- TERMINATED：终止状态，线程池已销毁

# 二、源码分析

1. **线程池的线程是如何做到复用的。** 线程池中的线程在循环中尝试取任务执行，这一步会被阻塞，如果设置了allowCoreThreadTimeOut为true，则线程池中的所有线程都会在keepAliveTime时间超时后还未取到任务而退出。或者线程池已经STOP，那么所有线程都会被中断，然后退出。
2. **线程池是如何做到高效并发的。** 看整个线程池的工作流程，有以下几个需要特别关注的并发点. ①: 线程池状态和工作线程数量的变更。这个由一个AtomicInteger变量 ctl来解决原子性问题。 ②: 向工作Worker容器workers中添加新的Worker的时候。这个线程池本身已经加锁了。 ③: 工作线程Worker从等待队列中取任务的时候。这个由工作队列本身来保证线程安全，比如LinkedBlockingQueue等。

# 三、Executors的使用

JDK已经给我们提供了很方便的线程池工厂类Executors, 方便我们快速创建线程池，可能在阅读源码之前，我们在面对具体的业务场景时，到底该选择哪种线程池配置是有疑问的，我们来看一下。

```java
		public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

newFixedThreadPool, 可以看到我们需要传入一个线程数量的参数nThreads，这样线程池的核心线程数和最大线程数都会设成nThreads, 而它的等待队列是一个LinkedBlockingQueue，它的容量限制是Integer.MAX_VALUE, 可以认为是没有边界的。核心线程keepAlive时间0，allowCoreThreadTimeOut默认false。所以这个方法创建的线程池适合能估算出需要多少核心线程数量的场景。

```java
		public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

newSingleThreadExecutor, 有且只有一个线程在工作，适合任务顺序执行，缺点但是不能充分利用CPU多核性能。

```java
		public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

newCachedThreadPool, 核心线程数0，最大线程数Integer.MAX_VALUE, 线程keepAlive时间60s，用的队列是SynchronousQueue，这种队列本身不会存任务，只做转发，所以newCachedThreadPool适合执行大量的，轻量级任务。

```java
		public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

newScheduledThreadPool, 执行周期性任务，类似定时器。