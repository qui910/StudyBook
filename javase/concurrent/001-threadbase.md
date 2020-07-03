# 1 线程基础

​		线程是并发的基础单元，这里从线程开始说明。实现多线程有以下两种方式：

```java
class TestThreadStateA extends Thread {
    @Override
    public void run() {
}
```

```java
        new Thread(()->{
            System.out.println("sub localVariable is "+localVariable.get());
        }).start();
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

### 1.3.3 YIELD

```java
// 让出CPU执行权限
public static native void yield();
```

​		当一个线程调用yield方法时，告知线程调度器此线程请求让出自己的CPU使用，但是线程调度器可以忽略此请求。当线程让出CPU使用权限后，就处理就绪状态。线程调度器会从线程的就绪队列中获取一个线程优先级最高的线程。

# 2 线程通知与等待

​		在Object.java中，定义了`wait()`, `notify()`和`notifyAll()`等接口。`wait()`的作用是让当前线程进入等待状态，同时，`wait()`也会让当前线程释放它所持有的锁。而`notify()`和`notifyAll()`的作用，则是唤醒当前对象上的等待线程；`notify()`是唤醒单个线程，而notifyAll()是唤醒所有的线程。

​		调用任意对象的 `wait()` 方法导致线程等待，并且该对象上的锁被释放。而调用任意对象的notify()方法则导致因调用该对象的 wait() 方法而等待的线程中随机选择的一个解除等待（但要等到获得锁后才真正可执行）。 

​		无论是`wait()`或`notify()`都必须首先获得该对象的监视器锁（synchronized）。

​		Object类中关于等待/唤醒的API详细信息如下：		

| 方法                          | 说明                                                         |
| :---------------------------- | ------------------------------------------------------------ |
| notify()                      | 唤醒在此对象监视器上等待的单个线程。                         |
| notifyAll()                   | 唤醒在此对象监视器上等待的所有线程。                         |
| wait()                        | 让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法”，当前线程被唤醒(进入“就绪状态”)。 |
| wait(long timeout)            | 让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量”，当前线程被唤醒(进入“就绪状态”)。 |
| wait(long timeout, int nanos) | 让当前线程处于“等待(阻塞)状态”，“直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量”，当前线程被唤醒(进入“就绪状态”)。 |

代码示例：

```java
/**
 * 线程通信测试
 */
public class ThreadWaitNotifyTest {

    // 生产者
    class ThreadA extends Thread {

        private Deque<String> queue;
        private int maxSize;

        public ThreadA(Deque<String> queue, int maxSize) {
            this.queue = queue;
            this.maxSize = maxSize;
        }

        @Override
        public void run() {
            while (true) {
                synchronized (queue) {
                  // 队列超过100个后，停止生产，进入等待
                  while (queue.size() == maxSize) {
                    try {
                        queue.notifyAll();
                        queue.wait();
                      System.out.println("Queue is Full");
                    } catch (InterruptedException e) {
                      e.printStackTrace();
                    }
                  }

                  Random random = new Random();
                  int i = random.nextInt();
                  String msg = "test" + i;
                  queue.add(msg);
                  System.out.println("生产：" + msg);
                }
            }
        }
    }

    class ThreadB extends Thread {

        private Deque<String> queue;
        private int maxSize;

        public ThreadB(Deque<String> queue, int maxSize) {
            this.queue = queue;
            this.maxSize = maxSize;
        }

        @Override
        public void run() {
            while (true) {
                synchronized (queue) {
                  // 队列空后，停止消费，进入等待
                  while (queue.size() == 0) {
                    try {
                        queue.notifyAll();
                        queue.wait();
                      System.out.println("Queue is Empty");
                    } catch (InterruptedException e) {
                      e.printStackTrace();
                    }
                  }

                  String msg = queue.remove();
                  System.out.println("消费：" + msg);
                }
            }
        }
    }

