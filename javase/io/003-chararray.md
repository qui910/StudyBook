# 1 概述

`CharArrayReader`和`CharArrayWriter`也属于**基础流**，其操作的是字符数组。



# 2 CharArrayReader

CharArrayReader 是字符数组输入流。它和ByteArrayInputStream类似，只不过ByteArrayInputStream是字节数组输入流，而CharArray是字符数组输入流。CharArrayReader 是用于读取字符数组，它继承于Reader。操作的数据是以字符为单位！

## 2.1 常用API

```java
//// 成员变量
// 字符数组缓冲
protected char buf[];
// 下一个被获取的字符的位置
protected int pos;
// 被标记的位置
protected int markedPos = 0;
// 字符缓冲的长度
protected int count;

// 构造器 根据buf数组来创建CharArrayReader对象
public CharArrayReader(char[] buf)
public CharArrayReader(char[] buf, int offset, int length)

// 关闭流
public void close()
// 保存当前位置。readAheadLimit在此处没有任何实际意义,mark()必须和reset()配合使用才有意义！     
public void mark(int readLimit)
// 读取下一个字符。即返回字符缓冲区中下一位置的值。    
public int read()
// 读取数据，并保存到字符数组b中从off开始的位置中，len是读取长度。    
public int  read(char[] buffer, int offset, int len)
// 判断“是否能读取下一个字符”。能的话，返回true。    
public boolean ready()  
// 重置“下一个读取位置”为“mark所标记的位置”    
public void reset()
// 跳过n个字符    
public long skip(long charCount)
```



## 2.2 源码分析



# 3 CharArrayWriter

## 3.1 常用API

```java
////成员变量
// 字符数组缓冲
protected char buf[];
// 下一个字符的写入位置
protected int count;

////构造器
// 默认缓冲区大小是32
public CharArrayWriter()
// 指定缓冲区大小是initialSize    
public CharArrayWriter(int initialSize)

// 将csq从start开始(包括)到end结束(不包括)的数据，写入到CharArrayWriter中。
public CharArrayWriter append(CharSequence csq, int start, int end)
// 将c写入到CharArrayWriter中,内部也是使用write实现，唯一区别是可以链式调用   
public CharArrayWriter append(char c)
// 将csq写入到CharArrayWriter中    
public CharArrayWriter append(CharSequence csq)
// 关闭流    
public void close()
// 刷新IO，无效    
public void flush()
// 重置    
public void reset()
// 返回CharArrayWriter的大小    
public int size()
// 将CharArrayWriter的全部数据对应的char[]返回    
public char[] toCharArray()
String     toString()
// 写入字符数组buffer到CharArrayWriter中。off是“字符数组buffer中的起始写入位置”，len是写入的长度    
public void write(char[] buffer, int offset, int len)
// 写入一个字符c到CharArrayWriter中    
public void write(int oneChar)
// 写入字符串str到CharArrayWriter中。off是“字符串的起始写入位置”，len是写入的长度   
public void write(String str, int offset, int count)
// 将CharArrayWriter写入到“Writer对象out”中    
public void writeTo(Writer out)
```

## 3.2 源码分析

因为在append方法中多次使用了`CharSequence`，这里进行单独的介绍。

### 3.2.1 CharSequence

CharSequence类是java.lang包下的一个接口，此接口对多种不同的对char访问的统一接口，像String、StringBuffer、StringBuilder类都是CharSequence的子接口；

CharSequence类和String类都可以定义字符串，但是String定义的字符串只能读，CharSequence定义的字符串是可读可写的；CharSequence就是字符序列，String, StringBuilder和StringBuffer本质上都是通过字符数组实现的！

# 4 示例

```java
/**
 * 测试字符数组操作类
 */
public class CharArrayReaderOrWriterTest {
    public static void main(String[] args) {
        CharArrayWriter writer = null;

        char[] chars = {'我','是','a','b','c','d','e'};

        try(
                CharArrayReader in = new CharArrayReader(chars);
        ){
            int ch = 0;
            while((ch=in.read())!=-1){
                if (in.ready() == true) {
                    System.out.println("输入数据：" + new String(new char[]{(char) ch},0,1));
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        try(
                CharArrayWriter out = new CharArrayWriter(20);
        ){
            out.write("我".toCharArray());
            out.write("是你".toCharArray());
            out.write("中国人".toCharArray());
            System.out.println("输出数据(默认):"+out.toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

