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

## 4.2 相关条件

​		



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

