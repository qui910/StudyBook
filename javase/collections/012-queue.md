# 1 概述

​		队列是一种先进先出（FIFO）的数据结构，队列接口直接继承`Collection`。不支持在队列中间添加或删除元素。

​		其类图如下：

![011-1](..\images\011-1.png)



# 2 常用API

```java
 // 如果队列没有满，将给定的元素添加到这个队列的尾部并返回 true。如果队列满了，第一个方法将拋出一个 IllegalStateException, 而第二个方法返回 false。 
boolean add(E element )
boolean offer(E element )
// 假如队列不空，删除并返回这个队列头部的元素。如果队列是空的，第一个方法抛出NoSuchElementException, 而第二个方法返回 null。
E remove( )
E poll ( )
// 如果队列不空，返回这个队列头部的元素， 但不删除。如果队列空，第一个方法将拋出一个 NoSuchElementException, 而第二个方法返回 null。
E element()
E peek()
```

# 3 AbstractQueue

​		AbstractQueue定义如下：

```java
public abstract class AbstractQueue<E> extends AbstractCollection<E>
    implements Queue<E> 
```

​		AbstractQueue是一个继承于AbstractCollection，并且实现Queue接口的抽象类。AbstractQueue未实现`BlockingQueue`,是非阻塞的。

# 4 队列分类

## 4.1 非阻塞

​		`PriorityQueue` 和 `ConcurrentLinkedQueue` 类在 Collection Framework 中加入两个具体集合实现。 

​		PriorityQueue 类实质上维护了一个有序列表。加入到 Queue 中的元素根据它们的天然排序（通过其 java.util.Comparable 实现）或者根据传递给构造函数的 java.util.Comparator 实现来定位。

​		ConcurrentLinkedQueue 是基于链接节点的、线程安全的队列。并发访问不需要同步。因为它在队列的尾部添加元素并从头部删除它们，所以只要不需要知道队列的大小。		

​		ConcurrentLinkedQueue 对公共集合的共享访问就可以工作得很好。收集关于队列大小的信息会很慢，需要遍历队列。



## 4.2 阻塞

​		`java.util.concurrent` 中加入了 BlockingQueue 接口和五个阻塞队列类。它实质上就是一种带有一点扭曲的 FIFO 数据结构。不是立即从队列中添加或者删除元素，线程执行操作阻塞，直到有空间或者元素可用。

​		五个队列所提供的各有不同：
　　* ArrayBlockingQueue ：一个由数组支持的有界队列。
　　* LinkedBlockingQueue ：一个由链接节点支持的可选有界队列。
　　* PriorityBlockingQueue ：一个由优先级堆支持的无界优先级队列。
　　* DelayQueue ：一个由优先级堆支持的、基于时间的调度队列。
　　* SynchronousQueue ：一个利用 BlockingQueue 接口的简单聚集（rendezvous）机制。

**LinkedBlockingQueue**的容量是没有上限的（说的不准确，在不指定时容量为Integer.MAX_VALUE，不要然的话在put时怎么会受阻呢），但是也可以选择指定其最大容量，它是基于链表的队列，此队列按 FIFO（先进先出）排序元素。


**ArrayBlockingQueue**在构造时需要指定容量， 并可以选择是否需要公平性，如果公平参数被设置true，等待时间最长的线程会优先得到处理（其实就是通过将ReentrantLock设置为true来 达到这种公平性的：即等待时间最长的线程会先操作）。通常，公平性会使你在性能上付出代价，只有在的确非常需要的时候再使用它。它是基于数组的阻塞循环队 列，此队列按 FIFO（先进先出）原则对元素进行排序。


**PriorityBlockingQueue**是一个带优先级的 队列，而不是先进先出队列。元素按优先级顺序被移除，该队列也没有上限（看了一下源码，PriorityBlockingQueue是对 PriorityQueue的再次包装，是基于堆数据结构的，而PriorityQueue是没有容量限制的，与ArrayList一样，所以在优先阻塞 队列上put时是不会受阻的。虽然此队列逻辑上是无界的，但是由于资源被耗尽，所以试图执行添加操作可能会导致 OutOfMemoryError），但是如果队列为空，那么取元素的操作take就会阻塞，所以它的检索操作take是受阻的。另外，往入该队列中的元 素要具有比较能力。


**DelayQueue**（基于PriorityQueue来实现的）是一个存放Delayed 元素的无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且poll将返回null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于或等于零的值时，则出现期满，poll就以移除这个元素了。此队列不允许使用 null 元素。



 ## 4.3 双端队列

​		下节介绍。

