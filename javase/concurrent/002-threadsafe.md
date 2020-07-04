# 1 线程安全

​		当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称为这个类是线程安全的。

​		线程安全的问题主要针对的就是共享变量，即线程安全问题可以理解为当多个线程同时读写一个共享变量并且没有任何同步措施时，导致出现脏数据或者其他不可见的结果。所以，无状态对象（即无共享变量）一定是线程安全的。

## 1.1 内存可见性

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

## 1.2 失效数据

​		当一个线程读取到一个状态的时候，这个状态可能是失效了的状态。即就是该状态的修改与读取没有同步的问题。

* 线程可能会读取两次状态但是，一次获得的是正确的，另一次获得是错误的。比如如下代码，num是共享数据，num正在被其它某个线程由3修改为5了，而同时另一个线程在执行如下语句时可能第一次读到的num是3（失效的），而第二次读num时就是5了，最终num值变成了8，一个完全错误的值。

```java
num=num+num;
```

* 比如对象的引用失效了，而线程却正好获取到了这个失效了的引用，就有可能导致报出异常或者更严重的安全问题。

## 1.3 非原子的64位操作

​		线程在没有同步的情况下读取了共享数据，可能会得到失效值，但这个值至少是某个线程设置的值，而不是一个随机我们就称这是最低的安全性（out-of-thin-airsafety）。

​		对于绝大多数变量，java内存模型是符合最低的安全性的，但是有个例外。对于非volatile类型的64位数值变量（double和long），jvm允许在读取操作或写入操作时分解为两个32位的操作指令，也就是先操作一个32位的内存，再操作另外一个32位的内存。这就有很可能在读取时，读取某个值的高32位和另一个值的低32位组成一个完全错误的数据。

## 1.4 volatile

​		volatile关键字具有有序性和可见性，不具备原子性。它也没有锁机制，所以是相对于synchronized关键字更轻量级的同步机制。相对的不具备原子性说明它不是一个完整的同步方式，volatile的同步具有一定局限性。

## 1.5 发布与逸出

- 发布(Publish)

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

- 逸出（Escape）

> 发布了本不该发布的对象，或错误的发布

​		逸出也是发布，区别在于：**发布**在于你想发布，**逸出**在于你不想发布的对象却也被发布了。或者在时间维度上说，在某一时刻你还没打算发布的对象也被发布出去，这也算逸出。
发布和逸出怎么分辨？这也是基于正确性2。

​		发布了本不该发布的对象，一般都是不正确的间接发布导致的。

### 1.5.1 示例1

```java
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

​		此例中，封装一个集合的目的就是要安全的访问这个集合。但是`get()`方法却意外的将`infos`的引用暴露给了外部环境，这样对于集合的安全访问就不受控制了。

### 1.5.2 示例2

```java
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

​		此例是在构造函数中，创建了一个监听线程，但是无形之中`EventListener`将外部类`EscapeDemo`的引用通过this给暴露出去了。这样可能导致外部系统访问到一个未构建好的`EscapeDemo`对象，导致不正确的构造溢出。

​		正确的做法：不要在构造函数中立即启动监听线程，而是先保存这个对象的引用，等构造完成后再通过其它方式启动它。如下，通过工厂模式，私有化构造函数，通过newInstance方法发布EscapeDemo 对象。

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

​		另外并不是只有线程放置在构造函数中会导致逸出，如果在构造函数中调用一个可改写的实例方法（不是终结方法，也不是私有方法），同样也会导致this引用在构造过程中逸出。



## 1.6 线程封闭

> 为了避免同步问题，最简单的方法就是不共享数据，让变量只在一个线程内可见使用，或者同一时段只有一个线程对该变量可见处理。这种技术就称为线程封闭。

​		最常见的线程封闭的例子就是局部变量。线程封闭技术是实现线程安全最简单的方式之一。实现线程封闭的几种方法：

* Ad-hoc线程封闭
  就是完全靠实现者代码控制的线程封闭。Ad-hoc线程封闭非常脆弱，没有任何一种语言特性能将对象封闭到目标线程上。

* 栈封闭
  就是前面所说的局部变量，局部变量的固有属性之一就是封闭在线程之内。局部变量的引用值是存储在栈内的局部变量表之中，其它线程无法访问到。

* ThreadLocal封闭




## 1.7 不变性

​		满足同步需求的另一种方法是不可变对象（Immutable Object）。由于对象的状态不可变了，那么共享的时候也不会出现之前所说的问题。不可变对象很简单，它只有一种状态，并且是在构造函数时确定的。

> 不可变对象一定是线程安全的

​		在java语言规范与java内存模型中没有给出不可变性的正式定义，书中给出了如下三个条件，满足这三个条件的对象才是不可变对象：

- 对象创建以后其状态就不能修改
- 对象的所有域都是final类型
- 对象是正确创建的（在创建对象期间，this引用没有逸出）

以上部分内容参考：

[JAVA并发编程之对象的共享](https://blog.csdn.net/u014296316/article/details/86320855)



#  2 代码示例

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

