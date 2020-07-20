# 1 Collection

​		前面我们讲述过Java集合包的一些实现类，这里我们再次总结下，内容包括Collection集合和Map类；而Collection集合又可以划分为List(队列)和Set(集合)。

## 1.1 List

​		**List的实现类主要有: LinkedList, ArrayList, Vector, Stack。**

* LinkedList是双向链表实现的双端队列；它不是线程安全的，只适用于单线程
* ArrayList是数组实现的队列，它是一个动态数组；它也不是线程安全的，只适用于单线程。
* Vector是数组实现的矢量队列，它也一个动态数组；不过和ArrayList不同的是，Vector是线程安全的，它支持并发。
* Stack是Vector实现的栈；和Vector一样，它也是线程安全的。



# 2 并发容器概述

​		Java集合包大多是“非线程安全的”，虽然可以通过Collections工具类中的方法获取java集合包对应的同步类，但是这些同步类的并发效率并不是很高。为了更好的支持高并发任务，并发大师Doug Lea在JUC(java.util.concurrent)包中添加了java集合包中单线程类的对应的支持高并发的类。

​		Java提供多种并发容器类来改进同步容器的性能。并发容器是针对多个线程并发访问设计。

​		并发容器如下：

## 1.1 List和Set

* CopyOnWriteArrayList相当于线程安全的ArrayList，它实现了List接口，用在遍历操作为主的情况下。CopyOnWriteArrayList是支持高并发的。
* CopyOnWriteArraySet相当于线程安全的HashSet，它继承于AbstractSet类。CopyOnWriteArraySet内部包含一个CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。
* ConcurrentSkipListSet是线程安全的有序的集合(相当于线程安全的TreeSet)；它继承于AbstractSet，并实现了NavigableSet接口。ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，它也支持并发。



## 1.2 Map

* ConcurrentHashMap是线程安全的哈希表(相当于线程安全的HashMap)；它继承于AbstractMap类，并且实现ConcurrentMap接口。ConcurrentHashMap是通过“锁分段”来实现的，它支持并发
* ConcurrentSkipListMap是线程安全的有序的哈希表(相当于线程安全的TreeMap); 它继承于AbstractMap类，并且实现ConcurrentNavigableMap接口。ConcurrentSkipListMap是通过“跳表”来实现的，它支持并发。



## 1.3 Queue

* ArrayBlockingQueue是数组实现的线程安全的有界的阻塞队列。
* LinkedBlockingQueue是单向链表实现的(指定大小)阻塞队列，该队列按 FIFO（先进先出）排序元素
* LinkedBlockingDeque是双向链表实现的(指定大小)双向并发阻塞队列，该阻塞队列同时支持FIFO和FILO两种操作方式。
* ConcurrentLinkedQueue是单向链表实现的无界队列，该队列按 FIFO（先进先出）排序元素。
* ConcurrentLinkedDeque是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式。