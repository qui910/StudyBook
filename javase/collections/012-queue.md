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

​		`PriorityQueue` 和 `ConcurrentLinkedQueue` 类在 `Collection Framework`中加入两个具体集合实现。 

### 4.1.1 PriorityQueue

​		PriorityQueue 类实质上维护了一个有序列表。加入到 Queue 中的元素根据它们的天然排序（通过其 java.util.Comparable 实现）或者根据传递给构造函数的 java.util.Comparator 实现来定位。

​		在任何时候调用`remove()`，总会删除并获得当前优先级队列中的最小元素。可以理解 PriorityQueue对于队列的意义来说，和TreeSet对于列表的意义一样。是在普通队列外，提供一个可以排序的队列。

​		优先级队列使用了一个优雅且高效的数据结构，称为堆（heap)，小顶堆。堆是一个可以自我调整的二叉树，对树执行添加 （add) 和删除（remore) 操作， 可以让最小的元素移动到根，而不必花费时间对元素进行排序。

​		与 TreeSet—样，一个优先级队列既可以保存实现了 Comparable 接口的类对象， 也可以保存在构造器中提供的 Comparator 对象。   

#### 4.1.1.1 源码解析

```java
//从插入最后一个元素的父节点位置开始建堆
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}

private void siftDownUsingComparator(int k, E x) {
    // 计算非叶子节点元素的最大位置
    int half = size >>> 1;
    // 如果不是叶子节点则一直循环
    while (k < half) {
    	//首先找到左右孩子中较小的那个，记录到c里，并用child记录其下标
        int child = (k << 1) + 1;//leftNo = parentNo*2+1
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;//然后用c取代原来的值
        k = child;
    }
    queue[k] = x;
}

//新加入的元素可能会破坏小顶堆的性质，因此需要进行必要的调整。
public boolean offer(E e) {
    if (e == null)//不允许放入null元素
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);//自动扩容
    size = i + 1;
    if (i == 0)//队列原来为空，这是插入的第一个元素
        queue[0] = e;
    else
        siftUp(i, e);//调整
    return true;
}

// 用于插入元素x并维持堆的特性。
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;//parentNo = (nodeNo-1)/2
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)//调用比较器的比较方法
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

完全二叉树的父子节点编号的取值，`leftNo = parentNo*2+1`，`rightNo = parentNo*2+2`，`parentNo = (nodeNo-1)/2`，而且
在整个二叉树中非叶子节点的个数为`size>>1`。

**参考资料：**

[深入理解Java PriorityQueue](https://www.cnblogs.com/CarpenterLee/p/5488070.html)

[深入Java集合系列之五：PriorityQueue](https://blog.csdn.net/u011116672/article/details/50997622)

#### 4.1.1.2 应用场景

​		使用优先级队列的典型示例是任务调度。每一个任务有一个优先级，任务以随机顺序添加到队列中。每当启动一个新的任务时，都将优先级最高的任务从队列中删除（由于习惯上将 1 设为“ 最高” 优先级，所以会将最小的元素删除 )。 

​		可以应用到获取TopN等场景：[PriorityQueue的实际应用场景](https://blog.csdn.net/a909301740/article/details/104183769/)

​		比方说我们有一个每日交易时段生成股票报告的应用程序，需要处理大量数据并且花费很多处理时间。客户向这个应用程序发送请求时，实际上就进入了队列。我们需要首先处理优先客户再处理普通用户。

​		比如一个电商网站搞特卖或抢购，用户登录下单提交后，考虑这个时间段用户访问下单提交量很大，通常表单提交到服务器后端后，后端程序一般不直接进行扣库存处理，将请求放到队列列，异步消费处理，用普通队列是FIFO的，这里有个需求是，用户会员级别高的，可以优先抢购到商品，可能这个时间段的级别较高的会员用户下单时间在普通用户之后，这个时候使用优先队列代替普通队列，基本能满足我们的需求。



#### 4.1.1.3 并发问题

* 10000个线程同时写入队列，并发情况下会造成数据丢失。在多线程情况下，可以考虑使用`PriorityBlockingQueue`
* 并发场景下也会出现fast-fast事件



### 4.1.2 ConcurrentLinkedQueue 

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