    public static void main(String[] args) {

        LinkedList<String> linkedList = new LinkedList<String>();
        int maxSize=2;

        ThreadA threadA = new ThreadWaitNotifyTest().new ThreadA(linkedList,maxSize);
        ThreadB threadB = new ThreadWaitNotifyTest().new ThreadB(linkedList,maxSize);
        threadA.start();
        threadB.start();
    }
}
```

注意：`notifyAll()`必须在`wait()`操作前触发，示例中`notifyAll()`也可以改在每次生成或消费完元素之后（即`System.out.println`语句之后），效果是一样的。

# 3 线程死锁

​		死锁就是两个或两个以上的线程在执行过程中，因争夺资源而造成的互相等待的现象。在无外在处理的情况下，这些线程会一直相互等待而无法继续运行。

​		死锁产生必备以下四个条件：

* 互斥条件：指线程对已经获取到的资源进行排他使用，即该资源同时只由一个线程占用。如果此时还有其他线程请求获取该资源，则请求线程只能等待，直至占用资源的线程释放该资源。
* 请求并持有条件：指一个线程已经持有了至少一个资源，但又提出了新的资源请求。而新的资源已经被其他线程所占用，所以当前线程会被阻塞，但阻塞的同时不会释放自己已经获取的资源。
* 不可剥夺条件：指线程获取到的资源在自己使用完之前不能被其他线程抢占。只有在自己使用完毕后才由自己释放。
* 环路等待条件：指在发生死锁时，必然存在一个线程--资源的环形链。即线程集合{T0，T1，T2....Tn} 中的T0正在等待一个T1占用的资源，T1 正在等待T2占用的资源，。。。Tn正在等待T0占用的资源。

## 3.1 代码示例

```java
/**
 * 线程死锁测试
 */
public class ThreadDeadLockTest {

  public static void main(String[] args) {
      Object resoureA = new Object();
      Object resoureB = new Object();

      new Thread(()->{
          synchronized (resoureA) {
              System.out.println(Thread.currentThread().getName()+" get resoureA");
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }

              synchronized (resoureB){
                  System.out.println(Thread.currentThread().getName()+" get resoureB");
              }
          }
      }).start();

      new Thread(()->{
          synchronized (resoureB) {
              System.out.println(Thread.currentThread().getName()+" get resoureB");
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }

              synchronized (resoureA){
                  System.out.println(Thread.currentThread().getName()+" get resoureA");
              }
          }
      }).start();
  }
}
```

运行结果：

```shell
Connected to the target VM, address: '127.0.0.1:35607', transport: 'socket'
Thread-0 get resoureA
Thread-1 get resoureB
```

程序一直等待，卡死。

## 3.2 分析

使用JVM中`jstack -l <pid>`来观察线程情况：

```shell
ull thread dump Java HotSpot(TM) 64-Bit Server VM (25.191-b12 mixed mode):

"DestroyJavaVM" #14 prio=5 os_prio=0 tid=0x00007f079800f800 nid=0x2fe3 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" #13 prio=5 os_prio=0 tid=0x00007f07982f1000 nid=0x3002 waiting for monitor entry [0x00007f07819dd000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.prd.concurrent.ThreadDeadLockTest.lambda$main$1(ThreadDeadLockTest.java:39)
	- waiting to lock <0x0000000782427280> (a java.lang.Object)
	- locked <0x0000000782427290> (a java.lang.Object)
	at com.prd.concurrent.ThreadDeadLockTest$$Lambda$2/1360767589.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Thread-0" #12 prio=5 os_prio=0 tid=0x00007f07982ef000 nid=0x3001 waiting for monitor entry [0x00007f0781ade000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.prd.concurrent.ThreadDeadLockTest.lambda$main$0(ThreadDeadLockTest.java:24)
	- waiting to lock <0x0000000782427290> (a java.lang.Object)
	- locked <0x0000000782427280> (a java.lang.Object)
	at com.prd.concurrent.ThreadDeadLockTest$$Lambda$1/2137211482.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Service Thread" #11 daemon prio=9 os_prio=0 tid=0x00007f079827f000 nid=0x2fff runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #10 daemon prio=9 os_prio=0 tid=0x00007f079827b800 nid=0x2ffe waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #9 daemon prio=9 os_prio=0 tid=0x00007f0798279800 nid=0x2ffd waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #8 daemon prio=9 os_prio=0 tid=0x00007f0798272800 nid=0x2ffc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Command Reader" #7 daemon prio=10 os_prio=0 tid=0x00007f075c001000 nid=0x2ffb runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Event Helper Thread" #6 daemon prio=10 os_prio=0 tid=0x00007f0798192800 nid=0x2ff9 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Transport Listener: dt_socket" #5 daemon prio=10 os_prio=0 tid=0x00007f079818f000 nid=0x2ff7 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f0798182800 nid=0x2ff6 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f0798150800 nid=0x2ff5 in Object.wait() [0x00007f07826ed000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000782108ed0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x0000000782108ed0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f079814e000 nid=0x2ff4 in Object.wait() [0x00007f07827ee000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000782106bf8> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x0000000782106bf8> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=0 tid=0x00007f0798144800 nid=0x2ff2 runnable 

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00007f0798025000 nid=0x2fe7 runnable 

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00007f0798027000 nid=0x2fe9 runnable 

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00007f0798028800 nid=0x2feb runnable 

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00007f079802a800 nid=0x2fec runnable 

