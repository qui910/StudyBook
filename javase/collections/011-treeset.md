# 1 概述

* 是一个有序的集合，它的作用是提供有序的Set集合。它继承于AbstractSet抽象类，实现了NavigableSet<E>, Cloneable, java.io.Serializable接口。
* 继承于AbstractSet，所以它是一个Set集合，具有Set的属性和方法。
* 实现了NavigableSet接口，意味着它支持一系列的导航方法。比如查找与指定目标最匹配项。
* 实现了Cloneable接口，意味着它能被克隆。
* java.io.Serializable接口，意味着它支持序列化。



​		TreeSet是基于TreeMap实现的。TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序。这取决于使用的构造方法。

​		TreeSet为基本操作（add、remove 和 contains）提供受保证的 log(n) 时间开销。

​		另外，TreeSet是非同步的。 它的iterator 方法返回的迭代器是fail-fast的。

# 2 成员变量

```java
    // NavigableMap对象
    private transient NavigableMap<E,Object> m;

    // TreeSet是通过TreeMap实现的，
    // PRESENT是键-值对中的值。
    private static final Object PRESENT = new Object();
```



# 3 常用API

```java
// 构造一个空树集。
TreeSet()
// 带比较器的构造函数。
TreeSet(Comparator<? super E> comparator)
// 构造一个树集， 并增加一个集合或有序集中的所有元素（对于后一种情况， 要使用同样的顺序。)
TreeSet(Collection<? extends E> elements)
TreeSet(SortedSet<E> s)
// 返回用于对元素进行排序的比较器。 如果元素用 Comparable 接口的 compareTo方法进行比较则返回 null。
Comparator<? super E> comparator()
// 返回有序集中的最小元素或最大元素。
E firstO
E 1ast()
// 返回大于 value 的最小元素或小于 value 的最大元素，如果没有这样的元素则返回 null。
E higher(E value)
E lower(E value)
// 返回大于等于 value 的最小元素或小于等于 value 的最大元素， 如果没有这样的元素则返回 null。
E ceiling(E value)
E floor(E value)
// 删除并返回这个集中的最大元素或最小元素， 这个集为空时返回 null。
E pollFirst()
E pol 1Last
// 返回一个按照递减顺序遍历集中元素的迭代器。
Iterator<E> descendingIterator()
```

