# 1 概述
`ByteArrayInputStream`和`ByteArrayOutputStream`属于**基础流**，是指直接操作数据源的类。这种类不需要和其他IO流组合，就可以直接读取或写入数据。

# 2 ByteArrayOutputStream

ByteArrayInputStream 是字节数组输入流。它继承于InputStream。它包含一个内部缓冲区，该缓冲区包含从流中读取的字节；通俗点说，它的内部缓冲区就是一个字节数组，而ByteArrayInputStream本质就是通过字节数组来实现的。

我们都知道，InputStream通过read()向外提供接口，供它们来读取字节数据；而ByteArrayInputStream 的内部额外的定义了一个计数器，它被用来跟踪 read() 方法要读取的下一个字节。

## 2.1 常用API

```java
//// 成员变量
// 保存字节输入流数据的字节数组
protected byte buf[];
// 下一个会被读取的字节的索引
protected int pos;
// 标记的索引
protected int mark = 0;
// 字节流的长度
protected int count;

//// 构造函数
// 创建一个内容为buf的字节流
public ByteArrayInputStream(byte[] buf)
// 创建一个内容为buf的字节流，并且是从offset开始读取数据，读取的长度为length    
public ByteArrayInputStream(byte[] buf, int offset, int length)

// 能否读取字节流的下一个字节
public synchronized int available()
// 为何实现都为空呢？？    
public void close();
// 保存当前位置。readAheadLimit在此处没有任何实际意义
public synchronized void mark(int readlimit)
// 是否支持“标签”    
public boolean  markSupported()
// 读取下一个字节
public synchronized int read()
// 将“字节流的数据写入到字节数组b中”,并返回读取的长度    
public synchronized int read(byte[] buffer, int offset, int length)
// 重置“字节流的读取索引”为“mark所标记的位置”    
public synchronized void reset()
// 跳过“字节流”中的byteCount个字节。    
public synchronized long skip(long byteCount)
```



## 2.2 源码解析

```java
public synchronized int read() { 
   return (pos < count) ? (buf[pos++] & 0xff) : -1;
}
```

这里& 0xff就是取 int32位中低8位的数据。其中0xff的二进制为 1111 1111。如果buf中值为`1111 1111 1111 1111 1111 1111 1000 0001`时，`& 1111 1111`结果就是`1000 0001`。个人觉得这里的作用比较小。因为byte的范围是-128~127之间，大于这个范围的数据写入byte[]后本身就会被截取。



# 3 ByteArrayOutputStream

ByteArrayOutputStream 是字节数组输出流。它继承于OutputStream。
ByteArrayOutputStream 中的数据被写入一个 byte 数组。缓冲区会随着数据的不断写入而自动增长。可使用 toByteArray() 和 toString() 获取数据。

## 3.1 常用API

```java
//// 成员变量
// 保存“字节数组输出流”数据的数组
protected byte buf[];
// “字节数组输出流”的计数
protected int count;

//// 构造函数
// 默认创建的字节数组大小是32。
public  ByteArrayOutputStream()
// 创建指定数组大小的“字节数组输出流”
public  ByteArrayOutputStream(int size)

// 关闭流
public void close()
// 重置“字节数组输出流”的计数,从0开始    
public synchronized void reset()
// 返回“字节数组输出流”当前计数值    
public int size()
// 将“字节数组输出流”转换成字节数组。    
public synchronized byte[]  toByteArray()
// 以特定的编码将“字节数组输出流”转换成字符串。    
public String  toString(String charsetName)
// 将“字节数组输出流”转换成字符串。    
public String toString()
// 写入字节数组b到“字节数组输出流”中。off是“写入字节数组b的起始位置”，len是写入的长度
public synchronized void write(byte[] buffer, int offset, int len)
// 写入一个字节b到“字节数组输出流”中，并将计数+1
public synchronized void write(int oneByte)
// 写入输出流outb到“字节数组输出流”中。    
public synchronized void writeTo(OutputStream out)
```

## 3.2 源码解析



# 4 示例

```java
/**
 * 测试字节数组操作流
 */
public class ByteArrayInOrOutStreamTest {

    public static void main(String[] args) {

        // 对应英文字母“abcddefghijklmnopqrsttuvwxyz”
        byte[] arrayLetters = {
                0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F,
                0x70, 0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A
        };

        try(
           ByteArrayInputStream in = new ByteArrayInputStream(arrayLetters);
        ){
            int ch = 0;
            while((ch=in.read())!=-1){
                System.out.println("输入数据："+new String(new byte[]{(byte) ch}));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        try(
           ByteArrayOutputStream out = new ByteArrayOutputStream(10);
        ){
            // 依次写入“A”、“B”、“C”三个字母。0x41对应A，0x42对应B，0x43对应C。
            out.write(0x41);
            out.write(0x42);
            out.write(0x43);
            // 将ArrayLetters数组中从“3”开始的后5个字节写入到baos中。
            // 即对应写入“0x64, 0x65, 0x66, 0x67, 0x68”，即“defgh”
            out.write(arrayLetters, 3, 5);
            System.out.printf("输出=%s\n", out);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

