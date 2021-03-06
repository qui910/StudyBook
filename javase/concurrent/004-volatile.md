# 1 概述

​		Java语言规范第三版中对volatile的定义如下： Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

​		volatile是一种同步机制，比synchronized或者lock相关类更轻量，因为适用volatile并不会发生上下文切换等开销很大的行为。

​		如果一个变量修饰为volatile，那么JVM就知道了这变量可能会被并发修改。

​		但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的**原子保护**，volatile仅在很有限的场景下才能发挥作用。



## 1.1 适用场合

不适用：a++

**适用场合1（纯赋值操作）**：状态标记 boolean flag，如果一个共享变量自始至终只被**各个线程赋值**，而没有其他的操作，那么久可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。这里的flag的改变不能依赖之前的旧值，否则即使是boolean也是有问题的。

**适用场合2**：作为刷新之前变量的触发器。示例可以参考 第2章 6.3.2.1，volatile b

```java
Map configOptions;
char[] configText;
volatile booean initialized = fase;
// Thread A
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfigOptions(configText,configOptions);
// Thread B
while(!initialized)
    sleep();
// use configOptions
```



**适用场合3**：double check。



## 1.2 volatile作用

可见性：读一个volatile变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个volatile属性会立即刷入到主内存。

禁止指令重排序优化：解决单例双重锁乱序问题。

## 1.3 volatile和synchronized

volatile在这方面可以看做时轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全。

# 2 内存模型

​		在第二章中说明的关于内存不一致导致线程并发安全的问题，主要的原理就是CPU缓存所造成的。下面就重点讲解下这部分。

## 2.1 操作系统语义

​		计算机在运行程序时，每条指令都是在CPU中执行的，在执行过程中势必会涉及到数据的读写。我们知道程序运行的数据是存储在主存中，这时就会有一个问题，读写主存中的数据没有CPU中执行指令的速度快，如果任何的交互都需要与主存打交道则会大大影响效率，所以就有了CPU高速缓存。CPU高速缓存为某个CPU独有，只与在该CPU运行的线程有关。

​		有了CPU高速缓存虽然解决了效率问题，但是它会带来一个新的问题：**数据一致性**。在程序运行中，会将运行所需要的数据复制一份到CPU高速缓存中，在进行运算时CPU不再也主存打交道，而是直接从高速缓存中读写数据，只有当运行结束后才会将数据刷新到主存中。

例如：

```java
i++
```

​		当线程运行这段代码时，首先会从主存中读取i( i = 1)，然后复制一份到CPU高速缓存中，然后CPU执行 + 1 （2）的操作，然后将数据（2）写入到告诉缓存中，最后刷新到主存中。其实这样做在单线程中是没有问题的.

​		有问题的是在多线程中，假如有两个线程A、B都执行这个操作（i++），按照我们正常的逻辑思维主存中的i值应该=3，但事实是这样么？分析如下：

​		两个线程从主存中读取i的值（1）到各自的高速缓存中，然后线程A执行+1操作并将结果写入高速缓存中，最后写入主存中，此时主存i==2,线程B做同样的操作，主存中的i仍然=2。所以最终结果为2并不是3。这种现象就是**缓存一致性问题**。

## 2.2 解决方案

​		解决缓存一致性方案有两种：

* 通过在总线加LOCK#锁的方式
* 通过缓存一致性协议
  	
  

​		但是方案1存在一个问题，它是采用一种独占的方式来实现的，即总线加LOCK#锁的话，只能有一个CPU能够运行，其他CPU都得阻塞，效率较为低下。synchronized内置锁，从原理上讲就是这种独占锁的方式，不过其是通过Java内部机制实现的。

​		第二种方案，缓存一致性协议（MESI协议）它确保每个缓存中使用的共享变量的副本是一致的。**其核心思想如下：当某个CPU在写数据时，如果发现操作的变量是共享变量，则会通知其他CPU告知该变量的缓存行是无效的，因此其他CPU在读取该变量时，发现其无效会重新从主存中加载数据。**volatile实现的原理，就是这种机制，下面会详细介绍。

