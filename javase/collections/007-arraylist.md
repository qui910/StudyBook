# 1 概述

* ArrayList 是一个数组队列，相当于 动态数组。与Java中的数组相比，它的容量能动态增长。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。
* ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
* ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。
* ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。
* ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。
* 和Vector不同，ArrayList中的操作不是线程安全的！以在并发访问时，如果在迭代的同时有其他线程修改了ArrayList, fail-fast的迭代器 Iterator/ListIterator会报`ConcurrentModificationException`错.所以，建议在单线程中才使用ArrayList，而在多线程中可以选择`Vector`或者`CopyOnWriteArrayList`或`Collections.synchronizedList`。

## 1.1 特点

* 容量不固定，想放多少放多少（当然有最大阈值，但一般达不到）

* 有序的（元素输出顺序与输入顺序一致）

* 元素可以为 null

* 效率高 

  * size(), isEmpty(), get(), set() iterator(), ListIterator() 方法的时间复杂度都是 O(1)

  * add() 添加操作的时间复杂度平均为 O(n)

  * 其他所有操作的时间复杂度几乎都是 O(n)

* 占用空间更小 ，对比 LinkedList，不用占用额外空间维护链表结构

# 2 成员变量

## 2.1 elementData

​		Object[]类型的数组，它保存了添加到ArrayList中的元素,允许元素为null。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。

​		elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，请参考源码分析中的ensureCapacity()函数。transient 说明这个数组无法序列化。 

​		初始时为 EMPTY_ELEMENTDATA 。

## 2.2 size
​		动态数组的实际大小

## 2.3 MAX_ARRAY_SIZE

​		数组最大容量.
​		`Integer.MAX_VALUE = 0x7fffffff;`换算成二进制： 2^31 - 1，1111111111111111111111111111111;十进制就是 ：2147483647，二十一亿多。
​		一些虚拟器需要在数组前加个头标签，所以减去8。 当想要分配比MAX_ARRAY_SIZE大的个数就会报 OutOfMemoryError。

#  3 常用API

```java
boolean             add(E object)
boolean             addAll(Collection<? extends E> collection)
void                clear()
boolean             contains(Object object)
boolean             containsAll(Collection<?> collection)
boolean             equals(Object object)
int                 hashCode()
boolean             isEmpty()
Iterator<E>         iterator()
boolean             remove(Object object)
boolean             removeAll(Collection<?> collection)
boolean             retainAll(Collection<?> collection)
int                 size()
<T> T[]             toArray(T[] array)
Object[]            toArray()
// AbstractCollection中定义的API
void                add(int location, E object)
boolean             addAll(int location, Collection<? extends E> collection)
E                   get(int location)
// 返回第一个相同元素的位置
int                 indexOf(Object object)
// 返回最后一个相同元素的位置
int                 lastIndexOf(Object object)
ListIterator<E>     listIterator(int location)
ListIterator<E>     listIterator()
E                   remove(int location)
E                   set(int location, E object)
List<E>             subList(int start, int end)
// ArrayList新增的API
Object               clone()
void                 ensureCapacity(int minimumCapacity)
void                 trimToSize()
void                 removeRange(int fromIndex, int toIndex)
```



# 4 源码解析

​		容量增加：

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 新容量为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 扩容容量如果大于旧容量的1.5倍，则大小以扩容容量为主
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

​		查询，修改等操作，直接根据角标对数组操作：

```java
    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
    
    public E get(int index) {
        // 检查是否越界
        rangeCheck(index);
        return elementData(index);
    }
    // 更新元素为新元素，并返回就元素
    public E set(int index, E element) {
        rangeCheck(index);
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 多线程并发时，问题点。（分为两步，第1是elementData[size] = e;
        // 第2是 size++ ），非原子操作，可能导致数据丢失或为空
        elementData[size++] = e;
        return true;
    }

    private void ensureExplicitCapacity(int minCapacity) {
        // 修改modCount，fast-fast事件发生原因
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

​		删除，还是有点慢：

```java
//根据位置删除
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    //挨个往前移一位
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //原数组中最后一个元素删掉
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

//删除某个元素
public boolean remove(Object o) {
    if (o == null) {
        //挨个遍历找到目标
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                //快速删除
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 内部方法，“快速删除”，就是把重复的代码移到一个方法里
// fastremove因为不需要边界检查和返回值，因此称作fast
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

//保留公共的
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

//删除或者保留指定集合中的元素
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    //使用两个变量，一个负责向后扫描，一个从 0 开始，等待覆盖操作
    int r = 0, w = 0;
    boolean modified = false;
    try {
        //遍历 ArrayList 集合
        for (; r < size; r++)
            //如果指定集合中是否有这个元素，根据 complement 判断是否往前覆盖删除
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        //发生了异常，直接把 r 后面的复制到 w 后面
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // 清除多余的元素，clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}

//清楚全部
public void clear() {
    modCount++;
    //并没有直接使数组指向 null,而是逐个把元素置为空
    //下次使用时就不用重新 new 了
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```



# 5 应用场景

​		单线程下，随机访问，读取，修改和新增、需要非线程安全的逐条遍历时，可以使用LinedList。



# 6 并发问题

结果1.并发导致数据丢失

结果2.并发导致插入null

结果3.并发导致数组越界

代码示例：

```java
JavaLearning：com.prd.colletctions.ArrayListText
```

[并发下的ArrayList错误分析](https://blog.csdn.net/lan861698789/article/details/81697409)

# 7 遍历

**ArrayList支持4种遍历方式**

* 第一种，**通过迭代器遍历**。即通过Iterator去遍历。

```java
Integer value = null;
Iterator iter = list.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```

* 第二种，**随机访问，通过索引值去遍历。**
  由于ArrayList实现了RandomAccess接口，它支持通过索引值去随机访问元素。

```java
Integer value = null;
int size = list.size();
for (int i=0; i<size; i++) {
    value = (Integer)list.get(i);        
}
```

* 第三种，**for循环遍历**。如下：

```java
Integer value = null;
for (Integer integ:list) {
    value = integ;
}
```

* 第四种，**forEach**。如下：

```java
list.forEach(l->{})
```

测试几种遍历效率：

```shell
17:06:49.869 [main] INFO  com.prd.colletctions.ArrayListText - 第1种迭代器访问,耗时5
17:06:49.873 [main] INFO  com.prd.colletctions.ArrayListText - 第2种随机访问,耗时3
17:06:49.877 [main] INFO  com.prd.colletctions.ArrayListText - 第3种for循环遍历访问,耗时4
17:06:49.919 [main] INFO  com.prd.colletctions.ArrayListText - 第4种forEach流式访问,耗时41
```

代码示例：

```
JavaLearning：com.prd.colletctions.ArrayListText
```

