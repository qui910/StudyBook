# 1 LockSupport

​		JDK中的rt.jar包里面的LockSupport是个工具类，它的主要作用是挂起和唤醒线程，是创建锁和其他同步类的基础。

​		LockSupport类与每个使用它的线程都会关联一个许可证，在默认情况下调用LockSupport类的方法的线程是不持有许可证的。LockSupport是使用Unsafe类实现的。

## 1.1 常用API

```java
// 如果调用park方法的线程thread1已经拿到了与LockSupport关联的许可证，则调用park时马上会返回，否则thread1被阻塞，状态为WAITING。
// 其他线程调用unpark并且将thread1作为参数时，thread1会从park的阻塞状态返回。另外，如果其他线程调用了thread1的interrupt时，设置了中断标志的thread1也会从阻塞状态返回。所以调用park时最好也使用循环条件判断方式
void park();

// 当一个线程调用unpark时，如果参数thread没有持有thread与LockSupport类关联的许可证，则让thread线程持有。如果thread之前因调用park挂起，则thread被唤醒。如果thread之前没有调用park，则调用unpark方法后，再调用park方法，其立刻返回。
void unpark(Thread thread);

// 和park类似，不同处在于，如果没有拿到许可证，则调用线程会被挂起nanos时间后修改为自动返回
void parkNanos(long nanos);

// Thread类里面有个变量volatile Object parkBlocker，用来存放park方法传递的blocker对象，也就是把blocker变量存放到了调用park方法的线程的成员变量里面
void park(Object blocker);

// 与park(Object blocker)类似，多了个超时时间
void parkNanos(Object blocker,long nanos);
    
// 参数deadlime为从1970到现在的毫秒数，与parkNanos类型，唯一的不同时是parkNanos是等待的毫秒数，而parkUntil是具体的某个时间点   
void parkUntil(Object blocker,long deadline);
```

## 1.2 几个示例

**示例1**：

```java
    /**
     * 测试常规用法
     * 线程test1 调用park时，因无许可证，被阻塞为WAITING
     * 阻塞后必须由其他线程调用unpark才能解除阻塞
     *
     */
    private static void testNormalUsed() {
        Thread test1 = new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("park前 线程状态:"+Thread.currentThread().getState().name());
            LockSupport.park();
            System.out.println("park后 线程状态:"+Thread.currentThread().getState().name());
        });
        test1.start();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("开始unpark前，park中线程状态："+test1.getState().name());
        LockSupport.unpark(test1);
        System.out.println("结束unpark");
    }
```

结果：

```shell
park前 线程状态:RUNNABLE
开始unpark前，park中线程状态：WAITING
结束unpark
park后 线程状态:RUNNABLE
```

**示例2**：

```java
    /**
     * 测试线程test2被park阻塞后，可以被中断
     * 且中断后 run方法执行完成。
     * 并测试park时，使用循环条件判断
     */
    private  static void testNormalInterrupte() {
        Thread test2 = new Thread(()->{
            while(!Thread.currentThread().isInterrupted()) {
                System.out.println("park前，线程中断状态："+Thread.currentThread().isInterrupted());
                LockSupport.park();
                System.out.println("park后，线程中断状态："+Thread.currentThread().isInterrupted());
            }
            System.out.println("dosome things");
        });
        test2.start();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 因为使用while循环判断，这里unpark后，线程马上又重新进入park阻塞状态，只有中断才能最终真正的唤醒线程
        System.out.println("开始unpark前，park中线程中断状态："+test2.isInterrupted());
        LockSupport.unpark(test2);
        System.out.println("结束unpark，线程的状态："+test2.getState().name());

        System.out.println("中断前，park中线程中断状态："+test2.isInterrupted());
        test2.interrupt();
        System.out.println("结束中断");
    }
```

结果：

```shell
park前，线程中断状态：false
开始unpark前，park中线程中断状态：false
park后，线程中断状态：false
park前，线程中断状态：false
结束unpark，线程的状态：WAITING
中断前，park中线程中断状态：false
结束中断
park后，线程中断状态：true
dosome things
```

## 1.3 使用LockSupport自定义锁

