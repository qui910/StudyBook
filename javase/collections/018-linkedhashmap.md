# 1 概述

​		LinkedHashMap是在HashMap的基础上做的扩展，可以认为是**HashMap+LinkedList**，即它既使用HashMap操作数据结构，又使用LinkedList维护插入元素的先后顺序。由此得出LinkedHashMap的几个特征。

* LinkedHashMap是否允许空，Key和Value都允许空
* LinkedHashMap是否允许重复数据，Key重复会覆盖、Value允许重复
* LinkedHashMap是否有序，有序
* LinkedHashMap是否线程安全，非线程安全

# 2 常用API

```java

```





# 3 成员变量

```java
//  头节点
transient LinkedHashMap.Entry<K,V> head;
//  尾节点
transient LinkedHashMap.Entry<K,V> tail;
// true表示最近最少使用次序，false表示插入顺序
final boolean accessOrder;
```

​		LinkedHashMap中并没有什么操作数据结构的方法，也就是说LinkedHashMap操作数据结构（比如put一个数据），和HashMap操作数据的方法完全一样，无非就是细节上有一些的不同罢了。

# 4 构造函数







# 5 源码解析

## 5.1 Entry

```java
 // LinkedHashMap的节点是在HashMap的基础上，增加before和after 两个属性，形成链表。
// 底层数据的存储还是使用hash表
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

## 5.2 linkNodeLast

```java
//  调用put方法时，通过newNode生成新数据节点的过程中在将新节点添加的链表中    
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

## 5.3 afterNodeRemoval

```java
// 删除节点后，调用此方法清除链表中的节点数据
void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }
```

## 5.4 afterNodeAccess

```java
//    这里配合accessOrder使用，当accessOrder为true时，将get等操作后的节点，放入外部链表的最后
//   也就是说，最新使用的节点在后，不经常使用的节点在前。
//   也就是指，迭代时呈现的顺序不是插入顺序，而是最小使用次数。
void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```



## 5.5 LinkedHashIterator

```java
// LinkedHashMap内部迭代器父类   
abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next;
    LinkedHashMap.Entry<K,V> current;
    int expectedModCount;

    LinkedHashIterator() {
        next = head;
        expectedModCount = modCount;
        current = null;
    }

    public final boolean hasNext() {
        return next != null;
    }

    // 与HashMap的区别就是，这里是通过外部保存的链表进行迭代，而非通过hash表。
    // 这样就保证了迭代的顺序就是写入的顺序
    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}
```





# 6 应用场景





# 7 并发问题





# 8 循环迭代

代码示例：

```java
JavaLearning：com.prd.colletctions.LinkedHashMapTest
```







