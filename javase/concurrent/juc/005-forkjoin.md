# 1 什么是ForkJoinPool
ForkJoinPool是JDK7引入的线程池，核心思想是将大的任务拆分成多个小任务（即fork），然后在将多个小任务处理汇总到一个结果上（即join），非常像MapReduce处理原理。同时，它提供基本的线程池功能，支持设置最大并发线程数，支持任务排队，支持线程池停止，支持线程池使用情况监控，也是AbstractExecutorService的子类，主要引入了“工作窃取”机制，在多CPU计算机上处理性能更佳。

​		![thread-029](..\..\images\thread-029.png)

## 1.1 work-stealing（工作窃取算法）

work-stealing（工作窃取），ForkJoinPool提供了一个更有效的利用线程的机制，当ThreadPoolExecutor还在用单个队列存放任务时，ForkJoinPool已经分配了与线程数相等的队列，当有任务加入线程池时，会被平均分配到对应的队列上，各线程进行正常工作，当有线程提前完成时，会从队列的末端“窃取”其他线程未执行完的任务，当任务量特别大时，CPU多的计算机会表现出更好的性能。

# 2 常用方法

## 2.1 ForkJoinTask

我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：

```java
RecursiveAction：用于没有返回结果的任务。
RecursiveTask ：用于有返回结果的任务。
```

## 2.2 ForkJoinPool

ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。
　

## 2.3 线程池监控
​	在线程池使用监控方面，主要通过如下方法：
　　isTerminated—判断线程池对应的workQueue中是否有待执行任务未执行完；
　　awaitTermination—判断线程池是否在约定时间内完成，并返回完成状态；
　　getQueuedSubmissionCount—获取所有待执行的任务数；
　　getRunningThreadCount—获取正在运行的任务数。

https://blog.csdn.net/niyuelin1990/article/details/78658251

https://blog.csdn.net/qq_33369979/article/details/87554719