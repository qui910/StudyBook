# 1 概述

​		为了解决造成线程不安全的共享变量的竞争问题，并使其操作变为原子性的。Java提供了一种内置的锁机制，即`synchronized`关键字。`synchronized`包含两个部分：一个是作为锁的对象引用，一个是作为锁的对象提供的相关“条件”。

​		每个Java对象都可以用做一个实现同步的锁，这些锁被称为**内置锁**（Intrinsic Lock）或**监视器锁**（Monitor Lock）。

​		Java的内置锁相当于一种互斥体（或**互斥锁**），意味着最多只有一个线程能持有这种锁。

# 2 原理

​		在java中，每一个对象有且仅有一个同步锁。这也意味着，同步锁是依赖于对象而存在。当我们调用某对象的synchronized方法时，就获取了该对象的同步锁。例如，synchronized(obj)就获取了“obj这个对象”的同步锁。

​		不同线程对同步锁的访问是互斥的。也就是说，某时间点，对象的同步锁只能被一个线程获取到！通过同步锁，我们就能在多线程中，实现对“对象/方法”的互斥访问。 

​		例如，现在有两个线程A和线程B，它们都会访问“对象obj的同步锁”。假设，在某一时刻，线程A获取到“obj的同步锁”并在执行一些操作；而此时，线程B也企图获取“obj的同步锁” —— 线程B会获取失败，线程B状态变更为阻塞状态，进入锁的阻塞队列，直到线程A释放了“该对象的同步锁”之后线程B才能获取到“obj的同步锁”。

​		synchronized是针对对象的隐式锁使用的，注意是**对象！**

# 3 基本规则

* 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的该“synchronized方法”或者“synchronized代码块”的访问将被阻塞。
* 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然可以访问“该对象”的非同步代码块。
* 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问将被阻塞。

# 4 基本特性

## 4.1 可重入性

​		从互斥锁的设计上来说，当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁，请求将会成功。

​		在java中synchronized是基于原子性的内部锁机制，是可重入的，因此在一个线程调用synchronized方法的同时在其方法体内部调用该对象另一个synchronized方法，也就是说一个线程得到一个对象锁后再次请求该对象锁，是允许的，这就是synchronized的可重入性

## 4.2 不能继承

​		synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法。

## 4.3 线程中断	

​		线程的中断操作对于正在等待获取的锁对象的synchronized方法或者代码块并不起作用，也就是对于synchronized来说，如果一个线程在等待锁，那么结果只有两种，要么它获得这把锁继续执行，要么它就保存等待，即使调用中断线程的方法，也不会生效。

​		即阻塞状态的线程无法响应中断。

代码示例：

```java
    /**
     * 测试阻塞状态是否响应中断
     */
    private static void testInterrupte() throws InterruptedException {
        Object obj = new Object();
        Thread threadA = new Thread(()->{
            synchronized (obj) {
                System.out.println("testInterrupte threadA has lock");
                while(true) {

                }
            }
        });
        threadA.start();
        TimeUnit.SECONDS.sleep(4);
        Thread threadB = new Thread(()->{
            synchronized (obj) {
                System.out.println("testInterrupte threadB has lock");
                while(true) {
                    
                }
            }
        });
        threadB.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("ThreadA 中断前状态："+threadA.getState().name());
        System.out.println("ThreadB 中断前状态："+threadB.getState().name());
        threadB.interrupt();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("ThreadB 中断后状态："+threadB.getState().name());
    }
}
```

运行结果：

```shell
testInterrupte threadA has lock
ThreadA 中断前状态：RUNNABLE
ThreadB 中断前状态：BLOCKED
ThreadB 中断后状态：BLOCKED
```



# 5 使用示例

## 5.1 实例锁

​		这种锁，一般是使用在实例的方法或块上添加`synchronized`关键字，只要某个线程访问了该实例的一个`synchronized`方法或块。那其他线程均不能访问该实例的其他`synchronized`方法或块，但可以访问非`synchronized`方法或块。

​		 注意：这里是指的同一个实例，而同一个类的不同实例不影响。块上使用`synchronized`关键字，要注意括号内必须传入当前的对象实例。

## 5.2 类锁

​		类锁，就是在类的静态方法，静态块上添加`synchronized`关键字，这是的内置锁是针对的类对象，而不是实例对象。即在访问类型的静态同步方法时，不影响其他线程访问对象实例的同步方法。