```java
/**
 * 自定义先进先出的锁
 */
public class MyLockSupportLock {
    /**
     * 锁被获取的标记
     */
    private AtomicBoolean locked = new AtomicBoolean();
    /**
     * 等待锁的队列
     */
    private Queue<Thread> waiters = new ConcurrentLinkedDeque<Thread>();


    public void lock() {
        boolean wasInterrupted = false;
        //  当前线程加入等待队列
        Thread current = Thread.currentThread();
        waiters.add(current);

        // 非队列首的线程进行阻塞
        Thread tmp = waiters.peek();
        while(tmp!=current || !locked.compareAndSet(false,true)) {
            LockSupport.park(current);
            // 获取中断,并清除中断
            if (Thread.interrupted()) {
                wasInterrupted = true;
            }

            // 重新回复中断状态
            if (wasInterrupted) {
                current.interrupt();
            }
        }
        waiters.remove();
    }

    public void unlock() {
        locked.set(false);
        LockSupport.unpark(waiters.peek());
    }

    public static void main(String[] args) {

        MyLockSupportLock lock = new MyLockSupportLock();

        Runnable runA =
            () -> {
              try {
                TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              lock.lock();
              System.out.println("线程"+Thread.currentThread().getName()+"获得锁");
              try {
                TimeUnit.SECONDS.sleep(5);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              lock.unlock();
                System.out.println("线程"+Thread.currentThread().getName()+"释放锁");
            };

        Thread thread1 = new Thread(runA,"thread1");
        Thread thread2 = new Thread(runA,"thread2");
        thread1.start();
        thread2.start();
    }
}
```

测试结果：

```
线程thread1获得锁
线程thread1释放锁
线程thread2获得锁
线程thread2释放锁
```

# 2 AQS

​		Java的内置锁一直都是备受争议的，在JDK 1.6之前，synchronized这个重量级锁其性能一直都是较为低下，虽然在1.6后，进行大量的锁优化策略,但是与Lock相比synchronized还是存在一些缺陷的：虽然synchronized提供了便捷性的隐式获取锁释放锁机制（基于JVM机制），但是它却缺少了获取锁与释放锁的可操作性，可中断、超时获取锁，且它为独占式在高并发场景下性能大打折扣。

​		在介绍Lock之前，我们需要先熟悉一个非常重要的组件，掌握了该组件JUC包下面很多问题都不在是问题了。该组件就是AQS。

​		AQS，AbstractQueuedSynchronizer，即队列同步器。它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。它是JUC并发包中的核心基础组件。

​		AQS解决了子类实现同步器时，涉及当的大量细节问题，例如获取同步状态、FIFO同步队列。基于AQS来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

​		在基于AQS构建的同步器中，只能在一个时刻发生阻塞，从而降低上下文切换的开销，提高了吞吐量。同时在设计AQS时充分考虑了可伸缩行，因此J.U.C中所有基于AQS构建的同步器均可以获得这个优势。

​		**AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态**。

## 2.1 AQS锁分类

​		AQS锁的类别分为“独占锁”和“共享锁”两种:

* 独占锁 -- 锁在一个时间点只能被一个线程锁占有。根据锁的获取机制，它又划分为“公平锁”和“非公平锁”。公平锁，是按照通过CLH等待线程按照先来先得的规则，公平的获取锁；而非公平锁，则当线程要获取锁时，它会无视CLH等待队列而直接获取锁。独占锁的典型实例子是ReentrantLock，此外，ReentrantReadWriteLock.WriteLock也是独占锁。
* 共享锁 -- 能被多个线程同时拥有，能被共享的锁。JUC包中的ReentrantReadWriteLock.ReadLock，CyclicBarrier， CountDownLatch和Semaphore都是共享锁。这些锁的用途和原理，在以后的章节再详细介绍。

## 2.2 AbstractOwnableSynchronizer

