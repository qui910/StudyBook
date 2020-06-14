# 1 概述

​		List是一个元素**有序**的、可以**重复**、可以为**null**的集合。

​		List的数据结构就是一个序列，存储内容时直接在内存中开辟一块**连续的空间**，然后将空间地址与索引对应。

​		Java集合框架中最常使用的几种List实现类是 ArrayList, LinkedList, Vector和Stack。其架构图如下:

![006-1](..\images\006-1.png)

​		AbstractList 是一个抽象类，它继承于AbstractCollection。AbstractList实现List接口中除size()、get(int location)之外的函数。

​		AbstractSequentialList 是一个抽象类，它继承于AbstractList。AbstractSequentialList 实现了“链表中，根据index索引值操作链表的全部函数”。

​		ArrayList 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低。

​		LinkedList 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率低。

​		Vector 是矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现。但是ArrayList是非线程安全的，而Vector是线程安全的。

​		Stack 是栈，它继承于Vector。它的特性是：先进后出(FILO, First In Last Out)。

## 1.1 常用API

​		List从Collection继承了部分方法,并新增部分新的操作方法。

```java
//返回列表中指定位置的元素
E get(int index);	
//替换列表中指定位置的元素
set(int index, E element);
//在列表的指定位置插入指定元素
void add(int index, E element);		
//将指定 collection 中的所有元素都插入到列表中的指定位置
boolean addAll(int index, Collection<? extends E> c);	
//移除列表中指定位置的元素
E remove(int index);	
//返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1
int indexOf(Object o);	
//返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1
int lastIndexOf(Object o);	
//返回此列表元素的列表迭代器（按适当顺序）。
ListIterator<E> listIterator();	
//返回列表中元素的列表迭代器（按适当顺序），从列表的指定位置开始。
ListIterator<E> listIterator(int index);	
//返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的部分视图
List<E> subList(int fromIndex, int toIndex);	
```

* `List.subList(int fromIndex, int toIndex)` 方法返回 List 在 fromIndex 与 toIndex 范围内的子集。注意是**左闭右开**，[fromIndex,toIndex)。subList返回的仍是List原来的引用，只不过设置了开始位置offset和结束位置size，添加和删除元素都在这个范围操作。
* 由于subList持有List同一个引用，所以对subList进行的操作也会影响到原有List。并且在subList的set,get,add等方法中也会判断modCount，如果不一致会产生fast-fast事件。

## 1.2 List使用场景

> 如果涉及到“栈”、“队列”、“链表”等操作，应该考虑用List，具体的选择哪个List，根据下面的标准来取舍。

1. 对于需要快速插入，删除元素，应该使用LinkedList。
2. 对于需要快速随机访问元素，应该使用ArrayList。
3. 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList)。对于“多线程环境，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector)。考虑到Vector是支持同步的，而Stack又是继承于Vector的。

注意：对于Vector，多线程访问时也是会出现线程不安全问题，具体看Vector描述。

## 1.3 LinkedList和ArrayList性能差异分析

* LinkedList中插入元素很快，而ArrayList中插入元素很慢。“删除元素”与“插入元素”的原理类似
* LinkedList中随机访问很慢，而ArrayList中随机访问很快。

## 1.4 Vector和ArrayList比较

### 1.4.1 相同之处

* 都是List,它们都继承于AbstractList，并且实现List接口。
* 都实现了RandomAccess和Cloneable接口,实现RandomAccess接口，意味着它们都支持快速随机访问；实现Cloneable接口，意味着它们能克隆自己。
* 都是通过数组实现的，本质上都是动态数组.
* 默认数组容量是10
* 都支持Iterator和listIterator遍历.

### 1.4.2 不同之处

* 线程安全性不一样,ArrayList是非线程安全；,而Vector是线程安全的，它的函数都是synchronized的，即都是支持同步的。,ArrayList适用于单线程，Vector适用于多线程。
* 对序列化支持不同,ArrayList支持序列化，而Vector不支持；即ArrayList有实现java.io.Serializable接口，而Vector没有实现该接口。
* 构造函数个数不同,ArrayList有3个构造函数，而Vector有4个构造函数。Vector除了包括和ArrayList类似的3个构造函数之外，另外的一个构造函数可以指定容量增加系数。
* 容量增加方式不同,逐个添加元素时，若ArrayList容量不足时，“新的容量”=“(原始容量x3)/2 + 1”。而Vector的容量增长与“增长系数有关”，若指定了“增长系数”，且“增长系数有效(即，大于0)”；那么，每次容量不足时，“新的容量”=“原始容量+增长系数”。若增长系数无效(即，小于/等于0)，则“新的容量”=“原始容量 x 2”。
* 对Enumeration的支持不同。Vector支持通过Enumeration去遍历，而ArrayList不支持.

# 2 AbstractList

​		AbstractList的定义如下：

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {}
```

​		AbstractList是一个继承于AbstractCollection，并且实现List接口的抽象类。它实现了List中除size()、get(int location)之外的函数。

​		AbstractList的主要作用：它实现了List接口中的大部分函数。从而方便其它类继承List。

​		另外，和AbstractCollection相比，AbstractList抽象类中，实现了iterator()接口。

## 2.1 内部类

### 2.1.1 Itr迭代器基础类

​		Itr只是简单实现了Iterator的next, remove方法。只能向后遍历。

源码（略）

### 2.1.2 ListItr列表迭代器

​		ListItr是Itr的增强版，在Itr基础上多了向前和set操作。

源码（略）

### 2.1.3 subList

​		subList还是使用的外部类的方法和数据，改动子列表中数据，实际外部List也会发生改变。

### 2.1.4 RandomAccessSubList

​		RandomAccessSubList只不过是在SubList之外加了个RandomAccess的标识，表明他可以支持随机访问而已。

### 2.1.5 RandomAccess

​		RandomAccess 是一个空的接口，它用来标识某个类是否支持 随机访问（随机访问，相对比“按顺序访问”）。一个支持随机访问的类明显可以使用更加高效的算法。

​		List 中支持随机访问最佳的例子就是 ArrayList, 它的数据结构使得 get(), set(), add()等方法的时间复杂度都是 O(1);反例就是 LinkedList, 链表结构使得它不支持随机访问，只能按序访问，因此在一些操作上性能略逊一筹。

​		通常在操作一个 List 对象时，通常会判断是否支持 随机访问，也就是`是否为 RandomAccess 的实例`，从而使用不同的算法。

​		比如遍历，实现了 RandomAccess 的集合使用 get():

```java
for (int i=0, n=list.size(); i &lt; n; i++)
	list.get(i);
```

​		比用迭代器更快：

```java
for (Iterator i=list.iterator(); i.hasNext(); )
      i.next();
```


​		实现了 RandomAccess 接口的类有： ArrayList, AttributeList, CopyOnWriteArrayList, Vector, Stack 等。

# 3 AbstractSequentialList

```java
public abstract class AbstractSequentialList<E> extends AbstractList<E>();
```

​		AbstractSequentialList 继承自 AbstractList，是 LinkedList 的父类，是 List 接口 的简化版实现。简化在 AbstractSequentialList只支持按次序访问，而不像 AbstractList 那样支持随机访问。

​		想要实现一个支持按次序访问的 List的话，只需要继承这个抽象类，然后把指定的抽象方法实现就好了。