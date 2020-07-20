# 1 概述

线程池架构图如下：![thread-024](..\..\images\thread-024.png)

## 1.1 Executor

它是"执行者"接口，它是来执行任务的。准确的说，Executor提供了execute()接口来执行已提交的 Runnable 任务的对象。Executor存在的目的是提供一种将"任务提交"与"任务如何运行"分离开来的机制。
它只包含一个函数接口：

```java
void execute(Runnable command)
```

##  1.2 **ExecutorService**

ExecutorService继承于Executor。它是"执行者服务"接口，它是为"执行者接口Executor"服务而存在的；准确的话，ExecutorService提供了"将任务提交给执行者的接口(submit方法)"，"让执行者执行任务(invokeAll, invokeAny方法)"的接口等等。

```java
// 请求关闭、发生超时或者当前线程中断，无论哪一个首先发生之后，都将导致阻塞，直到所有任务完成执行。
boolean awaitTermination(long timeout, TimeUnit unit)
    
// 执行给定的任务，当所有任务完成时，返回保持任务状态和结果的 Future 列表。
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    
// 执行给定的任务，当所有任务完成或超时期满时（无论哪个首先发生），返回保持任务状态和结果的 Future 列表。
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
    
// 执行给定的任务，如果某个任务已成功完成（也就是未抛出异常），则返回其结果。
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
    
// 执行给定的任务，如果在给定的超时期满前某个任务已成功完成（也就是未抛出异常），则返回其结果。
<T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
    
// 如果此执行程序已关闭，则返回 true。
boolean isShutdown()
    
// 如果关闭后所有任务都已完成，则返回 true。
boolean isTerminated()
    
// 启动一次顺序关闭，执行以前提交的任务，但不接受新任务。
void shutdown()
// 试图停止所有正在执行的活动任务，暂停处理正在等待的任务，并返回等待执行的任务列表。
List<Runnable> shutdownNow()
    
// 提交一个返回值的任务用于执行，返回一个表示任务的未决结果的 Future。
<T> Future<T> submit(Callable<T> task)
    
// 提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。
Future<?> submit(Runnable task)
    
// 提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。
<T> Future<T> submit(Runnable task, T result)
```

##  1.3 **AbstractExecutorService**

AbstractExecutorService是一个抽象类，它实现了ExecutorService接口。
AbstractExecutorService存在的目的是为ExecutorService中的函数接口提供了默认实现。

**AbstractExecutorService函数列表**由于它的函数列表和ExecutorService一样，这里就不再重复列举了。



## 1.4 **ThreadPoolExecutor**

ThreadPoolExecutor就是大名鼎鼎的"线程池"。它继承于AbstractExecutorService抽象类。



## 1.5 **ScheduledExecutorService**

ScheduledExecutorService是一个接口，它继承于于ExecutorService。它相当于提供了"延时"和"周期执行"功能的ExecutorService。

ScheduledExecutorService提供了相应的函数接口，可以安排任务在给定的延迟后执行，也可以让任务周期的执行。

```java
// 创建并执行在给定延迟后启用的 ScheduledFuture。
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)
// 创建并执行在给定延迟后启用的一次性操作。
ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit)
// 创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在 initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。
ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)
// 创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟。
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)
```

## 1.6 **ScheduledThreadPoolExecutor**

ScheduledThreadPoolExecutor继承于ThreadPoolExecutor，并且实现了ScheduledExecutorService接口。它相当于提供了"延时"和"周期执行"功能的ScheduledExecutorService。

ScheduledThreadPoolExecutor类似于Timer，但是在高并发程序中，ScheduledThreadPoolExecutor的性能要优于Timer。

## 1.7 **Executors**

Executors是个静态工厂类。它通过静态工厂方法返回ExecutorService、ScheduledExecutorService、ThreadFactory 和 Callable 等类的对象。

