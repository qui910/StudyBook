# 1 线程安全

​		**定义**：当多个线程访问某个类时，不用考虑运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称为这个类是线程安全的。

​		线程安全的问题主要针对的就是共享变量，即线程安全问题可以理解为当多个线程同时读写一个共享变量并且没有任何同步措施时，导致出现脏数据或者其他不可见的结果。所以，无状态对象（即无共享变量）一定是线程安全的。



# 2 线程不安全示例

## 2.1 运行结果错误

​		类似a++等这种操作，看似原子，但实际并非如此，如下例所示：

```java
    public static int index=0;

    class MyRunnable implements Runnable{
        @Override
        public void run() {
              // 这里不能使用while来做演示，因为while是不控制循环次数的
              // 只要没达到10000就可以进入循环。这样是没法判定线程安全问题的
//            while(index<10000){
//                ThreadSafeTest.index++;
//            }

            // 改用for来演示线程安全问题
            for (int i=0;i<10000;i++) {
                index++;
            }
        }
    }

    /**
     * 测试运行结果出错
     * 演示计数不准确（减少），找出具体出错的位置
     * 结果：每次结果不一样，或许可能达到20000，或许不可能
     */
    private void testAddNum() throws InterruptedException {
        Thread thread1 = new Thread(new ThreadSafeTest().new MyRunnable());
        Thread thread2 = new Thread(new ThreadSafeTest().new MyRunnable());
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("最后结果："+ThreadSafeTest.index);
    }


    public static void main(String[] args) throws InterruptedException {
        ThreadSafeTest test = new ThreadSafeTest();
        test.testAddNum();
    }
```

多次运行结果：

```shell
第1次  最后结果：14709
第2次  最后结果：20000
```

原因分析：

![thread-009](..\images\thread-009.png)

## 2.2 活跃性问题：死锁，活锁，饥饿

​		死锁发生的原因：就是两个线程同时争取对方已经获取的锁，但是又不释放自己活得的锁。哲学家就餐就是死锁发生的最好示例。

![thread-010](..\images\thread-010.png)



## 2.3 对象发布和初始化时的安全问题

### 2.3.1 发布

> 即发布对象，把**对象的引用**传递给当前作用域之外的代码，使对象被它们使用。

​		就是创建对象并赋给一个变量，这就算是发布了，如果这个变量是静态公有的或者能被多线程共享，那就是相当于将该对象*发布* 给这些线程使用了。

```java
public class PublishDemo {
	public static Set<EntityInfo> infos;
	
	public PublishDemo(){
		infos=new HashSet<EntityInfo>();
	}
}
```

​		PublishDemo构造函数同过静态变量infos发布了set集合对象。但这样产生一个问题，infos集合里面存储的是EntityInfo对象的引用，那么我们infos发布出去，相当于间接把EntityInfo对象也发布出去了。间接发布的手段也非常多，只要能把引用传递出去，都算是发布了。

### 2.3.2 逸出

> 发布了本不该发布的对象，或错误的发布

​		逸出也是发布，区别在于：**发布**在于你想发布，**逸出**在于你不想发布的对象却也被发布了。或者在时间维度上说，在某一时刻你还没打算发布的对象也被发布出去，这也算逸出。以下是几种逸出的情况：

**（1）方法返回一个private对象（private的本意是不让外部访问）**

```java
// 此例中，封装一个集合的目的就是要安全的访问这个集合。但是`get()`方法却意外的将`infos`的引用暴露给了外部环境，这样对于集合的安全访问就不受控制了。
public class EscapeDemo {
	private List<String> infos = new ArrayList<String>();

	public void add(String val) {
		infos.add(val);
	}

	public synchronized String removeFir() {
		if (infos.size() > 0) {
			return infos.remove(0);
		}
		return null;
	}

	public List<String> get() {
		return infos;
	}
}
```

**正确方法**：用“副本”代替“真身”

**（2）还未完成初始化（构造函数没有完全执行完毕）就把对象提供给外界**

**（2.1）在构造函数中未初始化完毕就this赋值**

此例中Point类还未初始化完成，就将引用赋值给了静态变量，导致对象不正常的发布
打印结果：Point{x=1, y=0}

