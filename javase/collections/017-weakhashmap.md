# 1 概述

​		WeakHashMap 继承于AbstractMap，实现了Map接口。

​		和HashMap一样，WeakHashMap 也是一个散列表，它存储的内容也是键值对(key-value)映射，而且键和值都可以是null。不过WeakHashMap的键是“弱键”。在 WeakHashMap 中，当某个键不再正常使用时，会被从WeakHashMap中被自动移除。更精确地说，对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的丢弃，这就使该键成为可终止的，被终止，然后被回收。某个键被终止时，它对应的键值对也就从映射中有效地移除了。

​		这个“弱键”的原理呢？

​		大致上就是，通过WeakReference和ReferenceQueue实现的。 WeakHashMap的key是“弱键”，即是WeakReference类型的；ReferenceQueue是一个队列，它会保存被GC回收的“弱键”。实现步骤是：

* 新建WeakHashMap，将“键值对”添加到WeakHashMap中。实际上，WeakHashMap是通过数组table保存Entry(键值对)；每一个Entry实际上是一个单向链表，即Entry是键值对链表。
* 当某“弱键”不再被其它对象引用，并被GC回收时。在GC回收该“弱键”时，这个“弱键”也同时会被添加到ReferenceQueue(queue)队列中。
* 当下一次我们需要操作WeakHashMap时，会先同步table和queue。table中保存了全部的键值对，而queue中保存被GC回收的键值对；同步它们，就是删除table中被GC回收的键值对。

​		这就是“弱键”如何被自动从WeakHashMap中删除的步骤了。

​		和HashMap一样，WeakHashMap是不同步的。可以使用 Collections.synchronizedMap 方法来构造同步的 WeakHashMap。

## 1.1 引用类型

​		理解WeakHashMap前，先来熟悉下引用的几个类型。Java对象的引用包括：强引用，软引用，弱引用，虚引用。提供这四种引用类型主要有两个目的：

* 第一是可以让程序员通过代码的方式决定某些对象的生命周期；
* 第二是有利于JVM进行垃圾回收。

### 1.1.1 强引用(Strong Reference)

​		强引用就是普通的Java引用，代码：

```java
StringBuffer buffer = new StringBuffer(); 
```

​		这句代码创建了一个StringBuffer对象，在变量buffer中存储了这个对象的一个强引用。强引用之所以称之为“强”（Strong），是因为他们与垃圾回收器（garbage collector）交互的方式。特别是（specifically），如果一个对象通过强引用连接（strongly reachable-强引用可到达），那么它就不在垃圾回收期处理之列。当你正在使用某个对象而不希望垃圾回收期销毁这个对象时，强引用通常正好能满足你所要的。

​		强引用有个最大的缺点就是内存泄漏问题，如果对象object是强引用，那在该对象不再使用后，应显示的将object引用赋值为null，否则只要引用存在，垃圾回收机制就不会回收该块内存。例如：Collections中的各种容器中，clear方法就是通过这种方式，让虚拟机自动回收释放原引用占用的资源。

```java
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```

代码示例：

```java
/**
 * 强引用测试
 * 运行参数 -Xmx200m -XX:+PrintGC
 */
@Slf4j
public class NomalReferenceTest {

  public static void main(String[] args) throws InterruptedException {
      //150M的缓存数据
      byte[] cacheData = new byte[100 * 1024 * 1024];
      log.info("GC前，cacheData的大小:{}",cacheData.length);
      System.gc();

      Thread.sleep(1000);
      //在分配一个80M的对象，看看缓存对象的回收情况
      byte[] newData = new byte[80 * 1024 * 1024];
      log.info("GC后，cacheData的大小:{}",cacheData.length);
  }
}
```

运行结果：

```shell
10:39:51.353 [main] INFO  com.prd.reference.NomalReferenceTest - GC前，cacheData的大小:104857600
[GC (System.gc())  146116K->109206K(182784K), 0.0094440 secs]
[Full GC (System.gc())  109206K->108899K(182784K), 0.0461330 secs]
[GC (Allocation Failure)  109380K->108939K(192512K), 0.0015233 secs]
[GC (Allocation Failure)  108939K->108939K(192512K), 0.0017608 secs]
[Full GC (Allocation Failure)  108939K->106252K(192512K), 0.0173967 secs]
[GC (Allocation Failure)  106252K->106252K(197120K), 0.0022011 secs]
[Full GC (Allocation Failure)  106252K->105823K(197120K), 0.0258462 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.prd.reference.NomalReferenceTest.main(NomalReferenceTest.java:20)

```

结果显示由`System.gc()`触发GC，但是并未强制回收资源，导致再行分配80M的空间时，提示OOM。



### 1.1.2 软引用(SoftReference)

​		软引用是用来描述一些还有用但并非必须的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。