​		AbstractOwnableSynchronizer是AbstractQueuedSynchronizer的父类，该类只有一个Thread类型的成员变量exclusiveOwnerThread，用来保存是哪个线程获得了独占锁或者写锁，当该值为null时，表示没有线程获得独占锁或者写锁。

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private static final long serialVersionUID = 3737899427754241961L;

    protected AbstractOwnableSynchronizer() { }

    //获得独占锁或者写锁的线程
    private transient Thread exclusiveOwnerThread;	

    //设置线程
    protected final void setExclusiveOwnerThread(Thread thread) {	
        exclusiveOwnerThread = thread;
    }
	//获取线程
    protected final Thread getExclusiveOwnerThread() {	
        return exclusiveOwnerThread;
    }
}
```

## 2.3 实现思路

​		AQS内部维护一个CLH队列来管理锁。

* 线程会首先尝试获取锁，如果失败，则将当前线程以及等待状态等信息包成一个Node节点加到同步队列里。
* 接着会不断循环尝试获取锁（条件是当前节点为head的直接后继才会尝试）,如果失败则会阻塞自己，直至被唤醒；
* 而当持有锁的线程释放锁时，会唤醒队列中的后继线程。

### 2.3.1 如何获取锁

​		获取锁的思路很直接：

```java
while (不满足获取锁的条件) {
    把当前线程包装成节点插入同步队列
    if (需要阻塞当前线程)
        阻塞当前线程直至被唤醒
}
将当前线程从同步队列中移除
```

​		以上是一个很简单的获取锁的伪代码流程，AQS的具体实现比这个复杂一些，也稍有不同，但思想上是与上述伪代码契合的。

​		通过循环检测是否能够获取到锁，如果不满足，则可能会被阻塞，直至被唤醒。

### 2.3.2 如何释放锁

​		释放锁的过程设计修改同步状态，以及唤醒后继等待线程：

```java
修改同步状态
if (修改后的状态允许其他线程获取到锁)
    唤醒后继线程
```

​		这只是很简略的释放锁的伪代码示意，AQS具体实现中能看到这个简单的流程模型。

## 2.4 成员变量

​		AQS使用一个int类型的成员变量state来表示同步状态，当state>0时表示已经获取了锁，当state = 0时表示释放了锁。它提供了三个方法（getState()、setState(int newState)、compareAndSetState(int expect,int update)）来对同步状态state进行操作，当然AQS通过CAS操作确保对state的操作是安全的。

```java
private volatile int state;
```

## 2.5 常用API

​		AQS主要提供了如下一些方法：

```java
// 返回同步状态的当前值
protected final int getState()

// 设置当前同步状态
protected final void setState(int newState)

// 使用CAS设置当前状态，该方法能够保证状态设置的原子性
protected final boolean compareAndSetState(int expect, int update)

// 独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获取同步状态
protected boolean tryAcquire(int arg)

// 独占式超时获取同步状态，如果当前线程在nanos时间内没有获取到同步状态，那么将会返回false，已经获取则返回true
public final boolean tryAcquireNanos(int arg, long nanosTimeout)

// 独占式释放同步状态
protected boolean tryRelease(int arg)

// 共享式获取同步状态，返回值大于等于0则表示获取成功，否则获取失败
protected int tryAcquireShared(int arg)
  
// 共享式获取同步状态，增加超时限制   
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
    
// 共享式释放同步状态
protected boolean tryReleaseShared(int arg)

// 当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占
protected boolean isHeldExclusively()

// 独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待，该方法将会调用可重写的tryAcquire(int arg)方法
public final void acquire(int arg)

// 与acquire(int arg)相同，但是该方法响应中断，当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException异常并返回
public final void acquireInterruptibly(int arg)

// 共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态
public final void acquireShared(int arg)
    
// 共享式获取同步状态，响应中断     
public final void acquireSharedInterruptibly(int arg)
    
// 独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒
public final boolean release(int arg)

