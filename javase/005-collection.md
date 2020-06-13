## 1 概述

​		Java集合是java提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。Java集合工具包位置是·`java.util.*`

​		Java集合主要可以划分为4个部分：List列表、Set集合、Map映射、工具类(Iterator迭代器、Enumeration枚举类、Arrays和Collections)。

​		Java集合框架图：

![005-1](F:\PersonalFolder\WorkFolder\GITBOOK仓库\StudyBook\javase\images\005-1.png)

## 2 Collection

​		`Collection`是一个接口，它主要的两个分支是：`List `和 `Set`。

​		`List`和`Set`都是接口，它们继承于`Collection`。`List`是有序的队列，`List`中可以有重复的元素；而`Set`是数学概念中的集合，`Set`中没有重复元素！

​		为了方便，我们抽象出了`AbstractCollection`抽象类，它实现了`Collection中`的绝大部分函数；这样，在`Collection`的实现类中，我们就可以通过继承`AbstractCollection`省去重复编码。`AbstractList`和`AbstractSet`都继承于`AbstractCollection`，具体的`List`实现类继承于`AbstractLis`，而`Set`的实现类则继承于`AbstractSet`。

​		另外，`Collection`中有一个`iterator()`函数，它的作用是返回一个`Iterator`接口。通常，我们通过`Iterator`迭代器来遍历集合。`ListIterator`是`List`接口所特有的，在`List`接口中，通过`ListIterator()`返回一个`ListIterator`对象。

### 2.1 API

```java
//确保此collection包含指定的元素object
abstract boolean         add(E object)	
//将指定collection中的所有元素都添加到此collection中
abstract boolean         addAll(Collection<? extends E> collection) 
//移除此 collection 中的所有元素
abstract void            clear()
//如果此collection包含指定的元素，则返回true
abstract boolean         contains(Object object)	
//如果此collection包含指定collection中的所有元素，则返回true
abstract boolean         containsAll(Collection<?> collection)	
//比较此collection与指定对象是否相等
abstract boolean         equals(Object object)	
//返回此collection的哈希码值
abstract int             hashCode()		
//如果此collection不包含元素，则返回true
abstract boolean         isEmpty()	
//返回在此collection的元素上进行迭代的迭代器
abstract Iterator<E>     iterator()	
//从此collection中移除指定元素的单个实例，如果存在的话
abstract boolean         remove(Object object)	
//移除此collection中那些也包含在指定collection中的所有元素
abstract boolean         removeAll(Collection<?> collection)
//仅保留此collection中那些也包含在指定collection的元素
abstract boolean         retainAll(Collection<?> collection)
//返回此collection中的元素数
abstract int             size()		
//返回包含此collection中所有元素的数组；返回数组的运行时类型与指定数组的运行时类型相同
abstract <T> T[]         toArray(T[] array)	
//返回包含此collection中所有元素的数组
abstract Object[]        toArray()	
```



## 3 AbstractCollection

​		`AbstractCollection`的定义如下：

```java
public abstract class AbstractCollection<E> implements Collection<E> {}
```

​		`AbstractCollection`是一个抽象类,不能实例化，它实现了`Collection`中除`iterator()`和`size()`之外的函数。

​		`AbstractCollection`的主要作用：它实现了`Collection`接口中的大部分函数。从而方便其它类实现`Collection`，比如`ArrayList`、`LinkedList`等，它们这些类想要实现`Collection`接口，通过继承`AbstractCollection`就已经实现了大部分的接口了。

## 4 Iterator

​		Iterator的定义如下：

```java
public interface Iterator<E> {}
```

​		Iterator是一个接口，它是集合的迭代器。集合可以通过Iterator去遍历集合中的元素。Iterator提供的API接口，包括：是否存在下一个元素、获取下一个元素、删除当前元素。

​		注意：Iterator遍历Collection时，是fail-fast机制的。即，当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 4.1 Iterator的API

```java
//以正向遍历列表时，如果next返回一个元素而不是抛出异常，则返回true。
abstract boolean hasNext()
//返回列表中的下一个元素。
abstract E next()
//从列表中移除由next返回的最后一个元素
abstract void remove()
```

## 5 ListIterator
​		ListIterator的定义如下：

```java
public interface ListIterator<E> extends Iterator<E> {}
```

​		ListIterator是一个继承于Iterator的接口，它是队列迭代器。专门用于遍历List，能提供向前/向后遍历。相比于Iterator，它新增了添加、是否存在上一个元素、获取上一个元素等等API接口。

### 5.1.ListIterator的API

```java
//从列表中移除由next或previous 返回的最后一个元素		
abstract void remove()
//将指定的元素插入列表
abstract void add(E object)
//如果以逆向遍历列表，列表迭代器有多个元素，则返回true
abstract boolean hasPrevious()	
//返回对next的后续调用所返回元素的索引
abstract int nextIndex()
//返回列表中的前一个元素
abstract E previous()
//返回对previous的后续调用所返回元素的索引
abstract int previousIndex()	
//指定元素替换next或previous返回的最后一个元素
abstract void set(E object)	
```

## 6 fail-fast

​		fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

​		例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 6.1 fail-fast原理

​		产生fail-fast事件，是通过抛出ConcurrentModificationException异常来触发的。

​		那么，ArrayList是如何抛出ConcurrentModificationException异常的呢?

​		我们知道，ConcurrentModificationException是在操作Iterator时抛出的异常。我们先看看Iterator的源码。ArrayList的Iterator是在父类AbstractList.java中实现的。代码如下：

```java
package java.util;
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    ...
    // AbstractList中唯一的属性
    // 用来记录List修改的次数：每修改一次(添加/删除等操作)，将modCount+1
    protected transient int modCount = 0;

    // 返回List对应迭代器。实际上，是返回Itr对象。
    public Iterator<E> iterator() {
        return new Itr();
    }
    
    // Itr是Iterator(迭代器)的实现类
    private class Itr implements Iterator<E> {
        int cursor = 0;
    
        int lastRet = -1;
    
        // 修改数的记录值。
        // 每次新建Itr()对象时，都会保存新建该对象时对应的modCount；
        // 以后每次遍历List中的元素的时候，都会比较expectedModCount和modCount是否相等；
        // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
        int expectedModCount = modCount;
    
        public boolean hasNext() {
            return cursor != size();
        }
    
        public E next() {
            // 获取下一个元素之前，都会判断“新建Itr对象时保存的modCount”和“当前的modCount”是否相等；
            // 若不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件。
            checkForComodification();
            try {
                E next = get(cursor);
                lastRet = cursor++;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }
    
        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            checkForComodification();
    
            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }
    
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
    ...

}
```





