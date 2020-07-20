# 1 概述

​		ArrayDeque类是双端队列的实现类，类的继承结构如下面，继承自AbastractCollection（该类实习了部分集合通用的方法，其实现了Collection接口），其实现的接口Deque接口中定义了双端队列的主要的方法，比如从头删除，从尾部删除，获取头数据，获取尾部数据等等。

```java
public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable
```

## 1.1 ArrayDeque基本特征
​		就其实现而言，ArrayDeque采用了循环数组的方式来完成双端队列的功能。

1. 无限的扩展，自动扩展队列大小的。（当然在不会内存溢出的情况下。）
2. 非线程安全的，不支持并发访问和修改。
3. 支持fast-fail.
4. 作为栈使用的话比比栈要快.
5. 当队列使用比linklist要快。
6. null元素被禁止使用。

