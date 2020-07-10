# 1 并发容器概述

​		Java提供多种并发容器类来改进同步容器的性能。并发容器是针对多个线程并发访问设计。

​		并发容器如下：

* ConcurrentHashMap 用来替换同步且给予散列的Map。
* CopyOnWriteArrayList 用来在遍历操作为主要操作的情况下替换同步的List
* CopyOnWriteArraySet	相当于线程安全的HashSet，它是通过CopyOnWriteArrayList实现的。
* ConcurrentLinkedQueue 传统的先进先出队列。
* ConcurrentSkipListMap 作为同步的SortedMap的并发替代。
* ConcurrentSkipListSet 作为同步的SortedSet的并发替代。