```java
public class ThreadSafeTest1 {
    static Point point;
    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            try {
                new Point(1,2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        Thread.sleep(50);
        if (point!=null) {
            System.out.println(point.toString());
        }
    }

    static class Point{
        private final int x,y;
        public Point(int x, int y) throws InterruptedException {
            this.x = x;
            ThreadSafeTest1.point=this;
            Thread.sleep(100);
            this.y = y;
        }
        @Override
        public String toString() {
            return "Point{" +
                    "x=" + x +
                    ", y=" + y +
                    '}';
        }
    }
}
```

**（2.2）隐式逸出-注册监听事件**

```java
// 此例是在构造函数中，创建了一个监听线程，但是无形之中`EventListener`将外部类`EscapeDemo`的引用通过this给暴露出去了。这样可能导致外部系统访问到一个未构建好的`EscapeDemo`对象，导致不正确的构造溢出。
public class EscapeDemo {
	public EscapeDemo(EventSource source){
		source.registerListener(
				new EventListener(
						public void onEvent(Event e){
							doSomeThing(e);
						}
				));
	}
}
```

**正确的做法**：不要在构造函数中立即启动监听线程，而是先保存这个对象的引用，等构造完成后再通过其它方式启动它。如下，通过工厂模式，私有化构造函数，通过newInstance方法发布EscapeDemo 对象。

```java
public class EscapeDemo {
	private final EventListener listener = null;
	private EscapeDemo(){
		listener=	new EventListener(
						public void onEvent(Event e){
							doSomeThing(e);
						});
	}

	public static newInstance(EventSource source) {
		EscapeDemo demo = new EscapeDemo();
		source.registerListener(demo.listener);
		return demo;
	}
}
```

另外并不是只有线程放置在构造函数中会导致逸出，如果在构造函数中调用一个可改写的实例方法（不是终结方法，也不是私有方法），同样也会导致this引用在构造过程中逸出。

**（2.3） 构造函数中运行线程**

```java
/**
 * 对象发布逸出测试
 * 构造函数中运行线程
 * 结果：
 * Exception in thread "main" java.lang.NullPointerException
 * 分析：就是在构造器中子线程是否完成初始化工作，无法控制
 * 所以不能再构造器中新开线程。
 * 生产环境中：就是在构造器中无意中进行获取数据库连接池等操作
 * 如果过早获取对象，连接池可能未初始好，会导致空点异常。
 */
public class ThreadSafeTest3 {

    public Map<String,String> states;

    public ThreadSafeTest3() {
        new Thread(()->{
            states = new HashMap<>();
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            states.put("a","1");
        }).start();
    }

    public static void main(String[] args) {
        ThreadSafeTest3 test = new ThreadSafeTest3();
        test.states.get("a");
    }
}

```

**针对第2个问题还未完成初始化的3个小问题，正确的处理方法都可以使用工厂模式来解决**

# 3 线程安全问题分析

​		经过第2节的几个线程不安全的示例，我们由此可以得出几个造成线程不安全的情况：

* 访问共享的变量或资源，会有并发风险，比如对象的属性，静态变量，共享缓存，数据库等。
* 所有依赖时序的操作，即使每一步操作都是线程安全的，还是存在并发问题：read-modify-write,check-then-act
* 不同的数据之间存在捆绑关系的时候，单独修改某个数据，就会造成脏数据。必须同时修改
* 使用其他类的时候，如果对方没有声明自己是线程安全的。

## 3.1 内存可见性

​		Java内存模型规定，将所有的变量都存放在主内存中，当线程使用变量时，会把主内存里面的变量复制到自己的工作空间或者叫工作内存，线程读写变量时操作的是自己工作内存中的变量，处理完成后将变量值更新到主内存。

​		多个线程同时处理共享变量时，由于都是各自在各自的工作空间中处理数据。这样就可能导致线程A对于变量的更新值，对于线程B是不可见。从而引发共享变量值的错误。

​		解决内存不可见的方法：有锁，volatile等，下面会挨个讲解到。

​		内存可见性的错误示例：