# 4 源码解析

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    // NavigableMap对象
    private transient NavigableMap<E,Object> m;

    // TreeSet是通过TreeMap实现的，
    // PRESENT是键-值对中的值。
    private static final Object PRESENT = new Object();

    // 不带参数的构造函数。创建一个空的TreeMap
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }

    // 将TreeMap赋值给 "NavigableMap对象m"
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }

    // 带比较器的构造函数。
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<E,Object>(comparator));
    }

    // 创建TreeSet，并将集合c中的全部元素都添加到TreeSet中
    public TreeSet(Collection<? extends E> c) {
        this();
        // 将集合c中的元素全部添加到TreeSet中
        addAll(c);
    }

    // 创建TreeSet，并将s中的全部元素都添加到TreeSet中
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }

    // 返回TreeSet的顺序排列的迭代器。
    // 因为TreeSet时TreeMap实现的，所以这里实际上时返回TreeMap的“键集”对应的迭代器
    public Iterator<E> iterator() {
        return m.navigableKeySet().iterator();
    }

    // 返回TreeSet的逆序排列的迭代器。
    // 因为TreeSet时TreeMap实现的，所以这里实际上时返回TreeMap的“键集”对应的迭代器
    public Iterator<E> descendingIterator() {
        return m.descendingKeySet().iterator();
    }

    // 返回TreeSet的大小
    public int size() {
        return m.size();
    }

    // 返回TreeSet是否为空
    public boolean isEmpty() {
        return m.isEmpty();
    }

    // 返回TreeSet是否包含对象(o)
    public boolean contains(Object o) {
        return m.containsKey(o);
    }

    // 添加e到TreeSet中
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }

    // 删除TreeSet中的对象o
    public boolean remove(Object o) {
        return m.remove(o)==PRESENT;
    }

    // 清空TreeSet
    public void clear() {
        m.clear();
    }

    // 将集合c中的全部元素添加到TreeSet中
    public  boolean addAll(Collection<? extends E> c) {
        // Use linear-time version if applicable
        if (m.size()==0 && c.size() > 0 &&
            c instanceof SortedSet &&
            m instanceof TreeMap) {
            SortedSet<? extends E> set = (SortedSet<? extends E>) c;
            TreeMap<E,Object> map = (TreeMap<E, Object>) m;
            Comparator<? super E> cc = (Comparator<? super E>) set.comparator();
            Comparator<? super E> mc = map.comparator();
            if (cc==mc || (cc != null && cc.equals(mc))) {
                map.addAllForTreeSet(set, PRESENT);
                return true;
            }
        }
        return super.addAll(c);
    }

    // 返回子Set，实际上是通过TreeMap的subMap()实现的。
    public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                                  E toElement,   boolean toInclusive) {
        return new TreeSet<E>(m.subMap(fromElement, fromInclusive,
                                       toElement,   toInclusive));
    }

    // 返回Set的头部，范围是：从头部到toElement。
    // inclusive是是否包含toElement的标志
    public NavigableSet<E> headSet(E toElement, boolean inclusive) {
        return new TreeSet<E>(m.headMap(toElement, inclusive));
    }

    // 返回Set的尾部，范围是：从fromElement到结尾。
    // inclusive是是否包含fromElement的标志
    public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
        return new TreeSet<E>(m.tailMap(fromElement, inclusive));
    }

    // 返回子Set。范围是：从fromElement(包括)到toElement(不包括)。
    public SortedSet<E> subSet(E fromElement, E toElement) {
        return subSet(fromElement, true, toElement, false);
    }

    // 返回Set的头部，范围是：从头部到toElement(不包括)。
    public SortedSet<E> headSet(E toElement) {
        return headSet(toElement, false);
    }

    // 返回Set的尾部，范围是：从fromElement到结尾(不包括)。
    public SortedSet<E> tailSet(E fromElement) {
        return tailSet(fromElement, true);
    }

    // 返回Set的比较器
    public Comparator<? super E> comparator() {
        return m.comparator();
    }

    // 返回Set的第一个元素
    public E first() {
        return m.firstKey();
    }

    // 返回Set的最后一个元素
    public E first() {
    public E last() {
        return m.lastKey();
    }

    // 返回Set中小于e的最大元素
    public E lower(E e) {
        return m.lowerKey(e);
    }

    // 返回Set中小于/等于e的最大元素
    public E floor(E e) {
        return m.floorKey(e);
    }

    // 返回Set中大于/等于e的最小元素
    public E ceiling(E e) {
        return m.ceilingKey(e);
    }

    // 返回Set中大于e的最小元素
    public E higher(E e) {
        return m.higherKey(e);
    }

    // 获取第一个元素，并将该元素从TreeMap中删除。
    public E pollFirst() {
        Map.Entry<E,?> e = m.pollFirstEntry();
        return (e == null)? null : e.getKey();
    }

    // 获取最后一个元素，并将该元素从TreeMap中删除。
    public E pollLast() {
        Map.Entry<E,?> e = m.pollLastEntry();
        return (e == null)? null : e.getKey();
    }

    // 克隆一个TreeSet，并返回Object对象
    public Object clone() {
        TreeSet<E> clone = null;
        try {
            clone = (TreeSet<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }

        clone.m = new TreeMap<E,Object>(m);
        return clone;
    }

    // java.io.Serializable的写入函数
    // 将TreeSet的“比较器、容量，所有的元素值”都写入到输出流中
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        s.defaultWriteObject();

        // 写入比较器
        s.writeObject(m.comparator());

        // 写入容量
        s.writeInt(m.size());

        // 写入“TreeSet中的每一个元素”
        for (Iterator i=m.keySet().iterator(); i.hasNext(); )
            s.writeObject(i.next());
    }

    // java.io.Serializable的读取函数：根据写入方式读出
    // 先将TreeSet的“比较器、容量、所有的元素值”依次读出
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden stuff
        s.defaultReadObject();

        // 从输入流中读取TreeSet的“比较器”
        Comparator<? super E> c = (Comparator<? super E>) s.readObject();

        TreeMap<E,Object> tm;
        if (c==null)
            tm = new TreeMap<E,Object>();
        else
            tm = new TreeMap<E,Object>(c);
        m = tm;

        // 从输入流中读取TreeSet的“容量”
        int size = s.readInt();

        // 从输入流中读取TreeSet的“全部元素”
        tm.readTreeSet(size, s, PRESENT);
    }

    // TreeSet的序列版本号
    private static final long serialVersionUID = -2479143000061671589L;
}
```

* TreeSet实际上是TreeMap实现的。当我们构造TreeSet时；若使用不带参数的构造函数，则TreeSet的使用自然比较器；若用户需要使用自定义的比较器，则需要使用带比较器的参数。
* TreeSet是非线程安全的。
* TreeSet实现java.io.Serializable的方式。当写入到输出流时，依次写入“比较器、容量、全部元素”；当读出输入流时，再依次读取。

# 5 应用场景





# 6 并发问题

代码示例：

```
JavaLearning：com.prd.colletctions.TreeSetTest
```



# 7 循环迭代

## 7.1 Iterator顺序遍历

```java
for(Iterator iter = set.iterator(); iter.hasNext(); ) { 
    iter.next();
}   
```

## 7.2 Iterator顺序遍历

```java
// 假设set是TreeSet对象
for(Iterator iter = set.descendingIterator(); iter.hasNext(); ) { 
    iter.next();
}
```

## 7.3 for-each遍历HashSet

```java
// 假设set是TreeSet对象，并且set中元素是String类型
String[] arr = (String[])set.toArray(new String[0]);
for (String str:arr)
    System.out.printf("for each : %s\n", str);
```


​		**TreeSet不支持快速随机遍历，只能通过迭代器进行遍历！**