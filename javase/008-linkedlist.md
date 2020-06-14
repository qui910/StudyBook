# 1 概述

* 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
* 实现 List 接口，能对它进行队列操作。
* 实现 Deque 接口，即能将LinkedList当作双端队列使用。
* 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
* 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
* 是非同步的。



# 2 成员变量

```java
// 头节点,prev为null的节点，（只有在列表为空时first为null,同时last也必须是null）
transient Node<E> first;
// 尾节点，next为null的节点，（只有在列表为空时last为null,同时first也必须是null）
transient Node<E> last;
// 容量
transient int size = 0;
```

## 2.1 Node

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

​		其说明如图：

![008-1](images\008-1.png)

## 2.2 DescendingIterator倒序迭代器

​		就是游标直接在迭代器尾部，然后颠倒黑白，说是向后遍历，实际是向前遍历.内部通过获取ListItr实现。

## 2.3 ListItr列表迭代器

# 3 常用API

```java
////// 从List接口
// 返回一份列表迭代器，以便用来访问列表中的元素
ListIterator<E>     listIterator()
// 返回一份列表迭代器，以便用来访问列表中的元素，这个元素是第一次调用next返回的给定索引的元素
ListIterator<E>     listIterator(int location)
// 给列表尾部追加元素
boolean       add(E object)
// 指定位置添加元素
void          add(int location, E object)
// 将某个集合中的所有元素添加到给定位置
void       addAll(Collection<? extends E> collection)
// 删除给定位置的元素并返回这个元素
E             remove(int location)
// 获取给定位置的元素
E             get(int location)
// 用新元素取代给定位置的元素，并返回原来的那个元素
E             set(int location, E object)
// 返回与指定元素相等的元素在列表中第一次出现的位置，如果没有则返回-1
int           indexOf(Object object)、
// 返回与指定元素相等的元素在列表中最后出现的位置，如果没有则返回-1
int           lastIndexOf(Object object)
////// 从ListIterator接口
// 在当前位置前添加一个元素
void		  add(E object)
// 用新元素取代next或previous上次访问的元素。如果在next后previous上次调用之后列表结构被修改了，将会抛出一个IllegalStateException异常
void		  set(E object)
// 当反向迭代列表时，还有可供访问的元素时，则返回true
boolean 	  hasPrevious()
// 返回前一个对象。如果已经达到列表的头部，则抛出NoSuchElementException
E			  previous()
// 返回下一次调用next方法时将返回的元素索引
int 		  nextIndex()
// 返回下一次调用previous方法时将返回的元素索引
int 	      previousIndex()
//// LinkedList
void          addFirst(E object)
void          addLast(E object)
E             getFirst()
E             getLast()
E             removeFirst()
E             removeLast()
```

* LinkedList 实际上是通过双向链表去实现的。它包含一个非常重要的内部类：Entry。Entry是双向链表节点所对应的数据结构，它包括的属性有：当前节点所包含的值，上一个节点，下一个节点。
* 从LinkedList的实现方式中可以发现，它不存在LinkedList容量不足的问题。
* LinkedList的克隆函数，即是将全部元素克隆到一个新的LinkedList对象中。
* LinkedList实现java.io.Serializable。当写入到输出流时，先写入“容量”，再依次写入“每一个节点保护的值”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。
* 由于LinkedList实现了Deque，而Deque接口定义了在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）。

![008-2](images\008-2.png)

* LinkedList可以作为FIFO(先进先出)的队列，作为FIFO的队列时，下表的方法等价：

| 队列方法  |   等效方法    |
| :-------: | :-----------: |
|  add(e)   |  addLast(e)   |
| offer(e)  | offerLast(e)  |
| remove()  | removeFirst() |
|  poll()   |  pollFirst()  |
| element() |  getFirst()   |
|  peek()   |  peekFirst()  |

* LinkedList可以作为LIFO(后进先出)的栈，作为LIFO的栈时，下表的方法等价：

| 栈方法  |   等效方法   |
| :-----: | :----------: |
| push(e) | addFirst(e)  |
|  pop()  | removeLast() |
| peek()  |  peekLast()  |





