# 1 线程基础

​		线程是并发的基础单元，这里从线程开始说明。实现多线程有以下两种方式：

```
class TestThreadStateA extends Thread {
    @Override
    public void run() {
}
```



## 1.1 线程属性

​		线程的属性包括线程的编号（ID），线程的名称（Name）,线程的类别（Daemon）和优先级（Priority）。

| 属性             | 属性类型及用途                                               | 只读属性 | 重要注意事项                                                 |
| ---------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| 编号( lD )       | 类型：long。用于标识不同的线程。不同的线程拥有不同的编号     | 是       | 某个编号的线程运行结束后，该编号可能被后续创建的线程使用。 不同线程拥有的编号虽然不同，但是这种编号的唯一性只在Java 虚拟机的一次运行有效 。 也就是说重启一个 Java 虚拟机（如重启 Web 服务器）后，某些线程的编号可能与上次 Java 虚拟机运行的某个线程的编号一样 ， 因此该属性的值不适合用作某种唯一标识，特别是作为数据库中的唯一标识（如主键） |
| 名称(Name)       | 类型:String。面向人（而非机器）的一个属性，用于区分不同的线程。默认值与线程的编号有关。默认值的格式为：“Thread-线程编号”，如“Thread-0” | 否       | Java 并不禁止我们将不同的线程的名称属性设置为相同的值。 尽管如此，设置线程的名称属性有助于代码调试和问题定位 |
| 线程类别(Daemon) | 类型:boolean。值为true表示相应的线程为守护线程，否则表示相应的线程为用户线程。该属性的默认值与相应线程的父线程的该属性的值相同。 | 否       | 该属性必须在相应线程启动之前设置，即对setDaemon方法的调用必须在对start方法的调用之前，否则 setDaemon 方法会抛出IllegalThreadStateException异常。负责一些关键任务处理的线程不适宜设置为守护线程 |
| 优先级(Priority) | 类型：int。该属性本质上是给线程调度器的提示，用于表示应用程序希望哪个线程能够优先得以运行。 Java 定义了 l～ 10 的 10个优先级 。默认值一般为 5（表示普通优先级）。对于具体的一个线程而言，其优先级的默认值与其父线程（创建该线程的线程）的优先级值相等 | 否       | 一般使用默认优先级即可。不恰当地设置该属性值可能导致严重的问题（线程饥饿） |

​		父线程和子线程之间的生命周期也没有必然的联系 。 比如父线程运行结束后 ，子线程可以继续运行，子线程运行结束也不妨碍其父线程继续运行 。

## 1.2 线程状态

​		Java线程的状态可以使用监控工具查看，也可以通过 Thread.getState（）调用来获取 。Thread.getState（）的返回值类型 Thread.State 是一个枚举类型（ Enum ） 。 

​		Thread.State所定义的线程状态包括以下几种 :

| 状态                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 新建状态(New)           | 新创建了一个线程对象，但还没有调用start()方法，就进入了新建状态。例如，Thread thread = new Thread()。 |
| 就绪状态(Runnable)      | Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。 |
| 阻塞状态(Blocked)       | 表示线程阻塞于锁。线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。同时放弃CPU使用权，暂时停止运行。 |
| 等待(WAITING)           | 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。同时放弃CPU使用权，暂时停止运行。例如：通过调用线程的wait()方法，让线程等待某工作的完成。 |
| 超时等待(TIMED_WAITING) | 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。该状态不同于WAITING，它可以在指定的时间后自行返回。 |
| 终止(TERMINATED)        | 线程执行完了或者因异常退出了run()方法，该线程结束生命周期。  |

下面这张图很好的将各个状态串接起来：

![thread-001](..\images\thread-001.png)

### 1.2.1 **初始状态**

​		实现Runnable接口和继承Thread可以得到一个线程类，new一个实例出来，线程就进入了初始状态。

### 1.2.2 **就绪状态**

