# 1 概述

​		由于volatile是无法解决操作的原子性的，而synchronized和锁要达成原子操作时，多线程情况下，只有一个线程能操作，其他线程会处于阻塞状态，性能比较低。而下面要介绍的Atomic相关类，就是一些能保证原子性操作的工具类。这些类是基于CAS，即 Compare and Swap，其是JDK提供的非阻塞的原子性操作，是通过硬件来实现的比较-更新操作。

1. 基本类型: `AtomicInteger`, `AtomicLong`, `AtomicBoolean` ;
2. 数组类型: `AtomicIntegerArray`,` AtomicLongArray`, `AtomicReferenceArray` ;
3. 引用类型: `AtomicReference`, `AtomicStampedRerence`, `AtomicMarkableReference` ;
4. 对象的属性修改类型: `AtomicIntegerFieldUpdater`, `AtomicLongFieldUpdater`, `AtomicReferenceFieldUpdater` 。

​		这些类存在的目的是对相应的数据进行原子操作。所谓原子操作，是指操作过程不会被中断，保证数据操作是以原子方式进行的。

​		这些类的路径`java.util.concurrent.atomic`，每个Atomic类基本都有`boolean compareAndSet(expectedValue, updateValue);`，此方法通过原子实现CAS操作，最底层基本基于汇编语言实现。

* 以知当前内存里面的值current和预期要传的值（expectedValue）进行比较（==）

* 如果比价（==）相等，返回true，更新值为updateValue；如果不相等，返回false，值不改变。

  架构图如下:

![thread-005](..\images\thread-005.png)

# 2 基本类型

​		基本类型主要介绍下 AtomicLong，其他两个类似。AtomicLong是作用是对长整形进行原子操作。

​		在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

​		主要的原理是通过volatile和cas配合，完成原子操作：

* 通过将value设置为volatile类型。这保证了线程修改value的值时，其他线程看到的value值都是最新的value值，即修改之后的volatile的值。
* 通过CAS设置value。这保证了：当某线程通过CAS函数（compareAndSet函数）设置value时，它的操作是原子的，即线程在操作value时不会别中断。

## 2.1 成员变量	

```java
// 初始值
private volatile long value;
// Unsafe类，CAS就是基于此类实现的
private static final Unsafe unsafe = Unsafe.getUnsafe();
// value在内存地址中的偏移量
private static final long valueOffset;
```

## 2.2 构造函数

```java
// 构造函数
AtomicLong()
// 创建值为initialValue的AtomicLong对象
AtomicLong(long initialValue)
```

## 2.3 常用API

```java
// 以原子方式设置当前值为newValue(这里的原子方式是通过volatile实现的)。
final void set(long newValue) 

// 获取当前值
final long get() 

// 以原子方式将当前值减 1，并返回减1后的值。等价于“--num”
final long decrementAndGet() 

// 以原子方式将当前值减 1，并返回减1前的值。等价于“num--”
final long getAndDecrement() 

// 以原子方式将当前值加 1，并返回加1后的值。等价于“++num”
final long incrementAndGet() 

// 以原子方式将当前值加 1，并返回加1前的值。等价于“num++”
final long getAndIncrement()  
  
// 以原子方式将delta与当前值相加，并返回相加后的值。
final long addAndGet(long delta) 

// 以原子方式将delta添加到当前值，并返回相加前的值。
final long getAndAdd(long delta) 

// 如果当前值 == expect，则以原子方式将该值设置为update。成功返回true，否则返回false，并且不修改原值。
final boolean compareAndSet(long expect, long update)

// 以原子方式设置当前值为newValue，并返回旧值。
final long getAndSet(long newValue)

// 返回当前值对应的int值
int intValue() 

// 获取当前值对应的long值
long longValue()    

// 以 float 形式返回当前值
float floatValue()   
 
// 以 double 形式返回当前值
double doubleValue()  
  
// 最后设置为给定值。延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。
final void lazySet(long newValue)

// 如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。
final boolean weakCompareAndSet(long expect, long update)
```

## 2.4 源码解析

​		JDK1.8 中`getAndIncrement`，`incrementAndGet`都和之前有了不同的实现，例如之前JDK版本的`incrementAndGet`