// 共享式释放同步状态  
public final boolean releaseShared(int arg)
```

​		

### 2.5.1 待覆盖方法

​		子类在自定义基于AQS的同步器时，可以选择覆盖实现以下几个方法来实现同步状态的管理：

```java
// 试获取独占锁
boolean tryAcquire(int arg)
// 试释放独占锁
boolean tryRelease(int arg)
// 试获取共享锁
int tryAcquireShared(int arg)
// 试释放共享锁
boolean tryReleaseShared(int arg)
// 当前线程是否获得了独占锁
boolean isHeldExclusively()
```

​		以上的几个试获取/释放锁的方法的具体实现应当是无阻塞的。

### 2.5.2 AQS通过的模板方法

​		AQS本身将同步状态的管理用模板方法模式都封装好了，以下列举了AQS中的一些模板方法：

```java
// 获取独占锁。会调用tryAcquire方法，如果未获取成功，则会进入同步队列等待
void acquire(int arg)
// 响应中断版本的acquire
void acquireInterruptibly(int arg)
// 响应中断+带超时版本的acquire
boolean tryAcquireNanos(int arg,long nanos)
// 获取共享锁。会调用tryAcquireShared方法
void acquireShared(int arg)
// 响应中断版本的acquireShared
void acquireSharedInterruptibly(int arg)
// 响应中断+带超时版本的acquireShared
boolean tryAcquireSharedNanos(int arg,long nanos)
// 释放独占锁
boolean release(int arg)
// 释放共享锁
boolean releaseShared(int arg)
// 获取同步队列上的线程集合
Collection getQueuedThreads()
```

​		上面看上去很多方法，其实从语义上来区分就是获取和释放，从模式上区分就是独占式和共享式，从中断相应上来看就是支持和不支持。

## 2.6 内部类

### 2.6.1 Node

​		无论是等待锁的CLH队列，还是condition条件的阻塞队列，其节点都是由Node完成的。一个Node表示一个线程，它保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next），其定义如下：

```java
static final class Node {
    /** 共享 用于标记一个节点在共享模式下等待 */
    static final Node SHARED = new Node();

    /** 独占 用于标记一个节点在独占模式下等待 */
    static final Node EXCLUSIVE = null;

    /**因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态；*/
    static final int CANCELLED =  1;

    /**后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行*/
    static final int SIGNAL = -1;

    /**节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()后，该节点将会从等待队列中转移到同步队列中，加入到同步状态的获取中*/
    static final int CONDITION = -2;

    /**表示下一次共享式同步状态获取将会无条件地传播下去*/
    static final int PROPAGATE = -3;

    /** 等待状态 */
    volatile int waitStatus;

    /** 前驱节点 */
    volatile Node prev;

    /** 后继节点 */
    volatile Node next;

    /** 获取同步状态的线程 */
    volatile Thread thread;

    /** 等待队列中的后继节点*/
    Node nextWaiter;
	
    /**当前节点是否处于共享模式等待*/
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**获取前驱节点，如果为空的话抛出空指针异常*/
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {
    }