![thread-004](..\images\thread-004.png)		

##  2.3 Java内存模型

​		2.1从操作系统层次阐述了如何保证数据一致性，下面我们来看一下Java内存模型，稍微研究一下Java内存模型为我们提供了哪些保证以及在Java中提供了哪些方法和机制来让我们在进行多线程编程时能够保证程序执行的正确性。

​		在并发编程中我们一般都会遇到这三个基本概念：原子性、可见性、有序性。

###  2.3.1 原子性

​		原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

​		原子性就像数据库里面的事务一样，他们是一个团队，同生共死。其实理解原子性非常简单，我们看下面一个简单的例子即可：

```java
i = 0;        ---1
j = i ;       ---2
i++;          ---3
i = j + 1;    ---4
```

​		上面四个操作，有哪个几个是原子操作，那几个不是？如果不是很理解，可能会认为都是原子性操作，其实只有1才是原子操作，其余均不是。

* 1—在Java中，对基本数据类型的变量和赋值操作都是原子性操作； 
* 2—包含了两个操作：读取i，将i值赋值给j 
* 3—包含了三个操作：读取i值、i + 1 、将+1结果赋值给i； 
* 4—同三一样

​		在单线程环境下我们可以认为整个步骤都是原子性操作，但是在多线程环境下则不同，Java只保证了基本数据类型的变量和赋值操作才是原子性的（注：在32位的JDK环境下，对64位数据的读取不是原子性操作*，如long、double）。要想在多线程环境下保证原子性，则**可以通过锁、synchronized来确保**。**volatile是无法保证复合操作的原子性**。

## 2.3.2 可见性

​		可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

​		在上面已经分析了，在多线程环境下，一个线程对共享变量的操作对其他线程是不可见的。**Java提供了volatile来保证可见性**。当一个变量被volatile修饰后，表示着线程本地内存无效，当一个线程修改共享变量后他会立即被更新到主内存中，当其他线程读取共享变量时，它会直接从主内存中读取。 

​		除了volatile可以让变量保证可见性外，**synchronized，lock，并发集合，Thread.join和Thread.start**等都可以保证可见性（即happens-before的原则）。

### 2.3.3 有序性

​		有序性：即程序执行的顺序按照代码的先后顺序执行。

​		在Java内存模型中，为了效率是允许编译器和处理器对**指令进行重排序**，当然重排序它不会影响单线程的运行结果，但是对多线程会有影响。