```java
public final long incrementAndGet() {
    // 这里使用自旋，循环重试更新值
    for (;;) {
        // 获取AtomicLong当前对应的long值
        long current = get();
        // 将current加1
        long next = current + 1;
        // 通过CAS函数，更新current的值
        if (compareAndSet(current, next))
            return next;
    }
}

public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```

​		JDK1.8中`AtomicLong`中的自旋，改为在Unsafe类中处理

```java
public final long incrementAndGet() {
    return unsafe.getAndAddLong(this, valueOffset, 1L) + 1L;
}
```



```java
// 记录下层JVM是否支持无锁的long型CAS操作。当Unsafe.compareAndSwapLong在某个情况下工作，一些构造器需要在Java层面处理来避免锁住用于可见的锁。
static final boolean VM_SUPPORTS_LONG_CAS = VMSupportsCS8();

//返回下层JVM是否支持无锁的long型CAS操作。只调用一次并缓存在VM_SUPPORTS_LONG_CAS。
private static native boolean VMSupportsCS8();
```



# 3 数组类型

​		AtomicIntegerArray，AtomicLongArray， AtomicReferenceArray这3个数组类型的原子类的原理和用法相似。AtomicLong是作用是对长整形进行原子操作。而AtomicLongArray的作用则是对"长整形数组"进行原子操作，存储在AtomicLongArray中的数组元素能够以原子方式进行更新。

​		本身AtomicIntegerArray的变革不是原子的。

## 3.1 成员变量

```java
// unsafe变量是AutomicLongArray实现数组原子化更新的核心，数组元素的修改操作由unsafe的CAS相关操作完成
private static final Unsafe unsafe = Unsafe.getUnsafe();
// base是数组首个元素偏移地址
private static final int base = unsafe.arrayBaseOffset(long[].class);
// 数组元素的偏移量
private static final int shift;
// 保存元素的数组
private final long[] array;	
```
## 3.2 构造函数

```java
// 创建给定长度的新 AtomicLongArray。
AtomicLongArray(int length)
// 创建与给定数组具有相同长度的新 AtomicLongArray，并从给定数组复制其所有元素。
AtomicLongArray(long[] array)
```



## 3.3 常用API

```java
// 以原子方式将给定值添加到索引 i 的元素。
long addAndGet(int i, long delta)
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(int i, long expect, long update)
// 以原子方式将索引 i 的元素减1。
long decrementAndGet(int i)
// 获取位置 i 的当前值。
long get(int i)
// 以原子方式将给定值与索引 i 的元素相加。
long getAndAdd(int i, long delta)
// 以原子方式将索引 i 的元素减 1。
long getAndDecrement(int i)
// 以原子方式将索引 i 的元素加 1。
long getAndIncrement(int i)
// 以原子方式将位置 i 的元素设置为给定值，并返回旧值。
long getAndSet(int i, long newValue)
// 以原子方式将索引 i 的元素加1。
long incrementAndGet(int i)
// 最终将位置 i 的元素设置为给定值。
void lazySet(int i, long newValue)
// 返回该数组的长度。
int length()
// 将位置 i 的元素设置为给定值。
void set(int i, long newValue)
// 返回数组当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean    weakCompareAndSet(int i, long expect, long update)
```

## 3.4 源码解析

```java
static {
    // 获取数组元素的增量偏移
    int scale = unsafe.arrayIndexScale(long[].class);
    // 判断是不是2的倍数
    if ((scale & (scale - 1)) != 0)
    throw new Error("data type scale not a power of two");
    // 获取long型元素的偏移量
    shift = 31 - Integer.numberOfLeadingZeros(scale);
}
```

**checkAndByteOffset(int i)**

```java
// 主要用于判断索引值是否越界，若越界则抛出越界异常，否则计算索引值对应的元素在数组中的内存偏移量
private long checkedByteOffset(int i) {
    if (i < 0 || i >= array.length)
    throw new IndexOutOfBoundsException("index " + i);

    return byteOffset(i);
}
```

```java
// 根据索引下标i计算出每个元素的内存地址，元素内存地址=首地址偏移+每个元素的相对于数组偏移，每个数组元素相对数组的偏移量等于元素所占内存8*数组下标i。
private static long byteOffset(int i) {
	return ((long) i << shift) + base;
}
```