​		软引用可用来实现内存敏感的高速缓存,比如**网页缓存、图片缓存**等。使用软引用能防止内存泄露，增强程序的健壮性。

​		SoftReference的特点是它的一个实例保存对一个Java对象的软引用， 该软引用的存在不妨碍垃圾收集线程对该Java对象的回收。也就是说，一旦SoftReference保存了对一个Java对象的软引用后，在垃圾线程对 这个Java对象回收前，SoftReference类所提供的get()方法返回Java对象的强引用。另外，一旦垃圾线程回收该Java对象之 后，get()方法将返回null。

```java
MyObject aRef = new  MyObject();  
SoftReference aSoftRef=new SoftReference(aRef); 
```

​		此时，对于这个MyObject对象，有两个引用路径，一个是来自SoftReference对象的软引用，一个来自变量aReference的强引用，所以这个MyObject对象是强可及对象。我们可以通过:

```java
MyObject anotherRef=(MyObject)aSoftRef.get(); 
```

​		重新获得对该实例的强引用。而回收之后，调用get()方法就只能得到null了。

​		Java虚拟机的垃圾收集线程对软可及对象和其他一般Java对象进行了区别对待：软可及对象的清理是由垃圾收集线程根据其特定算法按照内存需求决定的。也就是说，垃圾收集线程会在虚拟机抛出OutOfMemoryError之前回收软可及对象，而且虚拟机会尽可能优先回收长时间闲置不用的软可及对象，对那些刚刚构建的或刚刚使用过的“新”软可反对象会被虚拟机尽可能保留。

​		同样，使用引用队列检查已经清除了的软引用。代码：

```java
ReferenceQueue queue = new  ReferenceQueue();  
SoftReference  ref=new  SoftReference(aMyObject, queue); 
```


​		那么当这个SoftReference所软引用的aMyOhject被垃圾收集器回收的同时，ref所强引用的SoftReference对象被列入ReferenceQueue。也就是说，ReferenceQueue中保存的对象是Reference对象，而且是已经失去了它所软引用的对象的Reference对象。另外从ReferenceQueue这个名字也可以看出，它是一个队列，当我们调用它的poll()方法的时候，如果这个队列中不是空队列，那么将返回队列前面的那个Reference对象。

​		在任何时候，我们都可以调用ReferenceQueue的poll()方法来检查是否有它所关心的非强可及对象被回收。如果队列为空，将返回一个null,否则该方法返回队列中前面的一个Reference对象。利用这个方法，我们可以检查哪个SoftReference所软引用的对象已经被回收。于是我们可以把这些失去所软引用的对象的SoftReference对象清除掉。

代码示例：

```java
/**
 * 软引用测试
 * 运行参数 -Xmx200m -XX:+PrintGC
 * 测试GC时，不能用log4j等打印日志，会增加内存消耗，测试不准确
 */
public class SoftReferenceTest {
  public static void main(String[] args) throws InterruptedException {
      //100M的缓存数据
      byte[] cacheData = new byte[100 * 1024 * 1024];
      ReferenceQueue reQueue = new ReferenceQueue();
      //将缓存数据用软引用持有
      SoftReference<byte[]> cacheRef = new SoftReference<>(cacheData,reQueue);
      //将缓存数据的强引用去除
      cacheData = null;
      System.out.println("第一次GC前" + cacheData);
      System.out.println("第一次GC前" + cacheRef.get());
      System.out.println("第一次GC前" + reQueue.poll());
      //进行一次GC后查看对象的回收情况
      System.gc();
      //等待GC
      Thread.sleep(1000);
      System.out.println("第一次GC后" + cacheData);
      System.out.println("第一次GC后" + cacheRef.get());
      System.out.println("第一次GC后" + reQueue.poll());

      //在分配一个120M的对象，看看缓存对象的回收情况
      byte[] newData = new byte[120 * 1024 * 1024];
      System.out.println("第二次GC后" + cacheData);
      System.out.println("第二次GC后" + cacheRef.get());
      System.out.println("第二次GC后" + reQueue.poll());
  }
}
```

运行结果：

```
第一次GC前null
第一次GC前[B@2ac1fdc4
第一次GC前null
[GC (System.gc())  108175K->103145K(182784K), 0.0034476 secs]
[Full GC (System.gc())  103145K->103093K(182784K), 0.0095369 secs]
第一次GC后null
第一次GC后[B@2ac1fdc4
第一次GC后null
[GC (Allocation Failure)  105018K->103125K(192512K), 0.0016011 secs]
[GC (Allocation Failure)  103125K->103157K(192512K), 0.0013123 secs]
[Full GC (Allocation Failure)  103157K->102972K(192512K), 0.0084328 secs]
[GC (Allocation Failure)  102972K->102972K(197120K), 0.0047621 secs]
[Full GC (Allocation Failure)  102972K->535K(145920K), 0.0219562 secs]
第二次GC后null
第二次GC后null
第二次GC后java.lang.ref.SoftReference@5f150435
```

