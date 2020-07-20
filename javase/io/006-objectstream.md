# 1 概述

ObjectInputStream 和 ObjectOutputStream 的作用是，**对基本数据和对象进行序列化操作支持。**属于**过滤流**，意思就是在其他的流程的基础上添加额外的功能，不能直接操作基础数据。

创建“文件输出流”对应的ObjectOutputStream对象，该ObjectOutputStream对象能提供对“基本数据或对象”的持久存储；当我们需要读取这些存储的“基本数据或对象”时，可以创建“文件输入流”对应的ObjectInputStream，进而读取出这些“基本数据或对象”。

注意： 只有支持 java.io.Serializable 或 java.io.Externalizable 接口的对象才能被ObjectInputStream/ObjectOutputStream所操作！





# 2 常用API

## 2.1 DataInput

```java
boolean readBoolean()
byte readBytes()
char readChar()
double readDouble()
float readFloat()
int readInt()
long readLong()
short readShort()
// 读入一个给定类型的值    
void readFully(byte[] b)
// 将字节读入到数组b中，期间阻塞直到所有字节都读入 b数据读入的缓冲区，off 数据起始位置的偏移量  len读入字节的最大数量    
void readFully(byte[] b,int off,int len)
// 读入由“修订过的UTF-8”格式的字符构成的字符串    
String readUTF()
// 跳过n个字节，期间阻塞直到所有字节都被跳过    
int skipBytes(int n)    
```

## 2.2 DataOutput

```java
//写出一个给定类型的值
void writeBoolean(boolean b)
void writeByte(int b)
void writeChar(int c)
void writeDouble(double b)
void wrilteFloat(float f)
void writeInt(int i)
void writeLong(long i)  
void writeShort(int s)
//写出字符串中的所有字符    
void writeChars(String s)
//写出由“修订过的UTF-8”格式的字符构成的字符串    
void writeUTF(String s)
```





## 2.3 ObjectOutputStream

```java
//// 构造函数
// 创建一个ObjectOutputStream使得你可以将对象写出到指定的OutpuStream
public ObjectOutputStream(OutputStream output)
    
   
// 函数
public void close()
public void defaultWriteObject()
public void flush()
public ObjectOutputStream.PutField putFields()
public void reset()
public void useProtocolVersion(int version)
public void write(int value)
public void write(byte[] buffer, int offset, int length)
public void writeBoolean(boolean value)
public void writeByte(int value)
public void writeBytes(String value)
public void writeChar(int value)
public void writeChars(String value)
public void writeDouble(double value)
public void writeFields()
public void writeFloat(float value)
public void writeInt(int value)
public void writeLong(long value)
// 写出指定的对象到ObjectOutputStream，这个方法将存储指定对象的类，类的签名以及这个类及其超类中所有非静态和非瞬时的域的值    
public final void writeObject(Object object)
public void writeShort(int value)
public void writeUTF(String value)
public void writeUnshared(Object object)
```



## 2.4 ObjectInputStream

```java
//// 构造函数
// 创建一个ObjectInputStream用于从指定的InputStream中读会对象信息
ObjectInputStream(InputStream input)

public int available()
public void close()
public void defaultReadObject()
public int read(byte[] buffer, int offset, int length)
public int read()
public boolean readBoolean()
public byte readByte()
public char readChar()
public double readDouble()
public ObjectInputStream.GetField readFields()
public float readFloat()
public void readFully(byte[] dst)
public void readFully(byte[] dst, int offset, int byteCount)
public int readInt()
public String readLine()
public long readLong()
// 从ObjectInputStream中读入一个对象，特别是这个方法会读回对象的类，类的签名以及其超类中所有非静态和非瞬时的域的值。它执行的发序列化允许恢复多个对象应用。   
public final Object readObject()
public short readShort()
public String readUTF()
public Object readUnshared()
public int readUnsignedByte()
public int readUnsignedShort()
public synchronized void registerValidation(ObjectInputValidation object, int priority)
public int skipBytes(int length)
```



# 3 代码示例

