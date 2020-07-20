# 1 概述

在Java API中定义，可以从其中读取一个字节序列的对象成为输入流，而可以向其中写入一个字节序列的对象成为输出流。这些流可以操作文件，内存块或网络连接等，抽象类`InputStream`和`OutoutStream`构成了输入/输出(I/O)类层次结构的基础。

同时因为面向字节的流不编译处理以Unicode存储的信息，所以从抽象类`Reader`和`Writer`中继承出了一个专门用于处理Unicode字符的单独的层次结构。这些类拥有的读写操作都是基于两字节的Unicode码元。故Java的IO操作类有可以分为以下几种类型：

* 基于字节操作的IO接口：InputStream和OutputStearm
* 基于字符操作的IO接口：Writer和Reader
* 基于磁盘操作的IO接口：File
* 基于网络操作的IO接口：Socket

以上这些IO操作，即使在没有数据时`read()`也会造成阻塞，称为BIO（同步阻塞）。而BIO的效率不高，在这个数据大爆炸的时代容易称为瓶颈，如此Java1.4时推出了NIO（同步非阻塞）。Java7时又推出了更好的优化 NIO2（异步非阻塞），又称AIO。

## 1.1 传统IO体系

这里我们讲究的主要以上前3种IO操作（`java.io`包下），有关网络的IO（`java.net`包下）另外单独章节进行介绍。

从数据来源或者说是操作对象角度看，IO 类可以分为：

1、文件（file）：FileInputStream、FileOutputStream、FileReader、FileWriter

2、数组（[]）：

- 2.1、字节数组（byte[]）：ByteArrayInputStream、ByteArrayOutputStream
- 2.2、字符数组（char[]）：CharArrayReader、CharArrayWriter

3、管道操作：PipedInputStream、PipedOutputStream、PipedReader、PipedWriter

4、基本数据类型：DataInputStream、DataOutputStream

5、缓冲操作：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter

6、打印：PrintStream、PrintWriter

7、对象序列化反序列化：ObjectInputStream、ObjectOutputStream

8、转换：InputStreamReader、OutputStreWriter

9、字符串（String）：~~StringBufferInputStream、StringBufferOutputStream、StringReader、StringWriter~~（**Java8中已废弃**）

按照字节流和字符流可以划分为：

![io-001](..\images\io-001.png)

## 1.2 字节流

下图字节流的体系图

![io-002](..\images\io-002.png)



## 1.3 字符流

![io-003](..\images\io-003.png)



# 2 抽象类

IO 类虽然很多，但最基本的是 4 个抽象类：InputStream、OutputStream、Reader、Writer。最基本的方法也就是一个读 read() 方法、一个写 write() 方法。

## 2.1 InputStream

### 2.1.1 常用API

```java
// 读取数据,int返回值范围（0-255），如果已经到达流的末尾将返回-1。 此方法将阻塞，直到输入数据可用，检测到流的末尾或者抛出异常。
public abstract int read()	
    
// 将读取到的数据放在 byte 数组中，该方法实际上是根据下面的方法实现的，off 为 0，len 为数组的长度。如果b的长度是零，则返回0;如果流是在文件的结尾，则返回-1。此方法将阻塞，直到输入数据可用，检测到流的末尾或者抛出异常。
public int read(byte b[])	
    
// 从第 off 位置读取 len 长度字节的数据放到 byte 数组中，流是以 -1 来判断是否读取结束的（注意这里读取的虽然是一个字节，但是返回的却是 int 类型 4 个字节，这里当然是有原因，这里就不再细说了，推荐这篇文章，链接(https://blog.csdn.net/congwiny/article/details/18922847)）
public int read(byte b[], int off, int len)	

// 跳过指定个数的字节不读取，想想看电影跳过片头片尾
public long skip(long n)
    
// 返回可读的字节数量
public int available()	

// 读取完，关闭流，释放资源
public void close()	

// 标记读取位置，下次还可以从这里开始读取，使用前要看当前流是否支持，可以使用 markSupport() 方法判断
public synchronized void mark(int readlimit)
    
// 重置读取位置为上次 mark 标记的位置    
public synchronized void reset()
    
// 判断当前流是否支持标记流，和上面两个方法配套使用    
public boolean markSupported()	
```

### 2.1.2 示例