第一次GC后，软引用关联对象还存在（103145K->103093K(182784K)），第二次GC时因内存不足，GC自动回收资源（ 102972K->535K(145920K)）。



### 1.1.3 弱引用(WeakReference)

​		简单说，就是弱引用不足以将其连接的对象强制保存在内存中。弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，**无论内存是否充足，都会回收被弱引用关联的对象**。弱引用能够影响（leverage）垃圾回收器的某个对象的可到达级别。

代码：

```java
WeakReference<Widget> weakWidget = new WeakReference<Widget>(widget); 
```

​		可以使用weakWidget.get()方法老获取实际的强引用对象。但是在之后，有可能突然返回null值（如果没有其他的强引用在Widget之上），因为这个弱引用被回收。其中包装的Widget也被回收。

​		解决Widget序列号的问题，最简单的方法就是使用WeakHashmap。其key值为弱引用。如果一个WeakHashmap的key变成垃圾，那么它对应用的value也自动的被移除。

代码示例：

```java
/**
 * 弱引用测试
 * 运行参数  -Xmx200m -XX:+PrintGC
 * 测试GC时，不能用log4j等打印日志，会增加内存消耗，测试不准确
 */
@Slf4j
public class WeakReferenceTest {
  public static void main(String[] args) throws InterruptedException {
      //100M的缓存数据
      byte[] cacheData = new byte[100 * 1024 * 1024];
      //将缓存数据用软引用持有
      WeakReference<byte[]> cacheRef = new WeakReference<>(cacheData);
      System.out.println("第一次GC前" + cacheData);
      System.out.println("第一次GC前" + cacheRef.get());
      //进行一次GC后查看对象的回收情况
      System.gc();
      //等待GC
      Thread.sleep(500);
      System.out.println("第一次GC后" + cacheData);
      System.out.println("第一次GC后" + cacheRef.get());

      //将缓存数据的强引用去除
      cacheData = null;
      System.gc();
      //等待GC
      Thread.sleep(500);
      System.out.println("第二次GC后" + cacheData);
      System.out.println("第二次GC后" + cacheRef.get());
  }
}
```

运行结果：

```java
第一次GC前[B@5276e6b0
第一次GC前[B@5276e6b0
[GC (System.gc())  146114K->109120K(182784K), 0.0180283 secs]
[Full GC (System.gc())  109120K->108799K(182784K), 0.0722109 secs]
第一次GC后[B@5276e6b0
第一次GC后[B@5276e6b0
[GC (System.gc())  110243K->108839K(182784K), 0.0013638 secs]
[Full GC (System.gc())  108839K->3804K(182784K), 0.0441535 secs]
第二次GC后null
第二次GC后null
```

第一次GC强引用存在，GC不回收（ 109120K->108799K(182784K)）。第二次去掉强引用，剩余弱引用，GC时自动回收（108839K->3804K(182784K)）。

### 1.1.4 虚引用(PhantomReference)

​		虚引用于弱引用和软引用均不同。它控制其指向的对象非常弱（tenuous），以至于它不能获得这个对象。get()方法通常情况下返回的是null值。它唯一的作用就是跟踪列队在ReferenceQuene中的已经死去的对象。

​		虚引用和弱引用的不同在于其入队（enquene）进入ReferenceQuene的方式。当弱引用的对象成为弱可到达时，弱引用即列队进入ReferenceQuene。这个入队发生在终结（finialize）和垃圾回收实际发生之前。理论上，通过不正规（unorthodox）的`finilize()`方法，成为垃圾的对象能重新复活（resurrected），但是弱引用仍然是死的。幻象引用只有当对象在物理上从内存中移出时，才会入队。这就阻止我们重新恢复将死的对象。



参考网址：