```java
/**
 * 对象操作流测试
 */
public class ObjectInOrOutStream {

    private static final String TMP_FILE = "box.tmp";

    private static class User implements Serializable{
        long id;
        String name;

        public User(long id, String name) {
            this.id = id;
            this.name = name;
        }

        @Override
        public String toString() {
            return "User{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }

        private void writeObject(ObjectOutputStream out)
                throws IOException{
            System.out.println("开始序列化");
            // 使定制的writeObject()方法可以利用自动序列化中内置的逻辑。
            out.defaultWriteObject();
            out.writeLong(id+1000);
        }

        private void readObject(ObjectInputStream in)
                throws IOException, ClassNotFoundException {
            System.out.println("开始反序列化");
            in.defaultReadObject();
            id=in.readLong();
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        testWriteObject();
        testReadObject();
    }

    private static void testWriteObject() throws IOException {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(TMP_FILE));
        out.writeBoolean(false);
        out.writeBytes("abcd");
        out.writeChar('e');
        out.writeChars("fgh");
        out.writeDouble(1.3);
        out.writeLong(1233423);
        out.writeObject(new User(1,"prd"));
        out.close();
    }

    private static void testReadObject() throws IOException, ClassNotFoundException {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream(TMP_FILE));
        System.out.println("1:"+in.readBoolean());
        System.out.println("2:"+new String(new byte[]{in.readByte()}));
        System.out.println("2:"+new String(new byte[]{in.readByte()}));
        System.out.println("2:"+new String(new byte[]{in.readByte()}));
        System.out.println("2:"+new String(new byte[]{in.readByte()}));
        System.out.println("3:"+in.readChar());
        System.out.println("3:"+in.readChar());
        System.out.println("3:"+in.readChar());
        System.out.println("3:"+in.readChar());
        System.out.println("4:"+in.readDouble());
        System.out.println("5:"+in.readLong());
        System.out.println("6:"+in.readObject());
        in.close();
    }
}
```





# 4 序列化

序列化，就是为了**保存对象的状态**；而与之对应的反序列化，则可以**把保存的对象状态再读出来**。简言之：**序列化/反序列化，是Java提供一种专门用于的保存/恢复对象状态的机制。**

一般在以下几种情况下，我们可能会用到序列化：
a）**当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；**
b）**当你想用套接字在网络上传送对象的时候；**
c）**当你想通过RMI传输对象的时候。**

## 4.1 Serializable

如果想序列化我们自定义的类，需要实现Serializable接口，从而能支持对象的保存/恢复（Serializable接口只是个标记接口）。

若要支持序列化，除了“自定义实现Serializable接口的类”之外；java的“基本类型”和“java自带的实现了Serializable接口的类”，都支持序列化。

但是在序列化时要格外注意一下几点：

* 序列化对static和transient变量，是不会自动进行状态保存的。transient的作用就是，用transient声明的变量，不会被自动序列化。
* 对于Socket, Thread类，不支持序列化。若实现序列化的接口中，有Thread成员；在对该类进行序列化操作时，编译会出错！这主要是基于资源分配方面的原因。如果Socket，Thread类可以被序列化，但是被反序列化之后也无法对他们进行重新的资源分配；再者，也是没有必要这样实现。

## 4.2 修改默认的序列化机制

某些数据域是不可以序列化的，Java提供了简单的机制防止某些域被序列化。那就是将它们标记位transient。

另外还有如果想修改默认的序列化机制，就在我们要序列化的对象中，添加以下方法，就可以在默认的读写行为中添加任何验证或其他的操作。

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws IOException;
        
private void readObject(java.io.ObjectInputStream s)
    throws IOException, ClassNotFoundException;    
```





# 5 Externalizable

如果一个类要完全负责自己的序列化，则实现Externalizable接口，而不是Serializable接口。

Externalizable接口定义包括两个方法writeExternal()与readExternal()。需要注意的是：声明类实现Externalizable接口会有重大的安全风险。writeExternal()与readExternal()方法声明为public，恶意类可以用这些方法读取和写入对象数据。如果对象包含敏感信息，则要格外小心。

* 实现Externalizable接口的类，不会像实现Serializable接口那样，会自动将数据保存。
*  实现Externalizable接口的类，必须实现writeExternal()和readExternal()接口！否则，程序无法正常编译！
* 实现Externalizable接口的类，必须定义不带参数的构造函数！否则，程序无法正常编译！
* writeExternal() 和 readExternal() 的方法都是public的，不是非常安全！