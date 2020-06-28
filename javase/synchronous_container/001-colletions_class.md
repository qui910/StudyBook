# 1 同步容器

​		在Java的集合容器框架中，主要有四大类别：List、Set、Queue、Map。如果有多个线程并发地访问这些容器时，就会出现问题。因此，在编写程序时，必须要求程序员手动地在任何访问到这些容器的地方进行同步处理，这样导致在使用这些容器的时候非常地不方便。

​		所以，Java提供了同步容器供用户使用。在Java中，同步容器主要包括2类：

1）Vector、Stack、HashTable

2）Collections类中提供的静态工厂方法创建的类

　　Vector实现了List接口，Vector实际上就是一个数组，和ArrayList类似，但是Vector中的方法都是synchronized方法，即进行了同步措施。

　　Stack也是一个同步容器，它的方法也用synchronized进行了同步，它实际上是继承于Vector类。

　　HashTable实现了Map接口，它和HashMap很相似，但是HashTable进行了同步处理，而HashMap没有。

　　Collections类是一个工具提供类，注意，它和Collection不同，Collection是一个顶层的接口。在Collections类中提供了大量的方法，比如对集合或者容器进行排序、查找等操作。最重要的是，在它里面提供了几个静态工厂方法来创建同步容器类:

# 2 同步容器的缺陷

​		从同步容器的具体实现源码可知，同步容器中的方法采用了synchronized进行了同步，那么很显然，这必然会影响到执行性能，另外，同步容器不一定是真正地完全线程安全。

## 2.1 性能问题
​		进行同样多的插入操作，Vector的耗时是ArrayList的两倍。另外，由于Vector中的add方法和get方法都进行了同步，因此，在有多个线程进行访问时，如果多个线程都只是进行读取操作，那么每个时刻就只能有一个线程进行读取，其他线程便只能等待，这些线程必须竞争同一把锁。

​		因此为了解决同步容器的性能问题，在Java 1.5中提供了并发容器，位于java.util.concurrent目录下，并发容器的相关知识将在后续讲述。

## 2.2 同步容器不线程安全

​		同步容器类在单个方法被使用时可以保证线程安全。复合操作则需要额外的客户端加锁来保护。

看下面这段代码：

```java
public class VectorTest {
    private static Vector<Integer> vector=new Vector();
    public static void main(String[] args) {
        // 多线程情况下使用Vector 出现线程不安全的情况。
        // removeThread 中同时使用了size() 和 remove(i) 两个同步方式，导致操作不是原子的。
        // printThread 类似。如果要解决不安全情况，需要在for 循环外加把锁
        while(true){
            for(int i=0;i<10;i++){
                vector.add(i); //往vector中添加元素
            }
            Thread removeThread=new Thread(()->{
                //获取vector的大小
                for(int i=0;i<vector.size();i++){
                    //当前线程让出CPU,使例子中的错误更快出现
                    Thread.yield();
                    //移除第i个数据
                    vector.remove(i);
                }
            });
            Thread printThread=new Thread(()->{
                //获取vector的大小
                for(int i=0;i<vector.size();i++){
                    //当前线程让出CPU,使例子中的错误更快出现
                    Thread.yield();
                    //获取第i个数据并打印
                    System.out.println(vector.get(i));
                }
            });
            removeThread.start();
            printThread.start();
            //避免同时产生过多线程
            while(Thread.activeCount()>20);
        }
    }
}
```

## 2.3 ConcurrentModificationException异常
​		在对Vector等容器并发地进行迭代修改时，会报ConcurrentModificationException异常，即fail-fast异常。但是在并发容器中不会出现这个问题。

# 3 Collections同步容器

​		接下来重点讲述下Collections中的同步容器。在源码中，针对3.1 和 3.2 这些使用方法，如果方法的集合参数的大小 ，大于18则内部使用迭代器作为读取方式。否则如果集合是可随机读取，则使用get(i)进行操作。