测试在`read()`读取数据时，如果未准备好，是否会造成阻塞（注意这里的阻塞是指当前线程停止在`read()`方法处，而非指当前线程状态是阻塞）。

```java
    private static void testBlocking() throws InterruptedException, IOException {
        Thread test = new Thread(()->{
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("请输入一个字符，按 q 键结束");
            char c=0;
            do {
                try {
                    c = (char) bufferedReader.read();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                System.out.println("你输入的字符为"+c);
            } while (c != 'q');
        });
        test.start();
        Thread.sleep(1000);
        System.out.println("在未输入时，线程test的状态" + test.getState().name());
    }
```

运行结果：

```java
请输入一个字符，按 q 键结束
在未输入时，线程test的状态RUNNABLE
```

### 2.1.3 疑问

这里大家会有一个疑问，为何`read`和`write`明明都是操作的字节，为何参数都是int呢？具体解释，大家可以参考下这个博文：[java基础--Java 字节读取流的read方法返回int的原因](https://blog.csdn.net/congwiny/article/details/18922847)



## 2.2 **OutputStream**

### 2.2.1 常用API

```java
// 写入一个字节，可以看到这里的参数是一个 int 类型，对应上面的读方法，int 类型的 32 位，只有低 8 位才写入，高 24 位将舍弃。
public abstract void write(int b)	
// 	将数组中的所有字节写入，和上面对应的 read() 方法类似，实际调用的也是下面的方法。
public void write(byte b[])
// 将 byte 数组从 off 位置开始，len 长度的字节写入
public void write(byte b[], int off, int len)	
// 强制刷新，将缓冲中的数据写入
public void flush()	
// 关闭输出流，流被关闭后就不能再输出数据了    
public void close()	
```



## 2.3 **Reader**

### 2.3.1 常用API

```java
// 读取字节到字符缓存中
public int read(java.nio.CharBuffer target)	
// 读取单个字符    
public int read()
// 读取字符到指定的 char 数组中    
public int read(char cbuf[])
// 从 off 位置读取 len 长度的字符到 char 数组中    
abstract public int read(char cbuf[], int off, int len)	
// 跳过指定长度的字符数量    
public long skip(long n)
// 和上面的 available() 方法类似    
public boolean ready()
// 判断当前流是否支持标记流    
public boolean markSupported()	
// 标记读取位置，下次还可以从这里开始读取，使用前要看当前流是否支持，可以使用 markSupport() 方法判断    
public void mark(int readAheadLimit)
// 重置读取位置为上次 mark 标记的位置    
public void reset()	
// 关闭流释放相关资源
abstract public void close()	
```



## 2.4 **Writer**

### 2.4.1 常用API

```java
// 写入一个字符
public void write(int c)
// 写入一个字符数组    
public void write(char cbuf[])	
// 从字符数组的 off 位置写入 len 数量的字符    
abstract public void write(char cbuf[], int off, int len)	
// 写入一个字符串    
public void write(String str)
// 从字符串的 off 位置写入 len 数量的字符    
public void write(String str, int off, int len)
// 追加吸入一个字符序列    
public Writer append(CharSequence csq)	
// 追加写入一个字符序列的一部分，从 start 位置开始，end 位置结束    
public Writer append(CharSequence csq, int start, int end)
// 追加写入一个 16 位的字符    
public Writer append(char c)
// 强制刷新，将缓冲中的数据写入    
abstract public void flush()
// 关闭输出流，流被关闭后就不能再输出数据了    
abstract public void close()	
```



# 3 IOUtils

IOUtils是apache common已经提供了一个工具类方便进行IO操作。

# 4 IO中的装饰器模式





# 5 关闭流

关闭流只需要关闭最外层的包装流，其他流会自动调用关闭，这样可以保证不会抛异常。

有些方法中close方法除了调用被包装流的close方法外还会把包装流置为null，方便JVM回收。







# 参考网址

* [java io系列01之 "目录"](https://www.cnblogs.com/skywang12345/p/io_01.html)

* [Java8 I/O源码-目录](https://blog.csdn.net/panweiwei1994/article/details/78046000)

* [Java IO流关闭问题的深入研究](https://blog.csdn.net/weixin_30722589/article/details/98624874)  [java – 为什么要关闭()一个输入流？](https://www.jb51.cc/java/126163.html)