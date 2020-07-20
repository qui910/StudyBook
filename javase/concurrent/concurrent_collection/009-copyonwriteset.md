# 1 概述

它是线程安全的无序的集合，可以将它理解成线程安全的[HashSet](http://www.cnblogs.com/skywang12345/p/3311252.html)。有意思的是，CopyOnWriteArraySet和HashSet虽然都继承于共同的父类AbstractSet；但是，HashSet是通过[“散列表](http://www.cnblogs.com/skywang12345/p/3310835.html)(HashMap)”实现的，而CopyOnWriteArraySet则是通过“[动态数组(CopyOnWriteArrayList)](http://www.cnblogs.com/skywang12345/p/3498483.html)”实现的，并不是散列表。

和CopyOnWriteArrayList类似，CopyOnWriteArraySet具有以下特性：

* 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
* 它是线程安全的。
* 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
* 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
* 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。