* 就绪状态只是说你资格运行，调度程序没有挑选到你，你就永远是就绪状态。
* 调用线程的start()方法，此线程进入就绪状态。
* 当前线程sleep()方法结束，其他线程join()结束，等待用户输入完毕，某个线程拿到对象锁，这些线程也将进入就绪状态。
* 当前线程时间片用完了，调用当前线程的yield()方法，当前线程进入就绪状态。
* 锁池里的线程拿到对象锁后，进入就绪状态。

### 1.2.3 **运行中状态**

​		线程调度程序从可运行池中选择一个线程作为当前线程时线程所处的状态。这也是线程进入运行状态的唯一一种方式。

### 1.2.4 **阻塞状态**

​		阻塞状态是线程阻塞在进入synchronized关键字修饰的方法或代码块(获取锁)时的状态。

### 1.2.5 等待

​		处于这种状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态。

### 1.2.6 超时等待

​		处于这种状态的线程不会被分配CPU执行时间，不过无须无限期等待被其他线程显示地唤醒，在达到一定时间后它们会自动唤醒。

### 1.2.7 **终止状态**

* 当线程的run()方法完成时，或者主线程的main()方法完成时，我们就认为它终止了。这个线程对象也许是活的，但是，它已经不是一个单独执行的线程。线程一旦终止了，就不能复生。
* 在一个终止的线程上调用start()方法，会抛出java.lang.IllegalThreadStateException异常。

### 1.2.8 方法调用与线程状态

​		每个锁对象都有两个队列，一个是同步队列（即就绪队列），一个是等待队列（即阻塞队列）。就绪队列存储了已就绪（将要竞争锁）的线程，阻塞队列存储了被阻塞的线程。当一个阻塞线程被唤醒后，才会进入就绪队列，进而等待CPU的调度；反之，当一个线程被wait后，就会进入阻塞队列，等待被唤醒。

#### 1.2.8.1 等待队列

- 调用obj的wait(), notify()方法前，必须获得obj锁，也就是必须写在synchronized(obj) 代码段内。
- 与等待队列相关的步骤和图

![thread-002](..\images\thread-002.png)

#### 1.2.8.2 同步队列

* 当前线程想调用对象A的同步方法时，发现对象A的锁被别的线程占有，此时当前线程进入同步队列。简言之，同步队列里面放的都是想争夺对象锁的线程。
* 当一个线程1被另外一个线程2唤醒时，1线程进入同步队列，去争夺对象锁。
* 同步队列是在同步的环境下才有的概念，一个对象对应一个同步队列。、
* 线程等待时间到了或被notify/notifyAll唤醒后，会进入同步队列竞争锁，如果获得锁，进入RUNNABLE状态，否则进入BLOCKED状态等待获取锁。

代码示例：