**long get(int i)**

```java
// 以原子方式获取数组指定索引位置元素
public final long get(int i) {
    // 首先调用checkedByteOffset方法校验数组的索引下标i，校验通过返回对应下标元素的内存地址
	return getRaw(checkedByteOffset(i));
}
// 调用了unsafe.getLongVolitile方法获取数组指定下标i的元素
private long getRaw(long offset) {
	return unsafe.getLongVolatile(array, offset);
}
```

**void set(int i, long value)**

```java
// 以原子方式设置数组指定下标位置值
public final void set(int i, long newValue) {
	unsafe.putLongVolatile(array, checkedByteOffset(i), newValue);
}
```

 **long incrementAndGet(int i)** 

```java
// 以原子方式在指定下标i元素加1，返回更新后的值
public final long incrementAndGet(int i) {
	return getAndAdd(i, 1) + 1;
}
// 调用unsafe的getAndAddLong对该位置元素使用CAS操作更新，注意此时返回的long值是更新前的所以我们在调用处incrementAndGet方法还要加1。
public final long getAndAdd(int i, long delta) {
    return unsafe.getAndAddLong(array, checkedByteOffset(i), delta);
}
```

 **void lazySet**

```java
// 更新数组指定下标i处的值为newValue，与set的区别在于它是延时而不是立即更新，适用于对实时性要求不高的场景
public final void lazySet(int i, long newValue) {
    unsafe.putOrderedLong(array, checkedByteOffset(i), newValue);
}
```

# 4 引用类型

​		AtomicReference提供了一个可以被原子性读和写的对象引用变量。原子性的意思是多个想要改变同一个AtomicReference的线程不会导致AtomicReference处于不一致的状态。

​	    实现原理与AtomicLong类似，只不过这里换为了对象引用。

## 4.1 成员变量

```java
// unsafe变量
private static final Unsafe unsafe = Unsafe.getUnsafe();
//对象引用在内存中的偏移量
private static final long valueOffset;
//对象引用
private volatile V value;
```

## 4.2 构造函数

```java
// 使用 null 初始值创建新的 AtomicReference。
AtomicReference()
// 使用给定的初始值创建新的 AtomicReference。
AtomicReference(V initialValue)
```

## 4.3 常用API

```java
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(V expect, V update)
// 获取当前值。
V get()
// 以原子方式设置为给定值，并返回旧值。
V getAndSet(V newValue)
// 最终设置为给定值。
void lazySet(V newValue)
// 设置为给定值。
void set(V newValue)
// 返回当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean weakCompareAndSet(V expect, V update)
```

## 4.4 源码解析

​		这里的源码逻辑与AtomicLong查不多，只不过使用的对象引用的内存偏移量，故而不做详细介绍。



# 5 对象的属性修改类型

​		基本原子类和数组原子类在最初设计编码时候就已经考虑到了需要保证原子性。但是往往有很多情况就是，由于需求的更改，原子性需要在后面加入，类似于我不要求你这整个类操作具有原子性，只要求类里面一个字段操作具有原子性。所以，`AtomicIntegerFiledUpdater`、`AtomicLongFieldUpdater`和`AtomicReferenceFieldUpdater`就是这个作用。

​		 这里以`AtomicLongFieldUpdater`进行介绍：

​		`AtomicLongFiledUpdater`就是用来更新某一个实例对象里面的Long属性的，但是需要注意的是，用法上有如下规则：

* 字段必须是volatile类型的，在线程之间共享变量时保证立即可见。
* 字段的描述类型（修饰符public/protected/default/private）是与调用者与操作对象字段的关系一致。也就是说调用者能过直接操作对象字段，那么就可以反射进行原子操作。
* 对于父类的字段，子类时不能直接操作的，尽管子类可以访问父类的字段。
* 只能是实例变量，不能是类变量，也就是说不能加static关键字
* 只能是可修改变量，不能使final变量，因为final的语义就是不可修改的
* 对于AtomicIntegerFiledUpdater和AtomicLongFiledUpdater只能修改int/long类型的字段，不能修改其包装类型（Integer/Long）。如果要修改包装类型就需要使用AtomicReferenceFiledUpdater。