* 一个线程修改了状态，而另一个线程要读取这个状态返回的仍是之前的状态。
* 一个对象正在被一个线程实例化，而另一个线程读取这个的对象，但是读取到的却是未完全初始化好的对象；
* 第三种是指令重排导致的，jvm会帮我们把指令重新排序后以提高运行效率，但是重排之后可能会得到错误的结果。比如下面这段代码，如果main里面的语句重排后按312运行，那么输出的值可能是0，因为执行输出语句时，number的赋值语句可能还没执行或正在执行，这就要看哪边的线程跑的快了。

```java
public class CodeRearrangeDemo extends Thread {
	static boolean ready = false;
	static int number = 0;

	public void run() {
		while (!ready)
			Thread.yield();
		System.out.println(number);
	}

	public static void main(String[] args) {
		new CodeRearrangeDemo().start();// 1
		number = 42;// 2
		ready = true;// 3
	}
}
```

​		这些问题防不胜防，你不可能推断出所有出错的原因，但是你却可以简单的避免他们，只要数据在多线程之间共享，那就要使用正确的同步。

## 3.2 失效数据

​		当一个线程读取到一个状态的时候，这个状态可能是失效了的状态。即就是该状态的修改与读取没有同步的问题。

* 线程可能会读取两次状态但是，一次获得的是正确的，另一次获得是错误的。比如如下代码，num是共享数据，num正在被其它某个线程由3修改为5了，而同时另一个线程在执行如下语句时可能第一次读到的num是3（失效的），而第二次读num时就是5了，最终num值变成了8，一个完全错误的值。

```java
num=num+num;
```

* 比如对象的引用失效了，而线程却正好获取到了这个失效了的引用，就有可能导致报出异常或者更严重的安全问题。

## 3.3 非原子的64位操作

​		线程在没有同步的情况下读取了共享数据，可能会得到失效值，但这个值至少是某个线程设置的值，而不是一个随机我们就称这是最低的安全性（out-of-thin-airsafety）。

​		对于绝大多数变量，java内存模型是符合最低的安全性的，但是有个例外。对于非volatile类型的64位数值变量（double和long），jvm允许在读取操作或写入操作时分解为两个32位的操作指令，也就是先操作一个32位的内存，再操作另外一个32位的内存。这就有很可能在读取时，读取某个值的高32位和另一个值的低32位组成一个完全错误的数据。



# 4 线程安全的解决方法

​		根据上面讲解的几个线程不安全的分析原理，我们这里总结保证线程安全的几种方法。总结来说，就是保证可见性，有序性和原子性。

## 4.1 线程封闭

> 为了避免同步问题，最简单的方法就是不共享数据，让变量只在一个线程内可见使用，或者同一时段只有一个线程对该变量可见处理。这种技术就称为线程封闭。

​		最常见的线程封闭的例子就是局部变量。线程封闭技术是实现线程安全最简单的方式之一。实现线程封闭的几种方法：

* Ad-hoc线程封闭
  就是完全靠实现者代码控制的线程封闭。Ad-hoc线程封闭非常脆弱，没有任何一种语言特性能将对象封闭到目标线程上。

* 栈封闭
  就是前面所说的局部变量，局部变量的固有属性之一就是封闭在线程之内。局部变量的引用值是存储在栈内的局部变量表之中，其它线程无法访问到。

* ThreadLocal封闭

## 4.2 volatile

​		volatile关键字具有**有序性和可见性，不具备原子性**。它也没有锁机制，所以是相对于synchronized关键字更轻量级的同步机制。相对的不具备原子性说明它不是一个完整的同步方式，volatile的同步具有一定局限性。

## 4.3 锁

​		无论是synchronized和lock锁，都可以保证操作的有序性和可见性，同时也能保证操作的原子性。

## 4.4 Atomic原子类

​		使用JUC提供的原子类可以保证某个操作的原子性，但是如果连续调用多个原子操作，这样就会造成多个原子操作的集合不为原子的。故而在使用原子类时也要格外注意。


## 4.5 不变性