```java
/**
 * 线程状态测试
 */
public class ThreadStateTest {

  public static void main(String[] args) throws InterruptedException {
      // testState1();
      testState2();
  }


    /**
     * 测试从 NEW-》RUNNABLE-》TIMED_WAITING-》RUNNABLE-》TERMINATED
     */
  private static void testState1() throws InterruptedException {
      Thread thread1 = new TestThreadState();
      thread1.start();
      Thread.sleep(1000);
      // 该状态不同于WAITING，它可以在指定的时间后自行返回。
      System.out.println("线程超时等待(TIMED_WAITING)状态：" + thread1.getState().name());

      Thread.sleep(10000);
      // 表示该线程已经执行完毕
      System.out.println("线程终止(TERMINATED)状态：" + thread1.getState().name());
  }

    /**
     * 测试从 NEW-》RUNNABLE-》WAITING-》RUNNABLE-》TERMINATED
     */
  private static void testState2() throws InterruptedException {
      Thread thread1 = new TestThreadStateA();
      thread1.start();
      Thread.sleep(1000);
      // 该状态不同于WAITING，它可以在指定的时间后自行返回。
      System.out.println("线程等待(WAITING)状态：" + thread1.getState().name());
      Thread.sleep(2000);
      thread1.interrupt();
      Thread.sleep(10000);
      // 表示该线程已经执行完毕
      System.out.println("线程终止(TERMINATED)状态：" + thread1.getState().name());
  }
}

class TestThreadState extends Thread {

    public TestThreadState() {
        System.out.println("线程编号为：："+this.getId());
        // 新创建了一个线程对象，但还没有调用start()方法。
        System.out.println("线程初始（NEW）状态："+this.getState().name());
    }

    @Override
    public void run() {
        // Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
        // 线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，
        // 等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
        System.out.println("线程运行(RUNNABLE)状态：" + this.getState().name());
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 线程从超时等待返回后，状态变更为运行中
        System.out.println("线程运行(RUNNABLE)状态状态：" + this.getState().name());
    }
}

class TestThreadStateA extends Thread {

    public TestThreadStateA() {
        System.out.println("线程编号为：："+this.getId());
        // 新创建了一个线程对象，但还没有调用start()方法。
        System.out.println("线程初始（NEW）状态："+this.getState().name());
    }

    @Override
    public void run() {
        // Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
        // 线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，
        // 等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
        System.out.println("线程运行(RUNNABLE)状态：" + this.getState().name());

        try {
            synchronized (this) {
                this.wait();
            }
        } catch (InterruptedException e) {
            System.out.println("线程运行状态状态：" + this.getState().name());
        }

        // 线程从超时等待返回后，状态变更为运行中
        System.out.println("线程运行(RUNNABLE)状态状态：" + this.getState().name());
    }
}
```





##  1.3 线程方法

### 1.3.1 中断线程

```java
// 向线程发送中断请求。线程的中断状态被设置为true。
void interrupte();
// 测试当前线程是否被中断。它将当前线程的中断状态重置为fasle。
static boolean interrupted();
// 测试线程是否被中断。这不会改变线程的中断状态。
boolean isIterrupted();
```

####  1.3.1.1 就绪状态(Runnable)

​		当线程状态处于就绪状态(Runnable)时，由线程自身或者其他线程调用`interrupte()`方法时，将会把该线程的中断状态设置为true。此时，线程自身可以不时的检查这个标志，从而判断线程是否被中断，并处理这种问题。

#### 1.3.1.2 等待(WAITING)

​		当线程状态因`wait()`, `join()`进入等待状态时，由其他线程调用`interrupte()`方法后，线程中断状态设置为true，而线程本身处于等待状态无法检查中断标志，这时线程会清除中断状态（设置为false），并返回InterruptedException异常。

​		此时最好的做法是在catch代码块中捕获这个异常，并重置中断状态为true，并进行相应的业务逻辑处理后。或直接不做任何处理而直接抛出异常。

#### 1.3.1.3 超时等待(TIMED_WAITING)

​		当线程状态因`wait(long)`, `join(long)`或`sleep(long)`进入超时等待状态时，和1.3.1.2 类似也是产生InterruptedException异常。

#### 1.3.1.4 阻塞状态(Blocked)

* 当线程因等待锁`synchronized`同步锁进入阻塞状态后，由其他线程调用`interrupte()`方法后，线程中断状态设置为true。但是不能影响中断，线程会持续阻塞在等待锁的阻塞队列中。正在尝试获取同步锁的任务或正在尝试执行I/O的任务是不能被中断的。

* 当线程等待的锁是Lock时，如果使用的是方法`lockInterruptibly()`会响应中断并返回InterruptedException异常，同时线程状态变为就绪状态(Runnable)。同时需要注意`lock()`方法不会响应中断，区别在于park()等待被唤醒时lock会继续执行park()来等待锁，而 `lockInterruptibly`会抛出异常；

#### 1.3.1.5 其他中断