## 5.1 成员变量



## 5.2 构造函数

```java
// 受保护的无操作构造方法，供子类使用。
protected AtomicLongFieldUpdater()
```



## 5.3 常用API

```java
// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long addAndGet(T obj, long delta)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean compareAndSet(T obj, long expect, long update)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long decrementAndGet(T obj)
// 获取此更新器管理的在给定对象的字段中保持的当前值。
abstract long get(T obj)
// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long getAndAdd(T obj, long delta)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long getAndDecrement(T obj)
// 以原子方式将此更新器管理的给定对象字段的当前值加 1。
long getAndIncrement(T obj)
// 将此更新器管理的给定对象的字段以原子方式设置为给定值，并返回旧值。
long getAndSet(T obj, long newValue)
// 以原子方式将此更新器管理的给定对象字段当前值加 1。
long incrementAndGet(T obj)
// 最后将此更新器管理的给定对象的字段设置为给定更新值。
abstract void lazySet(T obj, long newValue)
// 为对象创建并返回一个具有给定字段的更新器。
static <U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName)
// 将此更新器管理的给定对象的字段设置为给定更新值。
abstract void set(T obj, long newValue)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean weakCompareAndSet(T obj, long expect, long update)
```

## 5.4 源码分析

​		AtomicLongFieldUpdater是一个抽象类，但是它内部有一个private final类型的默认子类，所以在调用newUpdater的时候，会用模式子类来实现：

```java
public static <U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass,String fieldName) {
    Class<?> caller = Reflection.getCallerClass();
    if (AtomicLong.VM_SUPPORTS_LONG_CAS)
        return new CASUpdater<U>(tclass, fieldName, caller);
    else
        return new LockedUpdater<U>(tclass, fieldName, caller);
}
```

​		newUpdater()的作用是获取一个AtomicLongFieldUpdater类型的对象。它实际返回的是CASUpdater对象，或者LockedUpdater对象；具体返回哪一个类取决于JVM是否支持long类型的CAS函数。CASUpdater和LockedUpdater都是AtomicIntegerFieldUpdater的子类，它们的实现类似。下面以CASUpdater来进行说明。

​		CASUpdater类的源码如下：

```java
public boolean compareAndSet(T obj, long expect, long update) {
    if (obj == null || obj.getClass() != tclass || cclass != null) fullCheck(obj);
    return unsafe.compareAndSwapLong(obj, offset, expect, update);
}
```

​		它实际上是，首先通过反射获得指定的字段，然后通过CAS函数操作。如果类的long对象的值是expect，则设置它的值为update。 



# 6 ABA问题

​			在前面介绍了`AtomicInteger`和`Atomiclong`的操作，但是在这两个类在CAS操作的时候会遇到**ABA问题**：简单来将就是多线程环境下，2次读写中一个线程修改A->B，然后又B->A，另一个线程看到的值未改变，又继续修改成自己的期望值。当然如果我们不关心过程，只关心结果，那么这个就是无所谓的ABA问题。为了解决ABA问题，Java为我们提供了`AtomicStampedReference`和`AtomicMarkableReference`类，两者的区别是：`AtomicStampedReference`侧重你改变了几次值，`AtomicMarkableReference`侧重你是否改变了值。

```java
/**
 * ABA解决问题测试
 */
public class AtomicStampReferenceTest {

    private static AtomicInteger atomicInt = new AtomicInteger(100);
    private static AtomicStampedReference atomicStampedRef = new AtomicStampedReference(100, 0);

    public static void main(String[] args) {

        testABACASByNormal();

        testABACAS();
    }

    /**
     * 测试使用AtomicInteger时ABA问题发生后是否CAS成功
     * 结果：可以更新成功，不会处理ABA问题
     */
    private static void testABACASByNormal(){
        Thread intT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                atomicInt.compareAndSet(100, 101);
                atomicInt.compareAndSet(101, 100);
            }
        });

        Thread intT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                }
                boolean c3 = atomicInt.compareAndSet(100, 101);
                System.out.println("使用AtomicInteger后ABA是否更新："+c3);
            }
        });

        intT1.start();
        intT2.start();
        try {
            intT1.join();
            intT2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }


    /**
     * 测试ABA后是否可以更新CAS
     * 结果：不可更新成功，ABA后 Pair对象发生了变化，不能更新
     */
    private static void testABACAS() {
        Thread refT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                }
                atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
                atomicStampedRef.compareAndSet(101, 100, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
            }
        });

        Thread refT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                int stamp = atomicStampedRef.getStamp();
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                }
                boolean c3 = atomicStampedRef.compareAndSet(100, 101, stamp, stamp + 1);
                System.out.println("使用AtomicStampedReference后ABA是否更新："+c3);
            }
        });

        refT1.start();
        refT2.start();
    }
}
```