```java
/**
 * 同步锁测试
 */
public class SynchronizedTest {

    public static void main(String[] args) {
//        testObjectLockA();
        testOjectLockB();
    }

    /**
     * 测试实例锁
     * A,B,C 是同一把锁，所以每次只能执行一个
     * A 与 D 是两把锁，所以不影响，同时执行
     * A 与 E 是同步方法与非同步方法的关系，可以同时执行
     */
    private static void testObjectLockA() {
        SynchrondClass classA = new SynchrondClass();
        new Thread(()->classA.methodA()).start();
        new Thread(()->classA.methodB()).start();
        new Thread(()->classA.methodC()).start();
        new Thread(()->classA.methodD()).start();
        new Thread(()->classA.methodE()).start();
    }

    /**
     * 测试类锁
     * A 与 staticA 是两把不同的锁，可以同时执行
     * staticA 与 staticB 是同一把锁，不可以同时执行
     */
    private static void testOjectLockB() {
        SynchrondClass classA = new SynchrondClass();
        new Thread(()->classA.methodA()).start();
        new Thread(()->SynchrondClass.staticMethodA()).start();
        new Thread(()->SynchrondClass.staticMethodB()).start();
    }
}

class SynchrondClass {

    /**
     * 实例同步方法A
     */
    public synchronized void methodA() {
        for (int i=0;i<10;i++) {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized void methodA "+i);
        }
    }

    /**
     * 实例同步方法B
     */
    public synchronized void methodB() {
        for (int i=0;i<10;i++) {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized void methodB "+i);
        }
    }

    /**
     * 实例同步块C
     */
    public void methodC() {
        // 这里要想使块与方法A，B为同一实例，要使用this，表明是使用的当前对象的内置锁
        synchronized (this) {
            for (int i=0;i<10;i++) {
                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("synchronized void methodC "+i);
            }
        }
    }

    /**
     * 实例同步块D
     */
    public void methodD() {
        // 这里使用的时另外的对象实例，与A，B，C方法不是同一个锁，可以同时访问
        Object obj = new Object();
        synchronized (obj) {
            for (int i=0;i<10;i++) {
                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("synchronized void methodD "+i);
            }
        }
    }

    /**
     * 实例非同步块E
     */
    public void methodE() {
        for (int i=0;i<10;i++) {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized void methodE "+i);
        }
    }

    /**
     * 静态方法
     */
    public static synchronized void staticMethodA() {
        for (int i=0;i<10;i++) {
            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("synchronized void staticMethodA "+i);
        }
    }

    /**
     * 静态块
     */
    public static void staticMethodB() {
        synchronized (SynchrondClass.class) {
            for (int i=0;i<10;i++) {
                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("synchronized void staticMethodB "+i);
            }
        }
    }
}
```

# 6 条件对象

```java
// 解除那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在同步方法或同步块内调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalMonitorStateException异常
void notifyAll();
// 随机选择一个在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在一个同步方法或同步块中调用。如果当前线程不是对象锁的持有者，该方法抛出IllegalMonitorStateException异常
void notify();
// 导致线程进入等待状态直到它被通知，该方法只能在一个同步方法中调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalMonitorStateException异常
void wait();
// 导致线程进入等待状态直到它被通知或经过指定的时间。这些方法只能在一个同步方法中调用。如果当前线程不是对象锁的持有者，该方法抛出一个IllegalMonitorStateException异常
void wait(long millis);
// millis 毫秒数  nanos 纳秒数
void wait(long millis,int nanos);
```

​	内部锁只有一个条件对象，wait方法添加一个线程到等待集，notify/notifyAll方法解除等待线程的阻塞状态。

​		由此可以总结得出，每个对象有一个内部锁，并且该锁有一个内部条件。由锁来管理那些视图进入synchronized方法的线程，由条件来管理那些调用wait的线程。

* 有synchronized的地方不一定有wait,notify。
* 有wait,notify的地方必有synchronized.这是因为wait和notify不是属于线程类，而是每一个对象都具有的方法（事实上，这两个方法是Object类里的），而且，这两个方法都和对象锁有关，有锁的地方，必有synchronized。 
* 锁是针对对象的，wait()/notify()的操作是与对象锁相关的，那么把wait()/notify()设计在Object中也就是合情合理的了。 
* synchronized方法中由当前线程占有锁。另一方面，调用wait()notify()方法的对象上的锁必须为当前线程所拥有。因此，wait()notify()方法调用必须放置在synchronized方法中，synchronized方法的上锁对象就是调用wait()notify()方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。 

* 调用wait()方法前的判断最好用while，而不用if；因为while可以实现被唤醒后线程再次作条件判断；而if则只能判断一次 。
* 用notifyAll()优先于notify()。 能调用wait()/notify()的只有当前线程，前提是必须获得了对象锁，就是说必须要进入到synchronized方法中。





# 7 底层语义原理

​			Java 虚拟机中的同步(Synchronization)基于进入和退出管程(Monitor)对象实现， 无论是显式同步(有明确的 monitorenter 和 monitorexit 指令,即同步代码块)还是隐式同步都是如此。在 Java 语言中，同步用的最多的地方可能是被 synchronized 修饰的同步方法。同步方法 并不是由 monitorenter 和 monitorexit 指令来实现同步的，而是由方法调用指令读取运行时常量池中方法的 ACC_SYNCHRONIZED 标志来隐式实现的，关于这点，稍后详细分析。下面先来了解一个概念Java对象头，这对深入理解synchronized实现原理非常关键。

（这块内容暂时不理解，稍后在做补充）

https://www.cnblogs.com/nevermorewang/p/9864797.html