​		Java提供volatile来保证一定的有序性。最著名的例子就是单例模式里面的DCL（双重检查锁  [具体可以参考](https://www.cnblogs.com/codingmengmeng/p/9846131.html)）。

```java
public class Singleton {
    private volatile static Singleton instance = null;
    public  static Singleton getInstance() {
        if(null == instance) {
            synchronized (Singleton.class) {
                if(null == instance) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

# 3 volatile小结

* volatile修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag；或者作为触发器，实现轻量级同步。
* volatile属性的读写操作都是无锁的，它不能替代synchronized，因为它没有提供原子性和互斥性。因为无锁，不需要花费时间在获取锁和释放锁上，所以它是第成本的。
* volatile只能作用域属性，我们用volatile修饰属性，这样compilers就不会对这个属性做指令重排序。
* volatile提供了可见性，任何一个线程堆其的修改将立刻对其他线程可见。volatile属性不会被线程缓存，始终从主内存中读取。
* volatile提供了happens-before保证，对volatile变量v的写入Happens-Before所有其他线程后续对v的读取操作
* volatile可以使得long和double的赋值是原子的。



# 4 单例模式

## 4.1 为什么需要单例

​		节省内存和计算，保证结果正确，方便管理

## 4.2 单例模式适用场景

*  无状态的工具类：比如日志工具类，不管是在哪里适用，我们需要的只是它帮我们记录日志信息，除此之外，并不需要在它的实例对象上存储任何状态，这时我们就只需要一个实例对象即可
* 全局信息类：比如我们在一个类上记录网站的访问次数，我们不希望有的访问被记录在对象A上，有的去记录在对象B上，这时候我们就让这个类成为单例。

## 4.3 单例模式的写法

### 4.3.1 饿汉式（静态常量）【可用】

因为是在类加载时初始化的，故而是线程安全的（线程安全的保证由JVM在类加载时控制）。

```java
/**
 * 饿汉式（静态常量）【可用】
 */
public class SingleTon1 {
    private  final static SingleTon1 INSTANCE = new SingleTon1();

    private SingleTon1() {
    }

    public static SingleTon1 getInstance() {
        return INSTANCE;
    }
}
```

### 4.3.2 饿汉式（静态代码块）【可用】

特性与4.3.1 一样

```java
/**
 * 饿汉式（静态代码块）【可用】
 */
public class SingleTon2 {
    private  final static SingleTon2 INSTANCE;

    static {
        INSTANCE = new SingleTon2();
    }

    private SingleTon2() {
    }

    public static SingleTon2 getInstance() {
        return INSTANCE;
    }
}
```

### 4.3.3 懒汉式（线程不安全）【不可用】

```java
/**
 * 懒汉式（线程不安全）【不可用】
 */
public class SingleTon3 {
    private  static SingleTon3 instance;

    private SingleTon3() {
    }

    public static SingleTon3 getInstance() {
        if (instance==null) {
            instance = new SingleTon3();
        }
        return instance;
    }
}

```

### 4.3.4 懒汉式（线程安全,同步方法）【不推荐用】

虽然实现了线程安装，但是却是性能较低，不推荐使用。

```java
/**
 * 懒汉式（线程安全）【不推荐】
 */
public class SingleTon4 {
    private  static SingleTon4 instance;

    private SingleTon4() {
    }

    public synchronized static SingleTon4 getInstance() {
        if (instance==null) {
            instance = new SingleTon4();
        }
        return instance;
    }
}
```

### 4.3.5 懒汉式（线程不安全,同步代码块）【不可用】

```java
/**
 * 懒汉式（线程不安全,同步代码块）【不可用】
 */
public class SingleTon5 {
    private  static SingleTon5 instance;

    private SingleTon5() {
    }

    public static SingleTon5 getInstance() {
        if (instance==null) {
            synchronized (SingleTon5.class) {
                instance = new SingleTon5();
            }
        }
        return instance;
    }
}
```

### 4.3.6  双重检查【推荐】

**优点**：线程安全，延迟加载，效率较高

为何要double-check?

​	线程安全

​	单check行不行

​	性能问题

为何要volatile？

1. 新建对象实际上有3个步骤 （ 创建空对象，调用构造器，引用赋值给变量）
2. 如果不设置volatile，则上述3个步骤可能发生重排序，这样可能会导致变量引用到空对象。

```java
/**
 * 双重检查（推荐）
 */
public class SingleTon6 {
    private volatile static SingleTon6 instance;

    private SingleTon6() {
    }

    public static SingleTon6 getInstance() {
        if (instance==null) {
            synchronized (SingleTon6.class) {
                if (instance==null) {
                    instance = new SingleTon6();
                }
            }
        }
        return instance;
    }
}
```

### 4.3.7 静态内部类【推荐】

归属于懒汉式，在外部类初始化时，内部类还不会初始化。只有在调用时，才会触发由JVM初始化内部类。同时，SingleTon7的构造是由JVM控制的，又保证了线程安全。

```java
/**
 * 静态内部类【推荐】
 */
public class SingleTon7 {

    private SingleTon7() {
    }

    private static class SingleTonInstence {
        private static final SingleTon7 INSTANCE= new SingleTon7();
    }

    public static SingleTon7 getInstance() {
       return SingleTonInstence.INSTANCE;
    }
}
```

### 4.3.8 枚举【推荐】

生产环境中推荐使用

```java
/**
 * 枚举【推荐，生成环境中推荐】
 */
public enum SingleTon8 {
    INSTANCE;

    public void dosomething() {
       // 与单例无关，只是在调用单例后可以执行的方法
        System.out.println("dosomething");
    }

    public static void main(String[] args) {
        SingleTon8.INSTANCE.dosomething();
    }
}
```