```java
// 返回 Callable 对象，调用它时可运行给定特权的操作并返回其结果。
static Callable<Object> callable(PrivilegedAction<?> action)
// 返回 Callable 对象，调用它时可运行给定特权的异常操作并返回其结果。
static Callable<Object> callable(PrivilegedExceptionAction<?> action)
// 返回 Callable 对象，调用它时可运行给定的任务并返回 null。
static Callable<Object> callable(Runnable task)
// 返回 Callable 对象，调用它时可运行给定的任务并返回给定的结果。
static <T> Callable<T> callable(Runnable task, T result)
// 返回用于创建新线程的默认线程工厂。
static ThreadFactory defaultThreadFactory()
// 创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。
static ExecutorService newCachedThreadPool()
// 创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们，并在需要时使用提供的 ThreadFactory 创建新线程。
static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
// 创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。
static ExecutorService newFixedThreadPool(int nThreads)
// 创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程，在需要时使用提供的 ThreadFactory 创建新线程。
static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
// 创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
// 创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
// 创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。
static ExecutorService newSingleThreadExecutor()
// 创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程，并在需要时使用提供的 ThreadFactory 创建新线程。
static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
// 创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。
static ScheduledExecutorService newSingleThreadScheduledExecutor()
// 创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。
static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
// 返回 Callable 对象，调用它时可在当前的访问控制上下文中执行给定的 callable 对象。
static <T> Callable<T> privilegedCallable(Callable<T> callable)
// 返回 Callable 对象，调用它时可在当前的访问控制上下文中，使用当前上下文类加载器作为上下文类加载器来执行给定的 callable 对象。
static <T> Callable<T> privilegedCallableUsingCurrentClassLoader(Callable<T> callable)
// 返回用于创建新线程的线程工厂，这些新线程与当前线程具有相同的权限。
static ThreadFactory privilegedThreadFactory()
// 返回一个将所有已定义的 ExecutorService 方法委托给指定执行程序的对象，但是使用强制转换可能无法访问其他方法。
static ExecutorService unconfigurableExecutorService(ExecutorService executor)
// 返回一个将所有已定义的 ExecutorService 方法委托给指定执行程序的对象，但是使用强制转换可能无法访问其他方法。
static ScheduledExecutorService unconfigurableScheduledExecutorService(ScheduledExecutorService executor)
```

# 2 Callable & Future

## 2.1 Callable

Callable 是一个接口，它只包含一个call()方法。Callable是一个返回结果并且可能抛出异常的任务。

可以将Callable比作一个Runnable接口，而Callable的call()方法则类似于Runnable的run()方法。

## 2.2 Future

Future 是一个接口。它用于表示异步计算的结果。提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。