# 4 源码解析

```java
	private void linkFirst(E e) {
       	//新增节点的next设置为f
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        // fisrt指向新节点
        first = newNode;
        // 如果原f为空，说明原列表是空，则first和last都指向newNode
        // 否则fisrt.
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }


	void linkLast(E e) {
        // 新节点的prev设为last，next为null
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        // last指向新节点
        last = newNode;
        // 如果l 即新last为null，说明原列表为空，则first和last都指向newNode
        // 否则新增为last的next
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

	// 使用二分法快速加快速度定位到需要的节点
	// 为何不使用多次二分法？
    Node<E> node(int index) {
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```





# 5 应用场景

​		可以应用于顺序访问，而且需要快速添加元素的场景。



# 6 并发问题

## 6.1  fast-fast异常

```java
        new Thread(()->{
            ListIterator<String> listIterator1 = staff.listIterator();
            listIterator1.next();
            listIterator1.remove();
        }).start();

        new Thread(()->{
            ListIterator<String> listIterator2 = staff.listIterator();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            listIterator2.next();
        }).start();
```

​		为防止多线程并发修改异常，可以遵循如下规则。

* 可以有多个可读的迭代器，不能同时存在可写迭代器。
* 只能存在一个即可写又可读的迭代器，同时不能有其他可读迭代器。

## 6.2 多线程并发和ArrayList一样会产生问题

```java
        LinkedList<String> integers = new LinkedList<>();
        for (int i=0;i<10000;i++) {
            new Thread(()->{
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                integers.add(Thread.currentThread().getName());
            },i+"").start();
        }
        Thread.sleep(10000);
        log.info("integers size={}",integers.size());
        for (int i=0;i<integers.size();i++) {
            String s = integers.get(i);
            if (null==s||"".equals(s)) {
                log.info("integers index i={}, s={}",i,s);
            }
        }
```



# 7 循环迭代

​		LinkedList支持多种遍历方式。建议不要采用随机访问的方式去遍历LinkedList，而采用逐个遍历的方式。
* 第一种，通过迭代器遍历。即通过Iterator去遍历。

  ```java
for(Iterator iter = list.iterator(); iter.hasNext();)
  iter.next();
  ```
  
* 第二种，通过快速随机访问遍历LinkedList

  ```java
int size = list.size();
for (int i=0; i<size; i++) {
  list.get(i);        
  }
  ```

* 第三种，通过另外一种for循环来遍历LinkedList

  ```java
for (Integer integ:list) 
  ;
  ```

* 第四种，通过pollFirst()来遍历LinkedList

  ```java
while(list.pollFirst() != null)
  ;
  ```

* 第五种，通过pollLast()来遍历LinkedList

  ```java
while(list.pollLast() != null)
  ;
  ```

* 第六种，通过removeFirst()来遍历LinkedList

  ```java
  try {
  while(list.removeFirst() != null)
      ;
  } catch (NoSuchElementException e) {
  }
  ```

* 第七种，通过removeLast()来遍历LinkedList

  ```java
  try {
  while(list.removeLast() != null)
      ;
  } catch (NoSuchElementException e) {
  }
  ```

*  第八种，list.forEach(l->{})
  ```java
  try {
  while(list.removeLast() != null)
      ;
  } catch (NoSuchElementException e) {
  }
  ```

测试结果：
```shell
20:41:44.083 [main] INFO  com.prd.colletctions.LinkedListTest - 第1种迭代器访问,耗时82
20:41:44.162 [main] INFO  com.prd.colletctions.LinkedListTest - 第3种for循环遍历访问,耗时77
20:41:44.275 [main] INFO  com.prd.colletctions.LinkedListTest - 第8种forEach流式访问,耗时112
```

代码示例：

```java
JavaLearning：com.prd.colletctions.LinkedListTest
```

同样的数据量级，LinkedList的循环效率比ArrayList要低。而且随机循环第二种方式，因为时间太久无法循环计算出最后的时间。