* 如果线程堵塞在java.nio.channels.InterruptibleChannel的IO上，Channel将会被关闭，线程被置为中断状态，并抛出java.nio.channels.ClosedByInterruptException；
* 如果线程堵塞在java.nio.channels.Selector上，线程被置为中断状态，select方法会马上返回，类似调用wakeup的效果；

代码示例：

```java
/**
 * 线程中断状态测试
 */
public class ThreadInterruptedTest {

    public static void main(String[] args) {
        //runnableTest();

        //waitingTest();

        //blockedTest();

        blockedTest1();
    }


    /**
     * 就绪状态测试
     */
    private static void runnableTest() {
        Thread targetTh = new Thread(()->{
            for (int i=0; i<10000 ; i++) {
                System.out.println("i:"+i);
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程中断");
                    break;
                }
            }
        });
        targetTh.start();
        try {
            TimeUnit.MICROSECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程状态:"+targetTh.getState().name());
        targetTh.interrupt();
    }

    /**
     * 等待状态或超时等待测试
     */
    private static void waitingTest(){
        Object obj = new Object();
        Thread targetTh = new Thread(()->{
            synchronized (obj) {
                try {
                    obj.wait();
                } catch (InterruptedException e) {
                    System.out.println("线程中断状态被清除为:"+Thread.currentThread().isInterrupted());
                    Thread.currentThread().interrupt();
                    System.out.println("线程中断状态重置为:"+Thread.currentThread().isInterrupted());
                } finally {
                    // 添加死循环，以便外部检查线程中断状态重置后的值
                    for (;;){
                    }
                }
            }
        });
        targetTh.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程状态:"+targetTh.getState().name());
        targetTh.interrupt();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 线程外部检查线程中断状态时，线程不能为终止状态（即线程不能执行完成）否则会返回false
        System.out.println("线程targetTh中断状态为:"+targetTh.isInterrupted());
    }


    /**
     * 阻塞状态synchronized
     */
    private static void blockedTest() {
        Object obj = new Object();
        Thread thread1 = new Thread(()->{
            synchronized (obj) {
                for(;;);
            }
        });
        Thread targetTh = new Thread(()->{
            synchronized (obj) {
                for(;;);
            }
        });
        thread1.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        targetTh.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程thread1状态:"+thread1.getState().name());
        System.out.println("线程targetTh状态:"+targetTh.getState().name());
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        targetTh.interrupt();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 线程状态被重置为true，但是因等待同步锁，线程还是阻塞状态.
        System.out.println("线程targetTh中断状态为:"+targetTh.isInterrupted());
        System.out.println("线程targetTh状态:"+targetTh.getState().name());
    }

    /**
     * 阻塞状态lock
     */
    private static void blockedTest1() {
        Object obj = new Object();
        Lock lock = new ReentrantLock();
        Thread thread1 = new Thread(()->{
            lock.lock();
            for(;;);
        });
        Thread targetTh = new Thread(()->{
            try {
                lock.lockInterruptibly();
            } catch (InterruptedException e) {
                System.out.println("线程内部中断状态:"+Thread.currentThread().isInterrupted());
            }
            for(;;);
        });
        thread1.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        targetTh.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程thread1状态:"+thread1.getState().name());
        System.out.println("线程targetTh状态:"+targetTh.getState().name());
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        targetTh.interrupt();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程targetTh中断状态为:"+targetTh.isInterrupted());
        System.out.println("线程targetTh状态:"+targetTh.getState().name());
    }
}
```

参考资料:

[Java 并发结束任务、中断线程 :blocked、interrupted](https://blog.csdn.net/SUKI547/article/details/101350176)

[Java线程源码解析之interrupt](https://www.jianshu.com/p/1492434f2810)



### 1.3.2 JOIN

```java
// 等待该线程终止，一直等待
public final void join() throws InterruptedException;
// 等待一定时限
public final synchronized void join(long millis) throws InterruptedException;
// 等待一定时限
public final synchronized void join(long millis, int nanos)
    throws InterruptedException;
```

​		 当我们调用某个线程的这个方法时，这个方法会挂起调用线程，直到被调用线程结束执行，调用线程才会继续执行。

* 三个方法都被final修饰，无法被子类重写。
* `join(long)`, `join(long, long)` 是synchronized method，同步的对象是当前线程实例。
* 无参版本和两个参数版本最终都调用了一个参数的版本。
* `join() `和` join(0)` 是等价的，表示一直等下去；`join(非0)`表示等待一段时间。从源码可以看到 join(0) 调用了Object.wait(0)，其中Object.wait(0) 会一直等待，直到被notify/中断才返回。`while(isAlive())`是为了防止子线程伪唤醒(spurious wakeup)，只要子线程没有TERMINATED的，父线程就需要继续等下去。
* `join() `和 `sleep() `一样，可以被中断（被中断时，会抛出 InterrupptedException 异常）；不同的是，`join()` 内部调用了 `wait()`，会出让锁，而 sleep() 会一直保持锁。

**代码示例：**

```java
public class ThreadJoinTest {
  public static void main(String[] args) throws InterruptedException {
      ThreadB threadB = new ThreadB();
      ThreadA threadA = new ThreadA(threadB);
      threadA.start();
      System.out.println("线程A在JOIN前的状态："+threadA.getState().name());
      TimeUnit.SECONDS.sleep(3);
      System.out.println("线程A在JOIN中的状态："+threadA.getState().name());
  }
}

class ThreadA extends Thread {

    public ThreadB threadB;

    public ThreadA(ThreadB threadB) {
        this.threadB = threadB;
    }

    @Override
    public void run() {
        threadB.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程B在JOIN前的状态："+threadB.getState().name());
        try {
            threadB.join(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程A在JOIN后的状态："+getState().name());
        System.out.println("线程B在JOIN后的状态："+threadB.getState().name());
    }
}

class ThreadB extends Thread {
    @Override
    public void run() {
        for(;;) {

        }
    }
}
```

*执行结果：*

```shell
线程A在JOIN前的状态：RUNNABLE
线程B在JOIN前的状态：RUNNABLE
线程A在JOIN中的状态：TIMED_WAITING
线程A在JOIN后的状态：RUNNABLE
线程B在JOIN后的状态：RUNNABLE
```

*分析：*

​		ThreadA.run() -> ThreadB.join() -> ThreadB.join(0) -> ThreadB.wait(0)（此时 ThreadA线程会获得 ThreadB实例作为锁，其他线程可以进入 ThreadB.join() ，但不可以进入 ThreadB.join(0)， 因为ThreadB.join(0)是同步方法）。

​		如果 ThreadB线程是 Active，则调用 ThreadB.wait(0)（为了防止子线程 spurious wakeup, 需要将 wait(0) 放入 while(isAlive()) 循环中。

​		一旦 ThreadB线程不为 Active （状态为 TERMINATED）, ThreadB.notifyAll() 会被调用-> ThreadB.wait(0)返回 -> ThreadB.join(0)返回 -> ThreadB.join()返回 -> ThreadA.run()继续执行, 子线程会调用this.notify()，ThreadB.wait(0)会返回到ThreadB.join(0) ，ThreadB.join(0)会返回到 ThreadB.join(), ThreadB.join() 会返回到 ThreadA父线程，ThreadA 父线程就可以继续运行下去了。

**注意：**

* 子线程结束之后，"会唤醒主线程"，父线程重新获取cpu执行权，继续运行

* 在调用 join() 方法的程序中，原来的多个线程仍然多个线程，并没有发生“合并为一个单线程”。真正发生的是调用 join() 的线程进入 TIMED_WAITING 状态，等待 join() 所属线程运行结束后再继续运行。