​		**注意**:AtomicMarkableReference它并不能解决ABA的问题 ，它是通过一个boolean来标记是否更改，本质就是只有true和false两种版本来回切换，只能降低ABA问题发生的几率，并不能阻止ABA问题的发生。

# 7 Unsafe类

​		JDK的rt.jar包中的Unsafe类提供了硬件级别的原子操作，Unsafe类的方法都是native方法，它们使用JNI方式访问本地C++实现库。下面根据其他文档的介绍，简单说下几个常用的API。

```java
// 返回指定的变量在所属类中的内存偏移地址，该偏移地址仅仅在Unsafe函数中访问指定字段时使用。
long objectFieldOffset(Field field);

// 获取数组中第一个元素的地址
int arrayBaseOffset(Class<?> arrayClass);

// 获取数组中一个元素占用的字节
int arrayIndexScale(Class<?> arrayClass);

// 比较对象obj中偏移量为offset的变量的值是否与expect相等，相等则使用update值更新，然后返回true，否则返回false
boolean compareAndSwapLong(Object obj, long offset, long expect, long update);

// 获取对象obj中偏移量为offset的变量对应volatile语义的值
long getLongVolatile(Object obj,long offset);

// 设置obj对象中offset偏移的类型为long的field的值为value，支持volatile语义
void putLongvolatile(Object obj,long offset,long value);

// 设置obj对象中offset偏移地址对应的long型field的值为value。这是一个有延迟的putLongvolatile方法，并且不能保证值修改对其他线程立刻可见。只有变量使用volatile修饰并且预计会被意外修改时才使用该方法
void putOrderedLong(Object obj,long offset,long value);

// 阻塞当前线程，其中参数isAbsolute等于false且time=0表示一直阻塞。time大于0表示等待指定的time后阻塞线程会被唤醒，这个time是个相似值，是个增量值，也就是相对当前时间累加time后当前线程就会被唤醒。如果isAbsolute等于true，并且tiem大于0，表示阻塞的线程到指定的时间点后会被唤醒，这里time是个绝对时间，是将某个时间点换算为ms后的值。另外，当其他线程调用了当前阻塞线程的iterrupter方法而中断了当前线程时，当前线程也会返回，而当其他线程调用了unPark方法并且把当前线程作为参数时当前线程也会返回
void park(boolean isAbsolute,long time);

// 唤醒调用park后阻塞的线程。
void unpark(Object thread);

// ------->JDK1.8 新增举例
// 获取对象obj中偏移量为offset的变量volatile语义的当前值，并设置变量volatile语义的值为update
long getAndSetLong(Object obj,long offset,long update);
// 获取对象obj中偏移量为offset的变量volatile语义的当前值，并设置变量值为原始值+addValue
long getAndAddLong(Object obj,long offset,long addValue);
```

# 8 LongAdder

​		JDK中原本提供的原子类是通过非阻塞算法来实现原子性的，比synchronized和锁的阻塞式算法已经提示了很大。但是如果遭遇到高并发时，还是会有大量的线程会竞争同一个原子变量，这时只会有1个线程可以CAS生成，其他的线程则会自旋重试，这样还是会造成大量的CPU浪费。

​		JDK8新增了一个原子性递增或递减类LongAdder来克服高并发下使用AtomicLong类的缺点。

##  8.1 原理

​		既然AtomicLong的性能瓶颈是由于过多线程去竞争一个变量的更新而产生，那么把一个变量分解为多个变量，让同样多的线程去竞争多个资源，就可以解决问题了。

​		