    /**addWaiter会调用此构造函数*/
    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    /**Condition会用到此构造函数*/
    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

### 2.6.2 ConditionObject

​		ConditonObject实现了Conditon，提供了在使用lock锁过程中的同步等待条件（类似于wait和nofity对于sychronized），及等待队列。

​		在ConditionObject中，通过一个等待队列来维护条线等待的线程。而在同一个AQS同步器中，可以定义多个Condition，只需要多次lock.newCondition()，每次都会返回一个新的ConditionObject对象。

​		在Condition中包含了firstWaiter和lastWaiter，每次加入到等待队列中的线程都会加入到等待队列的尾部，来构成一个FIFO的等待队列。

　　等待队列是一个FIFO的队列，在队列的每个节点都包含了一个线程引用。该线程就是在Condition对象上等待的线程。这里的节点和AQS中的同步队列中的节点一样，使用的都是AbstractQueuedSynchronizer.Node类。每个调用了condition.await()的线程都会进入到等待队列中去。

**详细解决可以参考如下文章:**

[ConditionObject分析](https://www.cnblogs.com/zerotomax/p/8969416.html)



## 2.7 AQS原理

### 2.7.1 CLH队列		

​		在我理解中AQS就是CLH队列（FIFO），队列中存放需要等待获取锁的线程。需要理解AQS获取锁的原理需要结合具体的锁实现来分析。

​		这里从最简单的独占非公平锁讲起，其他的原理在后面讲解每个锁或同步容器时顺便说明。以下是示例代码：

```java
        ReentrantLock lock = new ReentrantLock();
        Runnable runA =
                () -> {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    lock.lock();
                    System.out.println("线程"+Thread.currentThread().getName()+"开始");
                    try {
                        TimeUnit.SECONDS.sleep(30);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    lock.unlock();
                    System.out.println("线程"+Thread.currentThread().getName()+"结束");
                };
        Thread thread1 = new Thread(runA,"thread1");
        Thread thread2 = new Thread(runA,"thread2");
        thread1.start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread2.start();
```

（1）假设thread1先开始运行，调用`ReentrantLock.lock()`方法，默认`ReentrantLock`初始化时，默认是非公平锁。

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

那这里调用的实际就是`NonfairSync().lock`，其中`NonfairSync`是继承于Sync的内部类，而Sync由是继承于AQS （`abstract static class Sync extends AbstractQueuedSynchronizer`），具体看下lock方法的具体实现：

```java
final void lock() {
    if (compareAndSetState(0, 1))
    	setExclusiveOwnerThread(Thread.currentThread());
    else
    	acquire(1);
}
```

（2）调用AQS的`compareAndSetState`方法，这里就是一个简单的CAS，将AQS的state从0更新为1（0表示锁未被使用，1表示锁呗使用次数，）

```java
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```
（3）`compareAndSetState`返回true，接着调用`setExclusiveOwnerThread`设置独占锁的线程为thread1。到此lock方法执行完成，thread1打印“线程开始”，执行所属任务。

（4）此时，thread2开始运行，其执行到lock方法。调用`compareAndSetState`更新state值，但是此时state也已经是1了，无法更新返回false，方法执行else判断 `acquire(1)`。

**（5）** `acquire(1)`是AQS的重点方法，其源码如下：

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

* 首先调用 `tryAcquire(arg)`尝试去获得独占锁。`tryAcquire(arg)`由ReentranLock中Sync实现：

```java
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }

        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            // 0 标识锁未被占用
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // state++ 标识锁是可重入的
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

由获得锁的是thread1，故而此方法的两个if判断都无法执行，直接返回false。

* `addWaiter(Node.EXCLUSIVE)` ，将thread2封装为Node，并标记为EXCLUSIVE（等待独占锁），然后将Node节点加入到CLH等待队列中。

```java
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // 快速尝试添加尾节点
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            //CAS设置尾节点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 先通过快速尝试设置尾节点，如果失败，则调用enq(Node node)方法设置尾节点
        enq(node);
        return node;
    }

// 通过循环+CAS在队列中成功插入一个节点后返回
private Node enq(final Node node) {
	//多次尝试，直到成功为止
	for (;;) {
		Node t = tail;
		// 初始化head和tail 指向一个空节点
		if (t == null) {
			if (compareAndSetHead(new Node()))
				tail = head;
		} else {
			//设置为尾节点
			/*
 			 * AQS的精妙就是体现在很多细节的代码，比如需要用CAS往队尾里增加一个元素
 			 * 此处的else分支是先在CAS的if前设置node.prev = t，而不是在CAS成功之后再设置。
 			 * 一方面是基于CAS的双向链表插入目前没有完美的解决方案，另一方面这样子做的好处是：
 			 * 保证每时每刻tail.prev都不会是一个null值，否则如果node.prev = t
 			 * 放在下面if的里面，会导致一个瞬间tail.prev = null，这样会使得队列不完整。
 			 */
			node.prev = t;
			if (compareAndSetTail(t, node)) {
				t.next = node;
				return t;
			}
		}
	}
}
```

​		CLH队列入列是再简单不过了，无非就是tail指向新节点、新节点的prev指向当前最后的节点，当前最后一个节点的next指向当前节点。具体入队的示意图如下：

![thread-023](..\images\thread-023.png)

* `acquireQueued()`，通过tryAcquire()和addWaiter()，thread2获取资源失败，已经被放入等待队列尾部了。接下来就是进入等待状态休息，直到thread1彻底释放资源后唤醒thread2，thread2拿到资源，然后就可以执行任务了。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        //标记等待过程中是否被中断过
        boolean interrupted = false;
        //又是一个“自旋”
        for (;;) {
            //拿到前驱 （这里是哨兵节点）
            final Node p = node.predecessor();
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                //拿到资源后，将head指向该结点。所以head所指的哨兵节点，就是当前获取到资源的那个结点或null。
                setHead(node);
                // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                p.next = null; 
                // 成功获取资源
                failed = false; 
                //返回等待过程中是否被中断过
                return interrupted;
            }

            //如果自己可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                //如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
                interrupted = true;
        }
    } finally {
        // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
        if (failed) 
            cancelAcquire(node);
    }
}
```

​		这里假设thread1还未执行完成，thread2再次获取锁失败，则调用park进行waiting状态。

* `shouldParkAfterFailedAcquire` ，首先拿到前驱节点，也就是head（哨兵节点）的状态，先执行第一次循环，更新哨兵节点中waitStatus为Node.SIGNAL。接着就是第二次循环，前驱的等待状态为Node.SIGNAL，方法返回true。

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //拿到前驱的状态
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
         //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

* `parkAndCheckInterrupt`  thread2调用park阻塞自己，然后等待唤醒或中断。

```java
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

* `cancelAcquire`如果等待超时，或者被中断，导致没有成功获得自由，则将thread2移除出CLH队列。

（6）此时，thread2陷入lock的等待队列中，等待thread1完成后唤醒。唤醒操作是通过在thread1中调用unlock完成。

```java
    public void unlock() {
        sync.release(1);
    }
```

这里的唤醒操作，实际由AQS中的`release`完成。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        // head节点状态不会是CANCELLED，所以这里h.waitStatus != 0相当于h.waitStatus < 0
        if (h != null && h.waitStatus != 0)
            // 唤醒后继线程，此函数在acquire中已经分析过，不再列举说明
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

而`release`又由ReentranLock中的`tryRelease`来完成，释放锁工作。

```java
        protected final boolean tryRelease(int releases) {
            // state-1，如果是重入多次的，则挨个释放
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 状态为0表示锁已释放完成，清除独占锁当前线程
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

最后，如果释放锁成功，则调用`unparkSuccessor`唤醒后续线程。

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    // 尝试将node的等待状态置为0,这样的话,后继争用线程可以有机会再尝试获取一次锁。
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    /*
     * 这里的逻辑就是如果node.next存在并且状态不为取消，则直接唤醒s即可
     * 否则需要从tail开始向前找到node之后最近的非取消节点。
     *
     * 这里为什么要从tail开始向前查找也是值得琢磨的:
     * 如果读到s == null，不代表node就为tail，参考addWaiter以及enq函数中的我的注释。
     * 不妨考虑到如下场景：
     * 1. node某时刻为tail
     * 2. 有新线程通过addWaiter中的if分支或者enq方法添加自己
     * 3. compareAndSetTail成功
     * 4. 此时这里的Node s = node.next读出来s == null，但事实上node已经不是tail，它有后继了!
     */
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

### 2.7.2 Condition条件变量

​		有关在锁的过程中使用条件变量时，是如何使用AQS中的Condit

# 3 ReentrantLock

​		ReentrantLock是一个可重入的互斥锁，又被称为“独占锁”。意思就是在同一个时间点只能被一个线程锁持有；而可重入的意思是，可以被单个线程多次获取。

​		ReentrantLock分为“**公平锁**”和“**非公平锁**”。它们的区别体现在获取锁的机制上是否公平。“锁”是为了保护竞争资源，防止多个线程同时操作线程而出错，ReentrantLock在同一个时间点只能被一个线程获取(当某线程获取到“锁”时，其它线程就必须等待)；ReentraantLock是通过一个FIFO的等待队列（CLH）来管理获取该锁所有线程的。

​		在“公平锁”的机制下，线程依次排队获取锁；而“非公平锁”在锁是可获取状态时，不管自己是不是在队列的开头都会获取锁。

## 3.1 常用API

```java
// 创建一个 ReentrantLock ，默认是“非公平锁”。
public ReentrantLock()
// 创建策略是fair的 ReentrantLock。fair为true表示是公平锁，fair为false表示是非公平锁。
public ReentrantLock(boolean fair)

// 查询当前线程保持此锁的次数。
int getHoldCount()
// 返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正等待获取此锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正等待获取此锁的线程估计数。
int getQueueLength()
// 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回等待与此锁相关的给定条件的线程估计数。
int getWaitQueueLength(Condition condition)
// 查询给定线程是否正在等待获取此锁。
boolean hasQueuedThread(Thread thread)
// 查询是否有些线程正在等待获取此锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与此锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果是“公平锁”返回true，否则返回false。
boolean isFair()
// 查询当前线程是否保持此锁。
boolean isHeldByCurrentThread()
// 查询此锁是否由任意线程保持。
boolean isLocked()
// 获取锁。
void lock()
// 如果当前线程未被中断，则获取锁。
void lockInterruptibly()
// 返回用来与此 Lock 实例一起使用的 Condition 实例。
Condition newCondition()
// 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
boolean tryLock()
// 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。
boolean tryLock(long timeout, TimeUnit unit)
// 试图释放此锁。
void unlock()
```

## 3.2 示例1

```java
    /**
     * 演示可中断的非公平锁用法
     */
    private static void testNormalUsed() throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();

        Runnable run =
            () -> {
              System.out.println("线程"+Thread.currentThread().getName()+"准备获得锁");
              // lock不响应中断
//              lock.lock();
                try {
                    // 更换lockInterruptibly 响应中断
                    lock.lockInterruptibly();
                    System.out.println("线程"+Thread.currentThread().getName()+"获得锁");
                    for(;;) {

                    }
                } catch (InterruptedException e) {
                    System.out.println("线程"+Thread.currentThread().getName()+"响应中断");
                } finally{
                    // 这里必须添加此判断，因为线程B被阻塞从而响应中断后，如果不加此判断的话
                    // 会提示错误java.lang.IllegalMonitorStateException，是因为线程B并未获得锁，而unlock的话就提示错误
                    if (lock.hasQueuedThread(Thread.currentThread())) {
                        lock.unlock();
                    }
                }

            };

        Thread threada = new Thread(run,"A");
        Thread threadb = new Thread(run,"B");

        threada.start();
        Thread.sleep(1000);
        threadb.start();
        Thread.sleep(1000);

        System.out.println("线程"+threadb+"中断前状态"+threadb.getState().name());
        threadb.interrupt();
        System.out.println("线程"+threadb+"中断后状态"+threadb.getState().name());
    }
```

## 3.3 示例2

演示使用Condition的情况，以生产者消费者模型为例

```java
    /**
     * 这种使用condition的生产者消费者模式，存在问题
     * 就是只能是仓库生产慢后，才能消费。消费的同时是无法生产的。
     * 消费者的情况同理
     */
    private static void testCondition() {
        ReentrantLock lock = new ReentrantLock();
        AtomicInteger num = new AtomicInteger();
        ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
        Condition con1 = lock.newCondition();

    Thread threadA =
        new Thread(
            () -> {
              while (true) {
                lock.lock();
                try {
                  if (num.get() > 10) {
                    con1.await();
                  }
                  String nums = new Random().nextInt() + "";
                  queue.offer(nums);
                  Thread.sleep(1000);
                  num.incrementAndGet();
                  System.out.println("线程"+Thread.currentThread().getName()+"生成"+nums);
                  con1.signal();
                } catch (InterruptedException e) {
                  e.printStackTrace();
                } finally {
                  lock.unlock();
                }
              }
            },
            "A");

        Thread threadB = new Thread(()->{
            while(true){
                lock.lock();
                try{
                    if (num.get()<1) {
                        con1.await();
                    }
                    String nums = queue.poll();
                    Thread.sleep(1000);
                    num.decrementAndGet();
                    System.out.println("线程"+Thread.currentThread().getName()+"消费"+nums);
                    con1.signal();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally{
                    lock.unlock();
                }
            }
        },"B");

        threadA.start();
        threadB.start();
    }
```

## 3.4 源码分析



## 3.5 注意事项







# 4 ReentrantLockReadWriteLock

​		ReadWriteLock，顾名思义，是读写锁。它维护了一对相关的锁 — — “读取锁”和“写入锁”，一个用于读取操作，另一个用于写入操作。

* “读取锁”用于只读操作，它是“共享锁”，能同时被多个线程获取。
* “写入锁”用于写入操作，它是“独占锁”，写入锁只能被一个线程锁获取。

​		**注意：不能同时存在读取锁和写入锁！**

​		ReadWriteLock是一个接口，提供了"获取读锁的readLock()函数" 和 "获取写锁的writeLock()函数"。

```java
// 返回用于读取操作的锁。
Lock readLock()
// 返回用于写入操作的锁。
Lock writeLock()
```

​		ReentrantReadWriteLock是它的实现类，包含：sync对象，读锁readerLock和写锁writerLock。

​		读锁ReadLock和写锁WriteLock都实现了Lock接口。读锁ReadLock和写锁WriteLock中也都分别包含了"Sync对象"，它们的Sync对象和ReentrantReadWriteLock的Sync对象 是一样的，就是通过sync，读锁和写锁实现了对同一个对象的访问。

​		和"ReentrantLock"一样，sync是Sync类型；而且，Sync也是一个继承于AQS的抽象类。Sync也包括"公平锁"FairSync和"非公平锁"NonfairSync。sync对象是"FairSync"和"NonfairSync"中的一个，默认是"NonfairSync"。

## 4.1 常用API

```java
// 创建一个新的 ReentrantReadWriteLock，默认是采用“非公平策略”。
ReentrantReadWriteLock()
// 创建一个新的 ReentrantReadWriteLock，fair是“公平策略”。fair为true，意味着公平策略；否则，意味着非公平策略。
ReentrantReadWriteLock(boolean fair)

// 返回当前拥有写入锁的线程，如果没有这样的线程，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正在等待获取读取锁的线程。
protected Collection<Thread> getQueuedReaderThreads()
// 返回一个 collection，它包含可能正在等待获取读取或写入锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回一个 collection，它包含可能正在等待获取写入锁的线程。
protected Collection<Thread> getQueuedWriterThreads()
// 返回等待获取读取或写入锁的线程估计数目。
int getQueueLength()
// 查询当前线程在此锁上保持的重入读取锁数量。
int getReadHoldCount()
// 查询为此锁保持的读取锁数量。
int getReadLockCount()
// 返回一个 collection，它包含可能正在等待与写入锁相关的给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回正等待与写入锁相关的给定条件的线程估计数目。
int getWaitQueueLength(Condition condition)
// 查询当前线程在此锁上保持的重入写入锁数量。
int getWriteHoldCount()
// 查询是否给定线程正在等待获取读取或写入锁。
boolean hasQueuedThread(Thread thread)
// 查询是否所有的线程正在等待获取读取或写入锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与写入锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果此锁将公平性设置为 ture，则返回 true。
boolean isFair()
// 查询是否某个线程保持了写入锁。
boolean isWriteLocked()
// 查询当前线程是否保持了写入锁。
boolean isWriteLockedByCurrentThread()
// 返回用于读取操作的锁。
ReentrantReadWriteLock.ReadLock readLock()
// 返回用于写入操作的锁。
ReentrantReadWriteLock.WriteLock writeLock()
```

## 4.2 示例1

```java
    /**
     * 演示读锁和写锁使用的是同一个CLH对象
     * 且读锁和写锁不能共存，但是多个读锁可以共存
     */
    private static void testRWOnSameCLH() throws InterruptedException {
        ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        Lock writeLock = lock.writeLock();
        Lock readLock = lock.readLock();

    Runnable runW =
        () -> {
          writeLock.lock();
          System.out.println("线程" + Thread.currentThread().getName() + "获得锁");
          try {
            Thread.sleep(1000);
            System.out.println("CLH队列的等待线程数:"+
                    lock.getQueueLength()+",读锁:"+lock.getReadLockCount()+",写锁:"+lock.getWriteHoldCount());
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
          writeLock.unlock();
          System.out.println("线程" + Thread.currentThread().getName() + "释放锁");
        };

        Runnable runR = ()->{
            readLock.lock();
            System.out.println("线程"+Thread.currentThread().getName()+"获得锁");
            try {
                Thread.sleep(1200);
                System.out.println("CLH队列的等待线程数:"+
                        lock.getQueueLength()+",读锁:"+lock.getReadLockCount()+",写锁:"+lock.getWriteHoldCount());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            readLock.unlock();
            System.out.println("线程"+Thread.currentThread().getName()+"释放锁");
        };

        new Thread(runW,"WA").start();
        Thread.sleep(100);
        new Thread(runR,"RA").start();
        Thread.sleep(100);
        new Thread(runW,"WB").start();
        Thread.sleep(100);
        new Thread(runR,"RB").start();
        new Thread(runR,"RC").start();

    }
```

## 4.3 源码分析



## 4.4 注意事项







# 5 StampLock





