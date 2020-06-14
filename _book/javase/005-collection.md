## 1 概述

​		Java集合是java提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。Java集合工具包位置是·`java.util.*`

​		Java集合主要可以划分为4个部分：List列表、Set集合、Map映射、工具类(Iterator迭代器、Enumeration枚举类、Arrays和Collections)。

​		Java集合框架图：

![005-1](images\005-1.png)

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

​		fail-fast 机制是java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，当其中一个或几个线程通过`iterator`去遍历集合，其他的线程读写集合，则会导致fast-fast事件。

​		例如：当某一个线程A通过`iterator`去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

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

​		从中，我们可以发现在调用` next() `和 `remove()`时，都会执行 `checkForComodification()`。若 “modCount 不等于 expectedModCount”，则抛出ConcurrentModificationException异常，产生fail-fast事件。   

​		要搞明白 fail-fast机制，我们就要需要理解什么时候“modCount 不等于 expectedModCount”！。   

​		从Itr类中，我们知道 expectedModCount 在创建Itr对象时，被赋值为 modCount。通过Itr，我们知道：expectedModCount不可能被修改为不等于 modCount。所以，需要考证的就是modCount何时会被修改。  

​		接下来，我们查看ArrayList的源码，来看看modCount是如何被修改的。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ...
    // list中容量变化时，对应的同步函数
    public void ensureCapacity(int minCapacity) {
        modCount++;
        int oldCapacity = elementData.length;
        if (minCapacity > oldCapacity) {
            Object oldData[] = elementData;
            int newCapacity = (oldCapacity * 3)/2 + 1;
            if (newCapacity < minCapacity)
                newCapacity = minCapacity;
            // minCapacity is usually close to size, so this is a win:
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }

    // 添加元素到队列最后
    public boolean add(E e) {
        // 修改modCount
        ensureCapacity(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    // 添加元素到指定的位置
    public void add(int index, E element) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(
            "Index: "+index+", Size: "+size);

        // 修改modCount
        ensureCapacity(size+1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
             size - index);
        elementData[index] = element;
        size++;
    }

    // 添加集合
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        // 修改modCount
        ensureCapacity(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
   

    // 删除指定位置的元素 
    public E remove(int index) {
        RangeCheck(index);

        // 修改modCount
        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // Let gc do its work

        return oldValue;
    }

    // 快速删除指定位置的元素 
    private void fastRemove(int index) {
        // 修改modCount
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // Let gc do its work
    }

    // 清空集合
    public void clear() {
        // 修改modCount
        modCount++;

        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
    ...
}	
```

​		从中，我们发现：无论是`add()`、`remove()`，还是`clear()`，只要涉及到修改集合中的元素个数时，都会改变modCount的值。
接下来，我们再系统的梳理一下fail-fast是怎么产生的。步骤如下：

* 新建了一个ArrayList，名称为arrayList。
* 向arrayList中添加内容。
* 新建一个“线程a”，并在“线程a”中通过Iterator反复的读取arrayList的值。
* 新建一个“线程b”，在“线程b”中删除arrayList中的一个“节点A”。
* 这时，就会产生有趣的事件了。

​		在某一时刻，“线程a”创建了arrayList的Iterator。此时“节点A”仍然存在于arrayList中，创建arrayList时，expectedModCount = modCount(假设它们此时的值为N)。

​		在“线程a”在遍历arrayList过程中的某一时刻，“线程b”执行了，并且“线程b”删除了arrayList中的“节点A”。“线程b”执行remove()进行删除操作时，在remove()中执行了“modCount++”，此时modCount变成了N+1！“线程a”接着遍历，当它执行到next()函数时，调用checkForComodification()比较“expectedModCount”和“modCount”的大小；而“expectedModCount=N”，“modCount=N+1”,这样，便抛出ConcurrentModificationException异常，产生fail-fast事件。

​		至此，我们就完全了解了fail-fast是如何产生的！即，当多个线程对同一个集合进行操作的时候，某线程访问集合的过程中，该集合的内容被其他线程所改变(即其它线程通过add、remove、clear等方法，改变了modCount的值)；这时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

### 6.2 fail-fast解决办法

​		fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用`java.util.concurrent包下的类`去取代`java.util包下的类`。

​		所以，本例中只需要将ArrayList替换成java.util.concurrent包下对应的类即可。即，将代码
`private static List<String> list = new ArrayList<String>();`

​		替换为
`private static List<String> list = new CopyOnWriteArrayList<String>();`则可以解决该办法。

### 6.3 解决fail-fast的原理

​		上面，说明了“解决fail-fast机制的办法”，也知道了“fail-fast产生的根本原因”。接下来，我们再进一步谈谈java.util.concurrent包中是如何解决fail-fast事件的。

​		还是以和ArrayList对应的CopyOnWriteArrayList进行说明。我们先看看CopyOnWriteArrayList的源码：

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ...
    // 返回集合对应的迭代器
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    ...

    private static class COWIterator<E> implements ListIterator<E> {
        private final Object[] snapshot;
    
        private int cursor;
    
        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            // 新建COWIterator时，将集合中的元素保存到一个新的拷贝数组中。
            // 这样，当原始集合的数据改变，拷贝数据中的值也不会变化。
            snapshot = elements;
        }
    
        public boolean hasNext() {
            return cursor < snapshot.length;
        }
    
        public boolean hasPrevious() {
            return cursor > 0;
        }
    
        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }
    
        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }
    
        public int nextIndex() {
            return cursor;
        }
    
        public int previousIndex() {
            return cursor-1;
        }
    
        public void remove() {
            throw new UnsupportedOperationException();
        }
    
        public void set(E e) {
            throw new UnsupportedOperationException();
        }
    
        public void add(E e) {
            throw new UnsupportedOperationException();
        }
    }
    ...

}
```

​	从中，我们可以看出:

* 和`ArrayList`继承于`AbstractList`不同，`CopyOnWriteArrayList`没有继承于`AbstractList`，它仅仅只是实现了`List`接口。
* `ArrayList`的`iterator()`函数返回的`Iterator`是在`AbstractLis`中实现的；而`CopyOnWriteArrayList`是自己实现`Iterator`
* `ArrayList`的`Iterator`实现类中调用`next()`时，会“调用`checkForComodification()`比较`expectedModCount`和`modCount`的大小”；但是，`CopyOnWriteArrayList`的`Iterator`实现类中，没有所谓的`checkForComodification()`，更不会抛出`ConcurrentModificationException`异常！ 