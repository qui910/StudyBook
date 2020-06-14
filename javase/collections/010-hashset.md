# 1 概述

​		链表和数组列表可以按照一定的规则进行元素排序，如果要查找某个特定的元素就需要从整个列表中搜索。

​		散列表可以帮助我们快速定位到所需的对象。散列表为每个对象计算一个整数，称为散列码（hash code）。散列码是由对象的实例域产生的一个整数，拥有不同数据域的对象将产生不同的散列码。

​		注意：自己实现hashCode方法应该与equals方法兼容，即如果`a.equlas(b)`为true，a与b必须具有相同的散列码

​		在Java中散列表有链表实现，每个列表为桶，散列码与桶的总数求余得到的结果就是桶数。

![010-1](..\images\010-1.png)

​		有时遇到桶被占满的情况，称之为散列冲突（hash collision）。这时，需要用新对象与桶中的所有对象进行比较，查看这个对象是否已经存在。

​		如果要更好的控制散列表的运行性能，要指定一个合理的初始桶数。桶数是指用于收集具有相同散列值的桶的数目。通常认为桶数应该预计元素个数的75%~150%。

​		HashSet 是一个**没有重复元素的集合**。

​		它是由HashMap实现的，**不保证元素的顺序**，而且**HashSet允许使用 null 元素**

​		HashSet是**非同步的**。如果多个线程同时访问一个哈希 set，而其中至少一个线程修改了该 set，那么它必须 保持外部同步。这通常是通过对自然封装该 set 的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 Collections.synchronizedSet 方法来“包装” set。最好在创建时完成这一操作，以防止对该 set 进行意外的不同步访问：

```java
Set s = Collections.synchronizedSet(new HashSet(...));
```

​		HashSet通过iterator()返回的**迭代器是fail-fast的。**

# 2 成员变量

```java
// HashSet底层实际是用HashMap保存
private transient HashMap<E,Object> map;
// HashMap保存的Value，即空对象
private static final Object PRESENT = new Object();
```



# 3 常量API

```java
// 构造一个空散列表。
HashSet( )
// 构造一个散列集， 并将集合中的所有元素添加到这个散列集中。
HashSet( Col 1 ection<? extends E> elements )
// 构造一个空的具有指定容量（桶数）的散列集。
HashSet( int initialCapacity )
// 构造一个具有指定容量和装填因子（一个 0.0 ~ 1.0 之间的数值， 确定散列表填充的百分比， 当大于这个百分比时， 散列表进行再散列）的空散列集
HashSet(int initialCapacity , float 1 oadFactor )
// 返回这个对象的散列码。散列码可以是任何整数， 包括正数或负数。equals 和 hashCode的定义必须兼容，即如果 x.equals(y) 为 true, x.hashCodeO 必须等于 y.hashCodeO。
int hashCode( )
```



# 4 源码解析

​		底层借用HashMap实现，无特殊讲解。

```java
    // 带集合的构造函数
    public HashSet(Collection<? extends E> c) {
        // 创建map。
        // 为什么要调用Math.max((int) (c.size()/.75f) + 1, 16)，从 (c.size()/.75f) + 1 和 16 中选择一个比较大的树呢？
        // 首先，说明(c.size()/.75f) + 1
        //   因为从HashMap的效率(时间成本和空间成本)考虑，HashMap的加载因子是0.75。
        //   当HashMap的“阈值”(阈值=HashMap总的大小*加载因子) < “HashMap实际大小”时，
        //   就需要将HashMap的容量翻倍。
        //   所以，(c.size()/.75f) + 1 计算出来的正好是总的空间大小。
        // 接下来，说明为什么是 16 。
        //   HashMap的总的大小，必须是2的指数倍。若创建HashMap时，指定的大小不是2的指数倍；
        //   HashMap的构造函数中也会重新计算，找出比“指定大小”大的最小的2的指数倍的数。
        //   所以，这里指定为16是从性能考虑。避免重复计算。
        map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));
        // 将集合(c)中的全部元素添加到HashSet中
        addAll(c);
    }
```





# 5 应用场景

​		散列将元素分散在表的各个位置上， 所以访问它们的顺序几乎是随机的。只有不关心集合中元素的顺序时才应该使用HashSet。



# 6 并发问题

## 6.1 多线程同时操作,会导致数据不一致

```java
        // 多线程同时写入,会导致数据不一致
        new Thread(()->{
            try {
                for (int i=0;i<100;i++) {
                    Thread.sleep(100);
                    set1.add(i);
                }
            } catch (InterruptedException e) {
                log.error(e.getMessage(),e);
            }
            log.info("thread1 down");
        }).start();
        new Thread(()->{
            for (int i=0;i<100;i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                set1.add(i);
            }
            log.info("thread2 down");
        }).start();
        Thread.sleep(20000);
        log.info("HashSet 最后大小为:{}",set1.size());
```

# 7 循环迭代

## 7.1  通过Iterator遍历HashSet

```java
for(Iterator iterator = set.iterator();
       iterator.hasNext(); ) { 
    iterator.next();
}
```

## 7.2 通过for-each遍历HashSet

```java
String[] arr = (String[])set.toArray(new String[0]);
for (String str:arr)
    System.out.printf("for each : %s\n", str);
```



