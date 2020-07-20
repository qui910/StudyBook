# 1 概述

ConcurrentSkipListSet是线程安全的有序的集合，适用于高并发的场景。

ConcurrentSkipListSet和[TreeSet](http://www.cnblogs.com/skywang12345/p/3311268.html)，它们虽然都是有序的集合。但是，第一，它们的线程安全机制不同，TreeSet是非线程安全的，而ConcurrentSkipListSet是线程安全的。第二，ConcurrentSkipListSet是通过[ConcurrentSkipListMap](http://www.cnblogs.com/skywang12345/p/3498556.html)实现的，而TreeSet是通过TreeMap实现的。