## 3.1 排序操作

```java
// 反转  时间复杂度O(n)
public static void reverse(List<?> list) 
// 随机排序
public static void shuffle(List<?> list) 
// 按自然排序的升序排序
public static <T extends Comparable<? super T>> void sort(List<T> list)
// 定制排序，由Comparator控制排序逻辑
public static <T> void sort(List<T> list, Comparator<? super T> c)
// 交换两个索引位置的元素
public static void swap(List<?> list, int i, int j)
// 旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。
public static void rotate(List<?> list, int distance)
```

## 3.2 查找，替换操作

```java
// 对List进行二分查找，返回索引，注意List必须是有序的
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)
// 根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)

// 根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp)    

// 用元素obj填充list中所有元素(替换原有元素)
public static <T> void fill(List<? super T> list, T obj) 

// 统计元素出现次数
int frequency(Collection c, Object o)

// 统计targe在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target).
int indexOfSubList(List list, List target),

// 用新元素替换旧元素。
boolean replaceAll(List list, Object oldVal, Object newVal)
```

## 3.3 同步控制

​		Collections中几乎对每个集合都定义了同步控制方法，例如 SynchronizedList(), SynchronizedSet()等方法，来将集合包装成线程安全的集合。

```java
public static <T> Collection<T> synchronizedCollection(Collection<T> c);

public static <T> Set<T> synchronizedSet(Set<T> s);

public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s);

public static <T> List<T> synchronizedList(List<T> list);

public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
```

​		这些同步方法都返回了Collections的内部类对象，以List举例：`synchronizedList`返回`SynchronizedRandomAccessList`或`SynchronizedList`。最终都是由`SynchronizedList`实现的，而`SynchronizedList`类的原理就是将List接口的每个方法都用``Synchronized`关键字修饰，保证单方法操作时的同步性。

​		如果遇到复合操作，如2.2 ，会提示错误，这就要求必须在一个`Synchronized`块中进行复合操作。

```java
        List list = Collections.synchronizedList(new ArrayList());
            ...
        synchronized (list) {
            Iterator i = list.iterator(); // Must be in synchronized block
            while (i.hasNext())
                foo(i.next());
        }
```

下面是复合操作非线程安全示例：

```java
        List<String> arrayList = Collections.synchronizedList(new ArrayList<String>());
        while(true){
            for(int i=0;i<10;i++){
                arrayList.add(i+"");
            }
            Thread removeThread=new Thread(()->{
                //获取vector的大小
                for(int i=0;i<arrayList.size();i++){
                    //当前线程让出CPU,使例子中的错误更快出现
                    Thread.yield();
                    //移除第i个数据
                    arrayList.remove(i);
                }
            });
            Thread printThread=new Thread(()->{
                //获取vector的大小
                for(int i=0;i<arrayList.size();i++){
                    //当前线程让出CPU,使例子中的错误更快出现
                    Thread.yield();
                    //获取第i个数据并打印
                    System.out.println(arrayList.get(i));
                }
            });
            removeThread.start();
            printThread.start();
            //避免同时产生过多线程
            while(Thread.activeCount()>20);
        }
```

运行结果

```java
6
3
Exception in thread "Thread-9845" java.lang.IndexOutOfBoundsException: Index: 12, Size: 11
	at java.util.ArrayList.rangeCheck(ArrayList.java:657)
	at java.util.ArrayList.get(ArrayList.java:433)
	at java.util.Collections$SynchronizedList.get(Collections.java:2417)
	at com.prd.colletctions.CollectionsTest.lambda$main$1(CollectionsTest.java:32)
	at java.lang.Thread.run(Thread.java:748)
```

## 3.4 不可变（只读）集合

​		Collections提供了三类方法返回一个不可变集合，

* emptyXXX(),返回一个空的只读集合
* singleXXX()，返回一个只包含指定对象，只有一个元素，只读的集
* unmodifiablleXXX()，返回指定集合对象的只读视图。