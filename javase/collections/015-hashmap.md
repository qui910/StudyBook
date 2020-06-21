# 1 概述

* HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。
* HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。
* HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。
* HashMap 的实例有两个参数影响其性能：“初始容量” 和 “加载因子”。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行rehash操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。
* 通常，默认加载因子是 0.75, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。
  当发生 哈希冲突（碰撞）的时候，HashMap 采用 拉链法 进行解决（不熟悉 “哈希冲突” 和 “拉链法”），因此 HashMap 的底层实现是 数组+链表，如下图 所示：

![015-1](..\images\015-1.png)

# 2 常用API

```java
// map大小
public int size();
// map是否为空（即size==0）
public boolean isEmpty();
// 根据key的hashcode，定位到table中哪个下标（(n - 1) & hash），然后遍历这个链表（如第1节概述中的示图）或树形结构，找到hashcode相等的value返回。如果返回null，不代表key不存在，可能value就是null。所以判断key是否存在应使用containsKey
public V get(Object key);
// 判断key是否存在
public boolean containsKey(Object key);
// 添加元素
public V put(K key, V value);
```



# 3 成员变量

```java
// 初始容量 16 即 1 * 2^4
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

static final int TREEIFY_THRESHOLD = 8;

static final int UNTREEIFY_THRESHOLD = 6;

static final int MIN_TREEIFY_CAPACITY = 64;
// hash列表的基础数据节点 实现Entry接口
static class Node<K,V> implements Map.Entry<K,V>
// Node节点存放数组，大小为2的N次方    
transient Node<K,V>[] table;
// entrySet的缓存，为keySet()和values()查询时使用
transient Set<Map.Entry<K,V>> entrySet;
// map实际大小
transient int size;
// map的结构调整记录数，用来表示fast-fast事件
transient int modCount;
// 下次map重新调整时的大小，为容量*加载因子
int threshold;
// 加载因子
final float loadFactor;
```



# 4 构造函数

```java
// 指定初始容量，和加载因子
public HashMap(int initialCapacity, float loadFactor);
// 指定初识容量，(加载因子0.75)
public HashMap(int initialCapacity);
// (初识容量16，加载因子 0.75)
public HashMap();
// (初识容量为m的大小，加载因子0.75)
public HashMap(Map<? extends K, ? extends V> m);
```









# 5 源码解析

## 5.1 hash

```java
// 由于哈希表的容量都是 2 的 N 次方，在当前，元素的 hashCode() 在很多时候下低位是相同的，这将导致冲突（碰撞），因此 1.8 以后做了个移位操作：将元素的 hashCode() 和自己右移 16 位后的结果求异或。
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

由于 int 只有 32 位，无符号右移 16 位相当于把高位的一半移到低位：

![015-2](..\images\015-2.png)

例如：

![015-3](..\images\015-3.png)

这样可以避免只靠低位数据来计算哈希时导致的冲突，计算结果由高低位结合决定，可以避免哈希值分布不均匀。而且，采用位运算效率更高。

## 5.2 tableSizeFor

```java
// 经过若干次无符号右移、求异运算，得出最接近指定参数 cap 的 2 的 N 次方容量。假如你传入的是 5，返回的初始容量为 8 。    
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

## 5.2 resize

​		这里扩容是将数组扩容为原有大小的2倍。在理解扩容之前，要理解是如何计算数组下标的，`(n - 1) & hash`。

​		首先要明确一点，位运算的速度远远高于取余操作，假设我们要计算`X%Y`，如果 `Y是2的N次幂`，则`X%Y`可以变化为`X&(Y-1)`。这也就是为何Map的数组都是以2的N次幂来扩容，这样在通过hash定位数组下标时非常快速。

```java
log.info("int 15&13={}，13%16={}",15 & 13,13%16); // 15&13=13，13%16=13
log.info("int 15&29={}，29%16={}",15 & 29,29%16); // 15&29=13，29%16=13
```

​		 由上述代码假设，Map大小为16，元素13，29都将存入下标为13的链表中。

```java
    final Node<K,V>[] resize() {
        //保存旧的Hash表
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //超过最大容量，不再进行扩充
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //容量没有超过最大值，容量变为原来的两倍 2^n-->2^(n+1)
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1;
        }
        else if (oldThr > 0) 
            newCap = oldThr;
       // 初始化时默认值和默认加载因子
        else {               
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算新阀值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            //阀值没有超过最大阀值，设置新的阀值
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        // 创新新Hash表
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 单节点，直接计算下标保存
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 红黑树模式
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 链表保存的节点
                    // 根据hash值与旧hash表的大小2^n做与运算，将数据一分为二
                    else { 
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // e.hash/oldCap 商为0或偶数
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // e.hash/oldCap 商为奇数
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 为偶数时写入 原下标位置，且最后一个节点的下一个节点做空
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 为奇数时写入 原（下标+原数组大小）位置，且最后一个节点的下一个节点做空
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

​		扩容的关键语句`e.hash & oldCap`，`newTab[j] = loHead`和`newTab[j + oldCap] = hiHead;`。

```java
log.info("int 16&29={},16&13={}",16&29,16&13);// 16&29=16,16&13=0
log.info("int 29/16={},13/16={}",29/16,13/16);// 29/16=1,13/16=0
```

​		`e.hash & oldCap`看如上示例，拿刚才13和29举例说明，计算` X & Y`，`Y为2的N次幂`，只会有两种结果 `0` 或 `Y`，当 `X/Y` 商为`0或偶数`时为0，商为`奇数`时为Y。这样的做，就是将就hash表oldCap中的数据分为两份（奇数，偶数），偶数部分不动还是存放在原下标的位置即`newTab[13]`，而奇数部分存放到`newTab[13+16]`中。

​		与原hash表大小做与操作，并分别将数据保存在`newTab[j]`和`newTab[j + oldCap]`中，其实与新hash表重新计算下标的结果是一致的。同样是位操作，不明白为何这么做??

```java
log.info("int 31&13={}，13%32={}",31&13,13%32);// 31&13=13，13%32=13
log.info("int 31&29={}，29%32={}",31&29,29%32);// 31&29=29，29%32=29
```





参考网址：

[位运算总结 取模 取余](https://blog.csdn.net/black_ox/article/details/46411997)

[[java位运算](https://www.cnblogs.com/highriver/archive/2011/08/15/2139600.html)]



# 6 应用场景





# 7 并发问题

## 7.1 多个进程同时写入

可能导致写入的数据缺失。示例同时写入2000个数据，结果可能不为2000

```java
      HashMap<Integer,String> testMap = new HashMap<Integer,String>();
      // 测试并发写 HashMap
      for (int i=0;i<2000;i++) {
          String name = Thread.currentThread().getName()+i;
          final  int key = i;
          new Thread(()-> {
              testMap.put(key,name);
          }).start();
      }
      Thread.sleep(20000);
      log.info("Hashmap 的大小为{}",testMap.size());
```

结果：`Hashmap 的大小为1999`

## 7.2 [HashMap在高并发下引起的死循环](https://www.cnblogs.com/yjbjingcha/p/6957909.html)





# 8 循环迭代







