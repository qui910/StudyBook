# 1 概述

在Java中，**PipedOutputStream**和**PipedInputStream**分别是管道输出流和管道输入流，它们的作用是让多线程可以通过管道进行线程间的通讯。也可以归属于**基础流**，因为它们不依赖其他流而存在，而是相互依赖。

在使用管道通信时，必须将PipedOutputStream和PipedInputStream配套使用。
使用管道通信时，大致的流程是：我们在线程A中向PipedOutputStream中写入数据，这些数据会自动的发送到与PipedOutputStream对应的PipedInputStream中，进而存储在PipedInputStream的缓冲中；此时，线程B通过读取PipedInputStream中的数据。就可以实现，线程A和线程B的通信。

# 2 常用API

## 2.1 PipedInputStream

```java
// 读取流中more缓存大小，一次write的数据量不能大于这个数字
private static final int DEFAULT_PIPE_SIZE = 1024;
// 缓存数组
protected byte buffer[];

//// 构造器
// 和输出流建立连接，并设置缓存默认大小1024
public PipedInputStream(PipedOutputStream src);
// 和输出流建立连接，并设置缓存自定义大小
public PipedInputStream(PipedOutputStream src, int pipeSize);
// 设置缓存默认大小1024，未和输出流建立连接，后续调用connect来建立连接
public PipedInputStream();
// 设置缓存自定义大小
public PipedInputStream(int pipeSize);

//// 方法
// 与输出流建立连接
public void connect(PipedOutputStream src)
// 读取下一个字符。即返回字符缓冲区中下一位置的值。    
public synchronized int read()
// 读取数据，并保存到字符数组b中从off开始的位置中，len是读取长度。    
public synchronized int read(byte b[], int off, int len)
// 能否读取字节流的下一个字节    
public synchronized int available()
// 关闭连接
public void close()
```



## 2.2 PipedOutputStream

```java
// 输出和输入流建立关系后，sink会指向输入流，两个线程的信息专递就依靠这个
private PipedInputStream sink;

//// 构造器
// 也可以在构造输出流的时候来和输入流建立关系
public PipedOutputStream(PipedInputStream snk)
    
//// 方法
// 与输出入建立连接，上面的connect其实也是用这个实现的 
public synchronized void connect(PipedInputStream snk)
// 输出数据，通过调用sink.receive建数据写入到输入流的buffer[]中    
public void write(int b)
// 刷新缓存，并通过notifyAll唤醒读线程    
public synchronized void flush()  
// 关闭连接并唤醒阻塞线程    
public void close()    
```



# 3 源码解析









#  4 示例

```java
public class PipedInOrOutStreamTest {

    public static void main(String[] args) throws IOException, InterruptedException {
        PipedOutputStream out = new PipedOutputStream();
        PipedInputStream in = new PipedInputStream(out);

        Writers writers = new Writers("W",out);
        Readers readers = new Readers("R",in);
        writers.start();
        Thread.sleep(1000);
        readers.start();
    }


    private static class Readers extends Thread {
        PipedInputStream in;

        public Readers(String name, PipedInputStream in) {
            super(name);
            this.in = in;
        }

        @Override
        public void run() {
            byte[] bytes = new byte[4];
            int ch=0;
            try {
                while (ch!=-1) {
                    ch=in.read(bytes);
                    if (ch!=-1) {
                        System.out.println("取得：" + new String(bytes, "UTF-8"));
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private static class Writers extends Thread {
        PipedOutputStream out;

        public Writers(String name, PipedOutputStream out) {
            super(name);
            this.out = out;
        }

        @Override
        public void run() {
            String msg = "我是你胜多负少的风格";
            try {
                out.write(msg.getBytes("UTF-8"));
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

这里要格外注意，首先定义的`bytes`大小为4，又可知Unicode表示中文是2个字节，所以`bytes`是每次获取中文时，可能会获取到半个中文（1个字节），导致出现乱码。解决方式有两种：

1. 扩大`bytes`的定义空间，保证能存储所有中文字符。
2. 二就是使用字符流处理，而非字节流。