"VM Periodic Task Thread" os_prio=0 tid=0x00007f0798284000 nid=0x3000 waiting on condition 

JNI global references: 2483


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f0764006528 (object 0x0000000782427280, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f07640039d8 (object 0x0000000782427290, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
	at com.prd.concurrent.ThreadDeadLockTest.lambda$main$1(ThreadDeadLockTest.java:39)
	- waiting to lock <0x0000000782427280> (a java.lang.Object)
	- locked <0x0000000782427290> (a java.lang.Object)
	at com.prd.concurrent.ThreadDeadLockTest$$Lambda$2/1360767589.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
"Thread-0":
	at com.prd.concurrent.ThreadDeadLockTest.lambda$main$0(ThreadDeadLockTest.java:24)
	- waiting to lock <0x0000000782427290> (a java.lang.Object)
	- locked <0x0000000782427280> (a java.lang.Object)
	at com.prd.concurrent.ThreadDeadLockTest$$Lambda$1/2137211482.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.

Heap
 PSYoungGen      total 55808K, used 10609K [0x0000000782100000, 0x0000000785f00000, 0x00000007c0000000)
  eden space 48128K, 22% used [0x0000000782100000,0x0000000782b5c430,0x0000000785000000)
  from space 7680K, 0% used [0x0000000785780000,0x0000000785780000,0x0000000785f00000)
  to   space 7680K, 0% used [0x0000000785000000,0x0000000785000000,0x0000000785780000)
 ParOldGen       total 126976K, used 0K [0x0000000706200000, 0x000000070de00000, 0x0000000782100000)
  object space 126976K, 0% used [0x0000000706200000,0x0000000706200000,0x000000070de00000)
 Metaspace       used 4233K, capacity 4784K, committed 4992K, reserved 1056768K
  class space    used 478K, capacity 536K, committed 640K, reserved 1048576K
```

首先Thread1 和 Thread0 都在获得监视器对象锁是被阻塞，状态都为BLOCKED

```shell
"Thread-1" #13 prio=5 os_prio=0 tid=0x00007f07982f1000 nid=0x3002 waiting for monitor entry [0x00007f07819dd000]
   java.lang.Thread.State: BLOCKED (on object monitor)
   
"Thread-0" #12 prio=5 os_prio=0 tid=0x00007f07982ef000 nid=0x3001 waiting for monitor entry [0x00007f0781ade000]
   java.lang.Thread.State: BLOCKED (on object monitor)
```

其次`jstack`的日志中就已经检测出死锁的信息：

```shell
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f0764006528 (object 0x0000000782427280, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f07640039d8 (object 0x0000000782427290, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
```

## 3.3  死锁的处理

​		样例中会发生死锁，重要的原因就是线程A 先申请资源A，则申请资源B。而线程B则是先申请资源B，在申请资源A。发生了资源争抢所导致的。只要两个线程都先申请资源A，再申请资源B就可以解决（当然，这是在业务逻辑允许的情况下）

# 4 ThreadLocal

## 4.1 概述

​		ThreadLocal类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其get或set方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。ThreadLocal实例通常是类中的 **private static**字段，它们希望将状态与某一个线程（例如，用户ID 或事务ID）相关联。

​		ThreadLocal与线程同步机制不同，线程同步机制是多个线程共享同一个变量，而ThreadLocal是为**每一个线程创建一个单独的变量副本**，故而每个线程都可以独立地改变自己所拥有的变量副本，而不会影响其他线程所对应的副本。可以说ThreadLocal为多线程环境下变量问题提供了另外一种解决思路。

​		ThreadLocal定义了四个方法：

```java
// 返回此线程局部变量的当前线程副本中的值。
public T get();
// 返回此线程局部变量的当前线程的“初始值”。
protected T initialValue();
// 移除此线程局部变量当前线程的值。
public void remove();
// 将此线程局部变量的当前线程副本中的值设置为指定值。
public void set(T value);
```


​		ThreadLocal内部还有一个静态内部类ThreadLocalMap，该内部类才是实现线程隔离机制的关键，get()、set()、remove()都是基于该内部类操作。ThreadLocalMap提供了一种用键值对方式存储每一个线程的变量副本的方法，key为当前ThreadLocal对象，value则是对应线程的变量副本。

​		对于ThreadLocal需要注意的有两点：

* ThreadLocal实例本身是不存储值，它只是提供了一个在当前线程中找到副本值得key。
* 是ThreadLocal包含在Thread中，而不是Thread包含在ThreadLocal中。
  ThreadLocal，Thread，ThreadLocalMap 关系如下：实线表示强引用，虚线表示弱引用

![thread-003](..\images\thread-003.png)

* ThreadLocalMap类的定义在ThreadLocal，但是每个Thread维护一个ThreadLocalMap映射表，这个映射表的key是ThreadLocal实例本身，value是真正需要存储的Object。
* ThreadLocal变量的活动范围为某线程，是该线程“专有的，独自霸占”的，对该变量的所有操作均由该线程完成！也就是说，ThreadLocal 不是用来解决共享对象的多线程访问的竞争问题的，因为ThreadLocal.set() 到线程中的对象是该线程自己使用的对象，其他线程是不需要访问的，也访问不到的。当线程终止后，这些值会作为垃圾回收。

## 4.2 使用示例

```java
public class SeqCount {
    private static ThreadLocal<Integer> seqCount = new ThreadLocal<Integer>(){
        // 实现initialValue()
        public Integer initialValue() {
            return 0;
        }
    };
    public int nextSeq(){
        seqCount.set(seqCount.get() + 1);
        return seqCount.get();
    }
    public static void main(String[] args){
        SeqCount seqCount = new SeqCount();
        SeqThread thread1 = new SeqThread(seqCount);
        SeqThread thread2 = new SeqThread(seqCount);
        SeqThread thread3 = new SeqThread(seqCount);
        SeqThread thread4 = new SeqThread(seqCount);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
    }
    private static class SeqThread extends Thread{
        private SeqCount seqCount;
        SeqThread(SeqCount seqCount){
            this.seqCount = seqCount;
        }
        public void run() {
            for(int i = 0 ; i < 3 ; i++){
                System.out.println(Thread.currentThread().getName() + " seqCount :" + seqCount.nextSeq());
            }
        }
    }
}
```

运行结果：

```shell
Thread-1 seqCount :1
Thread-1 seqCount :2
Thread-1 seqCount :3
Thread-2 seqCount :1
Thread-2 seqCount :2
Thread-2 seqCount :3
Thread-3 seqCount :1
Thread-3 seqCount :2
Thread-3 seqCount :3
Thread-0 seqCount :1
Thread-0 seqCount :2
Thread-0 seqCount :3
```

从运行结果可以看出，ThreadLocal确实是可以达到线程隔离机制，确保变量的安全性。

## 4.3 源码解析

### 4.3.1 ThreadLocalMap

​		ThreadLocalMap是实现ThreadLocal的关键，其内部利用Entry来实现key-value的存储。

```java
static class Entry extends WeakReference<ThreadLocal> {
	/** The value associated with this ThreadLocal. */
	Object value;

	Entry(ThreadLocal k, Object v) {
		super(k);
		value = v;
	}
}
```

​		从上面代码中可以看出Entry的key就是ThreadLocal，而value就是值。同时，Entry也继承WeakReference，所以说Entry所对应key（ThreadLocal实例）的引用为一个弱引用（这里使用弱引用就是防止内存溢出，在GC时优先回收这部分数据，具体请参考[用弱引用堵住内存泄漏](https://www.ibm.com/developerworks/cn/java/j-jtp11225/)）

​		ThreadLocalMap的源码最核心的方法getEntry()、set(ThreadLocal> key, Object value)。

####  4.3.1.1 set方法

​		这个set()操作和我们在集合了解的put()方式有点儿不一样，虽然他们都是key-value结构，不同在于他们解决散列冲突的方式不同。集合Map的put()采用的是拉链法，而ThreadLocalMap的set()则是采用开放定址法（具体请参考[散列冲突处理系列博客](http://www.nowamagic.net/academy/detail/3008015)）。掌握了开放地址法该方法就一目了然了。

```java
private void set(ThreadLocal<?> key, Object value) {
	ThreadLocal.ThreadLocalMap.Entry[] tab = table;
	int len = tab.length;
	// 根据ThreadLocal的散列值，查找对应元素在数组中的位置
	int i = key.threadLocalHashCode & (len-1);

	// 采用“线性探测法”，寻找合适位置
	for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
		e != null;
		e = tab[i = nextIndex(i, len)]) {
	
		ThreadLocal<?> k = e.get();
	
		// key 存在，直接覆盖
		if (k == key) {
			e.value = value;
			return;
		}
	
		// key == null，但是存在值（因为此处的e != null），说明之前的ThreadLocal对象已经被回收了
		if (k == null) {
			// 用新元素替换陈旧的元素
			replaceStaleEntry(key, value, i);
			return;
		}
	}
	
	// ThreadLocal对应的key实例不存在也没有陈旧元素，new 一个
	tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
	
	int sz = ++size;
	
	// cleanSomeSlots 清除陈旧的Entry（key == null）
	// 如果没有清理陈旧的 Entry 并且数组中的元素大于了阈值，则进行 rehash
	if (!cleanSomeSlots(i, sz) && sz >= threshold)
		rehash();

}
```

​		set()操作除了存储元素外，还有一个很重要的作用，就是replaceStaleEntry()和cleanSomeSlots()，这两个方法可以清除掉key == null 的实例，防止内存泄漏。在set()方法中还有一个变量很重要：threadLocalHashCode，定义如下：

```java
private final int threadLocalHashCode = nextHashCode();
```

​		从名字上面我们可以看出threadLocalHashCode应该是ThreadLocal的散列值，定义为final，表示ThreadLocal一旦创建其散列值就已经确定了，生成过程则是调用`nextHashCode()`：

```java
private static AtomicInteger nextHashCode = new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;

private static int nextHashCode() {
   return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

​		`nextHashCode`表示分配下一个ThreadLocal实例的threadLocalHashCode的值，`HASH_INCREMENT`则表示分配两个ThradLocal实例的threadLocalHashCode的增量，从`nextHashCode`就可以看出他们的定义。

####  4.3.1.2 getEntry()
```java
private Entry getEntry(ThreadLocal<?> key) {
	int i = key.threadLocalHashCode & (table.length - 1);
	Entry e = table[i];
	if (e != null && e.get() == key)
		return e;
	else
		return getEntryAfterMiss(key, i, e);
}
```

​		由于采用了开放定址法，所以当前key的散列值和元素在数组的索引并不是完全对应的，首先取一个探测数（key的散列值），如果所对应的key就是我们所要找的元素，则返回，否则调用getEntryAfterMiss()，如下：

```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
	Entry[] tab = table;
	int len = tab.length;

	while (e != null) {
		ThreadLocal<?> k = e.get();
		if (k == key)
			return e;
		if (k == null)
			expungeStaleEntry(i);
		else
			i = nextIndex(i, len);
		e = tab[i];
	}
	return null;
}
```


​		这里有一个重要的地方，当key == null时，调用了expungeStaleEntry()方法，该方法用于处理key == null，有利于GC回收，能够有效地避免内存泄漏。

### 4.3.2.get()

​	返回当前线程所对应的线程变量

```java
public T get() {
	// 获取当前线程
	Thread t = Thread.currentThread();

	// 获取当前线程的成员变量 threadLocal
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		// 从当前线程的ThreadLocalMap获取相对应的Entry
		ThreadLocalMap.Entry e = map.getEntry(this);
		if (e != null) {
			@SuppressWarnings("unchecked")
	
			// 获取目标值        
			T result = (T)e.value;
			return result;
		}
	}
	return setInitialValue();
}
```

​		首先通过当前线程获取所对应的成员变量ThreadLocalMap，然后通过ThreadLocalMap获取当前ThreadLocal的Entry，最后通过所获取的Entry获取目标值result。

​		getMap()方法可以获取当前线程所对应的ThreadLocalMap，如下：

```java
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

### 4.3.3 set(T value)
​	设置当前线程的线程局部变量的值。

```java
public void set(T value) {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
}
```

​		获取当前线程所对应的ThreadLocalMap，如果不为空，则调用ThreadLocalMap的set()方法，key就是当前ThreadLocal，如果不存在，则调用createMap()方法新建一个，如下：

```java
void createMap(Thread t, T firstValue) {
	t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

### 4.3.4 initialValue()
​	返回该线程局部变量的初始值。

```
protected T initialValue() {
	return null;
}
```


​		该方法定义为protected级别且返回为null，很明显是要子类实现它的，所以我们在使用ThreadLocal的时候一般都应该覆盖该方法。该方法不能显示调用，只有在第一次调用get()或者set()方法时才会被执行，并且仅执行1次。

### 4.3.5 remove()
​	将当前线程局部变量的值删除。

```java
public void remove() {
	ThreadLocalMap m = getMap(Thread.currentThread());
	if (m != null)
		m.remove(this);
}
```


​		该方法的目的是减少内存的占用。当然，我们不需要显示调用该方法，因为一个线程结束后，它所对应的局部变量就会被垃圾回收。

## 4.4 ThreadLocal内存泄漏

​		前面提到每个Thread都有一个ThreadLocal.ThreadLocalMap的map，该map的key为ThreadLocal实例，它为一个弱引用，我们知道弱引用有利于GC回收。当ThreadLocal的key == null时，GC就会回收这部分空间，但是value却不一定能够被回收，因为他还与Current Thread存在一个强引用关系。

​		由于存在这个强引用关系，会导致value无法回收。如果这个线程对象不会销毁那么这个强引用关系则会一直存在，就会出现内存泄漏情况。所以说只要这个线程对象能够及时被GC回收，就不会出现内存泄漏。如果碰到线程池，那就更坑了。

​		那么要怎么避免这个问题呢？**在前面提过，在ThreadLocalMap中的setEntry()、getEntry()，如果遇到key == null的情况，会对value设置为null。当然我们也可以显示调用ThreadLocal的remove()方法进行处理。**

## 4.5 总结

​		ThreadLocal 不是用于解决共享变量的问题的，也不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制。这点至关重要。

​		每个Thread内部都有一个ThreadLocal.ThreadLocalMap类型的成员变量，该成员变量用来存储实际的ThreadLocal变量副本。

​		ThreadLocal并不是为线程保存对象的副本，它仅仅只起到一个索引的作用。它的主要目的是为每一个线程隔离一个类的实例，这个实例的作用范围仅限于线程内部。

​		最常见的ThreadLocal使用场景为 用来解决 数据库连接、Session管理等。

## 4.6 InheritableThreadLocal

​		ThreadLocal是不具备继承性质的，即父线程和子线程都使用一个ThreadLocal，但是各自的线程中是保留的各自的实例值。

​		为解决这个问题，使子线程能访问到父线程的Local实例。特别引入`InheritableThreadLocal`。

### 4.6.1 代码示例

```java
/**
 * InheritableThreadLocal 测试
 *
 */
public class InheritableThreadLocalTest {
    private static ThreadLocal<String> localVariable = new ThreadLocal();

    private static InheritableThreadLocal<String> inheritableLocalVariable = new InheritableThreadLocal();

    public static void main(String[] args) {
        localVariable.set("main thread");
        inheritableLocalVariable.set("main thread");
        System.out.println("main localVariable is "+localVariable.get());
        System.out.println("main inheritableLocalVariable is "+inheritableLocalVariable.get());

        testThreadLocal();

        testInheritableThreadLocal();
    }

    public static void testThreadLocal() {
        new Thread(()->{
            System.out.println("sub localVariable is "+localVariable.get());
        }).start();
    }

    public static void testInheritableThreadLocal() {
        new Thread(()->{
            System.out.println("sub inheritableLocalVariable is "+inheritableLocalVariable.get());
        }).start();
    }
}
```

### 4.6.2 源码解析

​		InheritableThreadLocal继承自ThreadLocal，其提供了一个特性，就是让子线程可以访问父线程设置的本地变量。

见书《Java并发编程之美第一章》