* [软引用、弱引用、虚引用-他们的特点及应用场景](https://www.jianshu.com/p/825cca41d962)



# 2 常用API

```java
void                   clear()
Object                 clone()
boolean                containsKey(Object key)
boolean                containsValue(Object value)
Set<Entry<K, V>>       entrySet()
V                      get(Object key)
boolean                isEmpty()
Set<K>                 keySet()
V                      put(K key, V value)
void                   putAll(Map<? extends K, ? extends V> map)
V                      remove(Object key)
int                    size()
Collection<V>          values()
```



# 3 成员变量

```java
// 初始容量
private static final int DEFAULT_INITIAL_CAPACITY = 16;
// 最大容量
private static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认加载因子
private static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 底层数据存储数组，大小为2的N次方
Entry<K,V>[] table;
// 数组扩容阈值 =(capacity * load factor)
private int threshold;
// 存放被清除的弱引用的队列
private final ReferenceQueue<Object> queue = new ReferenceQueue<>();
// 变更次数，多线程情况下会引起fail-fast异常
int modCount;
```



# 4 构造函数

```java
// 默认构造函数。
WeakHashMap()

// 指定“容量大小”的构造函数
WeakHashMap(int capacity)

// 指定“容量大小”和“加载因子”的构造函数
WeakHashMap(int capacity, float loadFactor)

// 包含“子Map”的构造函数
WeakHashMap(Map<? extends K, ? extends V> map)
```



# 5 源码解析

```java
// 担心原Key的HashCode有冲突，对HashCode做二次处理。这里还是沿用的JDK1.7的做法，HashMap中JDK1.8后

final int hash(Object k) {
    int h = k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

```java
    // 清空table中无用键值对。原理如下：
    // (01) 当WeakHashMap中某个“弱引用的key”由于没有再被引用而被GC收回时，
    //   被回收的“该弱引用key”也被会被添加到"ReferenceQueue(queue)"中。
    // (02) 当我们执行expungeStaleEntries时，
    //   就遍历"ReferenceQueue(queue)"中的所有key
    //   然后就在“WeakReference的table”中删除与“ReferenceQueue(queue)中key”对应的键值对
    private void expungeStaleEntries() {
        Entry<K,V> e;
        while ( (e = (Entry<K,V>) queue.poll()) != null) {
            int h = e.hash;
            int i = indexFor(h, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    e.next = null;  // Help GC
                    e.value = null; //  "   "
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
```

​		在Java中，WeakReference和ReferenceQueue 是联合使用的。在WeakHashMap中亦是如此：如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。 接着，WeakHashMap会根据“引用队列”，来删除“WeakHashMap中已被GC回收的‘弱键’对应的键值对”。



# 6 应用场景

代码示例：

```java
/**
 * 弱引用HashMap测试
 * -XX:+PrintGC
 */
@Slf4j
public class WeakHashMapTest {
  public static void main(String[] args) throws InterruptedException {

      // 初始化3个“弱键”
      String w1 = new String("one");
      String w2 = new String("two");
      String w3 = new String("three");

      WeakHashMap weakHashMap = new WeakHashMap();
      weakHashMap.put(w1,"w1");
      weakHashMap.put(w2,"w2");
      weakHashMap.put(w3,"w3");
      log.info("WeakHashMap size：{}",weakHashMap.size());

      // 将w1设置null。
      // 这意味着“弱键”w1再没有被其它对象引用，调用gc时会回收WeakHashMap中与“w1”对应的键值对
      w1=null;
      Thread.sleep(1000);
      System.gc();

      // 在GC后进行诸如迭代等操作后，执行size方法就会调用expungeStaleEntries(); 进行清除弱建。
//      Iterator iter = weakHashMap.entrySet().iterator();
//      while (iter.hasNext()) {
//          Map.Entry en = (Map.Entry)iter.next();
//          System.out.printf("next : %s - %s\n",en.getKey(),en.getValue());
//      }

      // 这里在GC后马上调用size()方法，不一定能清除到弱建，因此此时ReferenceQueue队列中还不一定存在弱建对象。
      // 等待1s后，在调用size2，则会清除弱建了。
      log.info("WeakHashMap size1：{}",weakHashMap.size());
      Thread.sleep(1000);
      log.info("WeakHashMap size2：{}",weakHashMap.size());


      // 这种添加方式，"2","3"等不是弱建，无法GC回收。只有new String("1") 才是弱建，原因：？？？
      WeakHashMap weakHashMap1 = new WeakHashMap();
      weakHashMap1.put(new String("1"),"w1");
      weakHashMap1.put("2","w2");
      weakHashMap1.put("3","w3");
      log.info("WeakHashMap1 size：{}",weakHashMap1.size());
      Thread.sleep(1000);
      System.gc();
      Thread.sleep(1000);
      log.info("WeakHashMap1 size：{}",weakHashMap1.size());
  }
}
```





# 7 并发问题





# 8 循环迭代

```java
// 假设map是WeakHashMap对象
// map中的key是String类型，value是Integer类型
Integer integ = null;
Iterator iter = map.entrySet().iterator();
while(iter.hasNext()) {
    Map.Entry entry = (Map.Entry)iter.next();
    // 获取key
    key = (String)entry.getKey();
        // 获取value
    integ = (Integer)entry.getValue();
}
```



```java
// 假设map是WeakHashMap对象
// map中的key是String类型，value是Integer类型
String key = null;
Integer integ = null;
Iterator iter = map.keySet().iterator();
while (iter.hasNext()) {
        // 获取key
    key = (String)iter.next();
        // 根据key，获取value
    integ = (Integer)map.get(key);
}
```



```java
// 假设map是WeakHashMap对象
// map中的key是String类型，value是Integer类型
Integer value = null;
Collection c = map.values();
Iterator iter= c.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}
```