```java
public interface Future<V> {
// 试图取消对此任务的执行。
boolean  cancel(boolean mayInterruptIfRunning)

// 如果在任务正常完成前将其取消，则返回 true。
boolean  isCancelled()

// 如果任务已完成，则返回 true。
boolean     isDone()

// 如有必要，等待计算完成，然后获取其结果。
V  get() throws InterruptedException, ExecutionException;

// 如有必要，最多等待为使计算完成所给定的时间之后，获取其结果（如果结果可用）。
V  get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

## 2.3 FutureTask

FutureTask是Future的计算结果。Callable, Future, FutureTask它们之间的关系图：

![thread-025](..\..\images\thread-025.png)

* 在通过AbstractExecutorService.submit提交任务后，会将Callable封装为FutureTask。

```java
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    // 创建一个RunnableFuture对象
    RunnableFuture<T> ftask = newTaskFor(task);
    // 执行“任务ftask”
    execute(ftask);
    // 返回“ftask”
    return ftask;
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
```

在newTaskFor()新建一个ftask对象之后，会通过execute(ftask)执行该任务。此时ftask被当作一个Runnable对象进行执行，最终会调用到它的run()方法；ftask的run()方法:

```java
// run()中会执行Callable对象的call()方法，并且最终将结果保存到result中，并通过set(result)将result保存。之后调用FutureTask的get()方法，返回的就是通过set(result)保存的值。
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        // 将callable对象赋值给c。
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 执行Callable的call()方法，并保存结果到result中。
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            // 如果运行成功，则将result保存
            if (ran)
                set(result);
        }
    } finally {
        runner = null;
        // 设置“state状态标记”
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

* 在通过ExecutorService.execute提交Runable任务后，会将Runable任务直接提交给线程池执行。

# 3 ThreadPoolExecutor

ThreadPoolExecutor是线程池类。对于线程池，可以通俗的将它理解为"存放一定数量线程的一个线程集合。线程池允许若个线程同时允许，允许同时运行的线程数量就是线程池的容量；当添加的到线程池中的线程超过它的容量时，会有一部分线程阻塞等待。线程池会通过相应的调度策略和拒绝策略，对添加到线程池中的线程进行管理。"

其数据结构图如下：

![thread-026](..\..\images\thread-026.png)

## 3.1 成员变量

```java
// 阻塞队列。
private final BlockingQueue<Runnable> workQueue;
// 互斥锁
private final ReentrantLock mainLock = new ReentrantLock();
// 线程集合。一个Worker对应一个线程。
private final HashSet<Worker> workers = new HashSet<Worker>();
// “终止条件”，与“mainLock”绑定。
private final Condition termination = mainLock.newCondition();
// 线程池中线程数量曾经达到过的最大值。
private int largestPoolSize;
// 已完成任务数量
private long completedTaskCount;
// ThreadFactory对象，用于创建线程。
private volatile ThreadFactory threadFactory;
// 拒绝策略的处理句柄。
private volatile RejectedExecutionHandler handler;
// 保持线程存活时间。
private volatile long keepAliveTime;
// 核心线程是否超时 默认true
private volatile boolean allowCoreThreadTimeOut;
// 核心池大小
private volatile int corePoolSize;
// 最大池大小
private volatile int maximumPoolSize;
```

**1. workers**

workers是HashSet<Work>类型，即它是一个Worker集合。而一个Worker对应一个线程，也就是说线程池通过workers包含了"一个线程集合"。当Worker对应的线程池启动时，它会执行线程池中的任务；当执行完一个任务后，它会从线程池的阻塞队列中取出一个阻塞的任务来继续运行。

wokers的作用是，线程池通过它实现了"允许多个线程同时运行"。

**2. workQueue**

workQueue是BlockingQueue类型，即它是一个阻塞队列。当线程池中的线程数超过它的容量的时候，线程会进入阻塞队列进行阻塞等待。

通过workQueue，线程池实现了阻塞功能。

**3. mainLock**
  mainLock是互斥锁，通过mainLock实现了对线程池的互斥访问。

**4. corePoolSize和maximumPoolSize**

corePoolSize是"核心池大小"，maximumPoolSize是"最大池大小"。它们的作用是调整"线程池中实际运行的线程的数量"。

例如，当新任务提交给线程池时(通过execute方法)。

* 如果此时，线程池中运行的线程数量< corePoolSize，则创建新线程来处理请求。
* 如果此时，线程池中运行的线程数量> corePoolSize，但是却< maximumPoolSize；则仅当阻塞队列满时才创建新线程。
* 如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。
* 如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。在大多数情况下，核心池大小和最大池大小的值是在创建线程池设置的；但是，也可以使用 setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。

**5. poolSize**

poolSize是当前线程池的实际大小，即线程池中任务的数量。

**6. allowCoreThreadTimeOut和keepAliveTime**

allowCoreThreadTimeOut表示是否允许"线程在空闲状态时，仍然能够存活"；而keepAliveTime是当线程池处于空闲状态的时候，超过keepAliveTime时间之后，空闲的线程会被终止。

**7. threadFactory**

threadFactory是ThreadFactory对象。它是一个线程工厂类，"线程池通过ThreadFactory创建线程"。

**8. handler**

handler是RejectedExecutionHandler类型。它是"线程池拒绝策略"的句柄，也就是说"当某任务添加到线程池中，而线程池拒绝该任务时，线程池会通过handler进行相应的处理"。