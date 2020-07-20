# 1 概述

​		CountDownLatch是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。其本质也是一个共享锁。

 		**CountDownLatch和CyclicBarrier的区别**

* CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。
* CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。



# 2 常用API

```java
public CountDownLatch(int count)
构造一个用给定计数初始化的 CountDownLatch。

// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。
void await()
// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。
boolean await(long timeout, TimeUnit unit)
// 递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
void countDown()
// 返回当前计数。
long getCount()
// 返回标识此锁存器及其状态的字符串。
String toString()
```

# 3 源码解析

​		CountDownLatch是通过“共享锁”实现的。在创建CountDownLatch中时，会传递一个int类型参数count，该参数是“锁计数器”的初始状态，表示该“共享锁”最多能被count给线程同时获取。当某线程调用该CountDownLatch对象的await()方法时，该线程会等待“共享锁”可用时，才能获取“共享锁”进而继续运行。而“共享锁”可用的条件，就是“锁计数器”的值为0！而“锁计数器”的初始值为count，每当一个线程调用该CountDownLatch对象的countDown()方法时，才将“锁计数器”-1；通过这种方式，必须有count个线程调用countDown()之后，“锁计数器”才为0，而前面提到的等待线程才能继续运行！



# 4 示例

## 4.1 一般用法

```java
    /**
     * 测试CountDownLatch一般用法
     */
    private static void testNormalUsed() throws InterruptedException {
        CountDownLatch count = new CountDownLatch(2);
        System.out.println("主线程开始");
        new Thread(
                () -> {
                  System.out.println("线程"+Thread.currentThread().getName()+"开始执行");
                    try {
                        Thread.sleep(1000);
                        System.out.println("线程"+Thread.currentThread().getName()+"执行完成");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally{
                        count.countDown();
                    }
                },"A")
            .start();
        new Thread(
                () -> {
                    System.out.println("线程"+Thread.currentThread().getName()+"开始执行");
                    try {
                        Thread.sleep(2000);
                        System.out.println("线程"+Thread.currentThread().getName()+"执行完成");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally{
                        count.countDown();
                    }
                },"B")
                .start();
        count.await();
        System.out.println("主线程执行完成");
    }
```

结果:

```shell
主线程开始
线程A开始执行
线程B开始执行
线程A执行完成
线程B执行完成
主线程执行完成
```