​		满足同步需求的另一种方法是不可变对象（Immutable Object）。由于对象的状态不可变了，那么共享的时候也不会出现之前所说的问题。不可变对象很简单，它只有一种状态，并且是在构造函数时确定的。

> 不可变对象一定是线程安全的

​		在java语言规范与java内存模型中没有给出不可变性的正式定义，书中给出了如下三个条件，满足这三个条件的对象才是不可变对象：

- 对象创建以后其状态就不能修改
- 对象的所有域都是final类型
- 对象是正确创建的（在创建对象期间，this引用没有逸出）

以上部分内容参考：

[JAVA并发编程之对象的共享](https://blog.csdn.net/u014296316/article/details/86320855)

## 4.6 代码示例

```java
/**
 * 内存共享测试
 */
public class ThreadSharedMemoryTest {

  public static void main(String[] args) {

//      testNormal();
//      testVolatile();
      testSynchronized();
  }

    /**
     * 针对多线程情况下，使用共享变量，不做特殊限制。
     * 最后的结果：9981
     * 说明：这对共享变量产生竞争
     * @throws InterruptedException
     */
  private static void testNormal() {
      TestBean bean = new ThreadSharedMemoryTest().new TestBean();
      for (int i=0;i<9999;i++) {
          NormalThread temp = new ThreadSharedMemoryTest().new NormalThread("thread"+i,bean);
          temp.start();
      }
  }

    /**
     * 多线程情况下，将共享变量设置为volatile
     * 最后的结果：9994
     * 说明：虽然使用了volatitle关键字，但是因为run中的操作不是原子的，所以还是会出现线程不安全
     * @throws InterruptedException
     */
    private static void testVolatile() {
        TestBean bean = new ThreadSharedMemoryTest().new TestBean();
        for (int i=0;i<9999;i++) {
            VolatileThread temp = new ThreadSharedMemoryTest().new VolatileThread("thread"+i,bean);
            temp.start();
        }
    }

    /**
     * 多线程情况下，将get和set操作通过使用锁，形成原子操作
     * 最后的结果：10000
     * 线程安全
     */
    private static void testSynchronized() {
        TestBean bean = new ThreadSharedMemoryTest().new TestBean();
        for (int i=0;i<9999;i++) {
            SynchronizedThread temp = new ThreadSharedMemoryTest().new SynchronizedThread("thread"+i,bean);
            temp.start();
        }
    }


  class TestBean {
      private int value=1;

      private volatile  int value1=1;

      public int getValue() {
          return value;
      }

      public void setValue(int value) {
          this.value = value;
      }

      public int getValue1() {
          return value1;
      }

      public void setValue1(int value1) {
          this.value1 = value1;
      }
  }

  class NormalThread extends Thread {
      TestBean bean;

      public NormalThread(String name, TestBean bean) {
          super(name);
          this.bean = bean;
      }

      @Override
      public void run() {
          bean.setValue(bean.getValue()+1);
          System.out.println(getName()+"当前结果："+bean.getValue());
      }
  }

    class VolatileThread extends Thread {
        TestBean bean;

        public VolatileThread(String name, TestBean bean) {
            super(name);
            this.bean = bean;
        }

        @Override
        public void run() {
            bean.setValue1(bean.getValue1()+1);
            System.out.println(getName()+"当前结果："+bean.getValue1());
        }
    }

    class SynchronizedThread extends Thread {
        TestBean bean;

        public SynchronizedThread(String name, TestBean bean) {
            super(name);
            this.bean = bean;
        }

        @Override
        public void run() {
            synchronized (bean) {
                bean.setValue1(bean.getValue1() + 1);
                System.out.println(getName() + "当前结果：" + bean.getValue1());
            }
        }
    }
}

```

# 5 多线程导致的性能问题

​		多线程情况下，下述两种情况会导致性能问题的发生：

##  5.1 上下文切换

​		上下文切换可以认为是内核（操作系统的核心）在CPU上对于进程（包括线程）进行以下的活动的个：（1）挂起一个进程，将这个进程在CPU中的状态（上下文）存储于内存中的某处。（2）在内存中检索下一个进程的上下文并将其在COU的寄存器中恢复。（3）跳转到程序计数器所指向的位置（即跳转到进程被中断的代码行 如sleep），以恢复该进程、

​		缓存开销：CPU重新缓存

​		什么会导致密集的上下文切换：频繁的竞争锁或者由于IO读写等原因导致频繁阻塞。

## 5.2 内存同步

​		JVM内存模型，为了数据的正确性，同步手段往往会使用禁止编译器优化，使CPU内的缓存失效。

# 6 Java内存模型（JMM）

​		在了解了线程安全的一些示例和分析后，我们还是要持续的研究下Java的底层原理，即JMM。只有这样才能更好的了解并发，控制并发。

## 6.1  为何要研究底层原理

​		为何要了解底层原理，从以下两个方面来讲解。

### 6.1.1 从Java代码到CPU指令

​		首先，下图是Java代码到CPU指令的示意图。

![thread-011](..\images\thread-011.png)

​		从图上我们可以得出以下4步，是我们Java代码能执行关键。

1.最开始，我们编写的Java代码，是*.java文件

2.在编译（javac命令）后，从刚才*.java文件会变成一个新的Java字节码文件( ` *.class`）

3.JVM会执行刚才生成的字节码文件，并把字节码文件转化机器指令

4.机器指令可以直接在CPU上运行，也就是最终的程序执行

​		虽然，JVM给我们带来了很大的方便（一次编译，到处运行），但是不同系统的JVM实现却会带来不同的“翻译”，不同的CPU平台的机器指令又千差万别，无法保证并发安全的效果一致。这就导致我们的并发程序最终在不同机器的运行结果会千差万别。

### 6.1.2 重点向下钻研

​		为了解决翻译为机器指令的不规范性，我们研究底层原理就是为了能更好的将转换过程规范化，原则统一化。

 ## 6.2 3个模型的区别

​		在介绍底层原理前，我们首先区分3个模型的区别：JVM内存结构 ，Java内存模型，Java对象模型。

### 6.2.1  JVM内存结构

和Java虚拟机的运行时区域有关。

![thread-012](..\images\thread-012.png)



### 6.2.2 Java内存模型

和Java的并发编程有关。将在下个小节，重点讲述。

### 6.2.3 Java对象模型

和Java对象在虚拟机中的表现形式有关，主要描述Java对象自身的存储模型![thread-013](..\images\thread-013.png)

* JVM会个这个类创建一个instanceKclass。保存方法区，用来在JVM层表示该Java类型

* 当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对昂中包含了对象头以及实例数据
* 如果有方法中使用者两个对象变量，则会在栈中保存着两个对象的引用地址

## 6.3 Java内存模型

​		Java内存模型，即Java Memory Model，简称JMM。在诸如C，C++中并不存在内存模型的概念，这样就导致C程序的运行结果就依赖处理器，不同处理器结果不一样，就无法保证并发安全。在这种情况下，急需一个标准，让多线程运行的结果可预期。

​		从上述理解，JMM实际上是一组规范，需要各个JVM的实现来遵守JMM规范，以便于开发者可以利用这些规范，更方便的开发多线程程序。

​		在我们日常的并发编程中，可能对于JMM并过多的干预。但实际行，如果没有这样的一个JMM内存模型来规范，那么狠可能经过了不同JVM的不同规则的重排序之后，导致不同的虚拟机运行的结果不一样，会造成很大的问题。

​		JMM除了是规范外，还是工具类和关键字的原理。例如volatile，synchronized，Lock等的原理都是JMM。如果没有JMM，那就需要我们自己指定什么时候用内存栅栏等，那就是相当麻烦，正是因为有了JMM，让我们只需要用同步工具和关键字就可以开发并发程序。

​		JMM的最重要的3点就是重排序，可见性，原子性。

### 6.3.1 重排序

#### 6.3.1.1 重排序的案例

```java
/**
 * 演示重排序（OutOfOrderExecution）的现象
 * 因为重排序不是每次都发生，需要直到某个条件才停止，测试小概率事件
 */
public class ThreadJMMTest {
    private static int x=0,y=0;
    private static int a=0,b=0;

    public static void main(String[] args) throws InterruptedException {

        int i = 0;

        // 这里使用循环就是测试小概率事件
        for(;;) {
            i++;
            x=0;
            y=0;
            a=0;
            b=0;

            // 如果要演示第3种情况，需要2个线程同时开始，这里就使用CountDownLatch
            CountDownLatch latch = new CountDownLatch(1);

            Thread one = new Thread(()->{
                try {
                    latch.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                a=1;
                x=b;
            });
            Thread two = new Thread(()->{
                try {
                    latch.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                b=1;
                y=a;
            });
            one.start();
            two.start();
            latch.countDown();
            one.join();
            two.join();

            String result = "第"+i+"次(x="+x+",y="+y+")";
            if (x==0 && y==0) {
                System.out.println(result);
                break;
            } else {
                System.out.println(result);
            }
        }
    }
}
```

这里的运行结果一共有4种情况：

```shell
1.  a=1;x=b(0);b=1;y=a(1),最终结果 x=0，y=1
2.  b=1;y=a(0);a=1;x=b(1),最终结果 x=1，y=0
3.  b=1;a=1;x=b(1);y=a(1),最终结果 x=1，y=1
4.  x=b(0);y=a(0);a=1;b=1,最终结果 x=0，y=0  （重排序）
```

第4种情况就是属于重排序，指的就是a=1;和x=b;两个指令的顺序重新排序，变为x=b;先执行。线程2同理。

重排序：在线程1内部的两行代码的实际执行顺序和代码在Java文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，它们的顺序被改变了，这就是重排序，这里被颠倒的是y=a和b=1这两行语句。

#### 6.3.1.2 重排序的好处

提高处理速度。

举例说：如下面的语句 a=3;b=2;a=a+1; 正常的指令为：

```c
// a=3
load a;
set to 3;
store a;
// b=2
load b;
set to 2;
store b;
// a=a+1
load a;
set to 4;
store a;
```

重排序后：

```c
// a=3;a=a+1
load a;
set to 3;
set to 4;
store a;
// b=2
load b;
set to 2;
store b;
```

这样优化的好处，就是减少了部分指令，使该部分计算速度加快。

#### 6.3.1.3 重排序的3种情况

**编译器优化：**包括JVM，JIT编译器等。

**CPU指令重排序：**就算编译器不发送重排，CPU也可能对指令进行重排

**内存的重排序：**这是因为各自线程的的缓存影响，线程A的修改线程B却看不到，引出可见性问题。最后结果其实和重排序类似。

### 6.3.2 可见性

#### 6.3.2.1 可见性问题案例

```java
/**
 * 演示可见性带来的问题
 */
public class ThreadJMMTest1 {
    int a=1;
    int b=2;

    private void print() {
        System.out.println("b="+b+",a="+a);
    }

    private void change() {
        a=3;
        b=a;
    }

    public static void main(String[] args) {

        while(true) {
            ThreadJMMTest1 test1 = new ThreadJMMTest1();
			// 线程1
            new Thread(() -> {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                test1.change();
            }).start();
			// 线程2
            new Thread(() -> {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                test1.print();
            }).start();
        }
    }
}
```

这里的运行结果一共有3种情况：

```shell
1. a=3,b=2  # 属于线程1执行一部分后线程2打印
2. a=1,b=2  # 属于线程2先打印 或者 线程1运行完成但是线程2看不到改变结果
3. a=3,b=3  # 属于线程1执行完成后线程2打印
4. a=1,b=3  # 可见性问题，线程1执行完成后，只有b可见，而a不可见
```

解决方法：

* 主要将a和b都改为volatile，则问题4不会发生。

* 给b添加volatile，不仅b被影响，也可以实现轻量级同步。b之前的写入（对应代码b=a）对读取b后的代码（print b）都可见，所以在writeThread里对a的赋值，一定会对readThread里的读取可见，所有这里的a即使不加volatile，只要b读到3，就可以由happen-before原则保证了读取到的都是3而不可能读取到1.

#### 6.3.2.2 可见性产生的原因

![thread-014](..\images\thread-014.png)

CPU有多级缓存，导致读的数据过期

* 高速缓存的容量比主内存小，但是速度仅次于寄存器，所以在CPU和主内存之间就多了Cache层
* 线程间的对于共享变量的可见性问题不是直接由多核引起的，而是由多缓存引起的。
* 如果所有核心都只用一个缓存，那么也就不存在内存可见性问题了
* 每个核心都会将自己需要的数据读到独占缓存中，数据修改也是写入到缓存中，然后等待刷入到主存中。所以会导致有些核心读取的值时一个过期的值。

#### 6.3.2.3 JMM抽象：主内存和本地内存

​		Java作为高级语言，屏蔽了这些底层细节，用JMM定义了一套读写内存数据的规范，虽然我们不需要关系一级缓存和二级缓存的问题，但是，JMM抽象了主内存和本地内存的概念。

​		这里的本地内存并不是真的是一块给每个线程分配的内存，而是JMM的一个抽象，是对于寄存器，一级缓存，二级缓存等的抽象。这种抽象是根据处理器的不同而分别抽象的。

![thread-015](..\images\thread-015.png)		

图二也是

![thread-016](..\images\thread-016.png)

主内存和本地内存的关系，JMM有以下规定：

* 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量是主内存中的拷贝
* 线程不能直接读写主内存中的变量，而是只能操作自己工作内存中的变量，然后再同步到主内存中
* 主内存是多个线程共享的，但线程间不能共享工作内存，如果线程间需要通信，必须借助主内存中转来完成。

总结：所有的共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是通过本地内存交换的，所以才导致了可见性问题。

#### 6.3.2.4 Happens-Before原则

**什么是happens-before？**

* happens-before规则是用来解决可见性问题的：在时间上，动作A发生在动作B之间，B保证能看见A，这既是happens-before。

* 两个操作可以用happens-before来确定它们的执行顺序：如果一个操作happens-before于另一个操作，那么我们说第一个操作对于第二个操作是可见的。

**什么不是happens-before？**

* 两个线程没有相互配合的机制，所以代码X和Y的执行结果并不能保证总被对方看到，这就不具备什么是happens-before。

如 6.3.2.1 的案例，就不是happens-before。`b=a`保证happens-before `print(b)`，但是`a=3` 不能保证happens-before `print(a)`。

**Happens-Before规则：**

* 单线程规则

![thread-017](..\images\thread-017.png)

happens-before并不影响重排序

* **锁操作（synchronized和Lock）**

![thread-018](..\images\thread-018.png)

![thread-019](..\images\thread-019.png)

* **volatile变量**

![thread-020](..\images\thread-020.png)

* 线程启动

![thread-021](..\images\thread-021.png)

* 线程join

![thread-022](..\images\thread-022.png)

* 传递性

如果hb（A，B）而且hb（B，C），那么可以退出hb（A，C）

* 中断

一个线程被其他线程interrupt时，那么检查中断（isInterrupted）或者抛出InterruptedException一定能看到。

* 构造方法

对象构造方法的最后一行指令happens-before与finalize()方法的第一行指令

* 工具类的happens-before原则
   * 线程安全的容器的get一定能看到在此之间的put等存入操作
   * CountDownLatch
   * Semaphore
   * Future：get一定能获取到callable执行的结果
   * 线程池
   * CyclicBarrier



#### 6.3.2.5 volatile关键字

详细请查看第004章

#### 6.3.2.6 保证可见性的措施

详细请查看第004章

#### 6.3.2.7 synchronized可见性的理解

详细请查看第003章

### 6.3.3 原子性

#### 6.3.3.1 什么是原子性

一系列的操作，要么全部执行成功，要么全部不执行，不会出现执行一半的情况，是不可分割的。

i++ 不是原子性。

用synchronized可以实现原子性。

#### 6.3.3.2 Java中的原子操作

* 除long和double之外的基本类型（int，byte，boolean，short，char，float）的赋值操作
* 所有引用reference的赋值操作，不管是32位的机器还是64位的机器
* java.concurrent.Atomic.* 包中所有类的原子操作

#### 6.3.3.3 long和double的原子性



#### 6.3.3.4 原子操作+原子操作！=原子操作

 

