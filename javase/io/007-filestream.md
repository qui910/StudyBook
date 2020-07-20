# 1 概述

FileInputStream 是文件输入流，它继承于InputStream。通常，我们使用FileInputStream从某个文件中获得输入字节。

FileOutputStream 是文件输出流，它继承于OutputStream。通常，我们使用FileOutputStream 将数据写入 File 或 FileDescriptor 的输出流。

FileInputStream和FileOutputStream在传统的IO中比较重要，因为传统IO很一大部分就是处理文件。同时二者又都是**基础流**。



# 2 常用API

## 2.1 FileInputStream

```java
// 构造函数1：创建“File对象”对应的“文件输入流”
FileInputStream(File file)  
// 构造函数2：创建“文件描述符”对应的“文件输入流”    
FileInputStream(FileDescriptor fd)
// 构造函数3：创建“文件(路径为path)”对应的“文件输入流”    
FileInputStream(String path)       

// 返回“剩余的可读取的字节数”或者“skip的字节数”    
int      available()
// 关闭“文件输入流”    
void     close() 
// 返回“FileChannel”    
FileChannel      getChannel()
// 返回“文件描述符”    
final FileDescriptor     getFD() 
// 返回“文件输入流”的下一个字节    
int      read()  
// 读取“文件输入流”的数据并存在到buffer，从byteOffset开始存储，存储长度是byteCount。    
int      read(byte[] buffer, int byteOffset, int byteCount)
// 跳过byteCount个字节    
long     skip(long byteCount)    
```



## 2.2 FileOutputStream

```java
// 构造函数1：创建“File对象”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。
FileOutputStream(File file) 
// 构造函数2：创建“File对象”对应的“文件输入流”；指定“追加模式”。    
FileOutputStream(File file, boolean append)
// 构造函数3：创建“文件描述符”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。    
FileOutputStream(FileDescriptor fd)
// 构造函数4：创建“文件(路径为path)”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。    
FileOutputStream(String path)
// 构造函数5：创建“文件(路径为path)”对应的“文件输入流”；指定“追加模式”。    
FileOutputStream(String path, boolean append) 

// 关闭“输出流”
void                    close()
// 返回“FileChannel”    
FileChannel             getChannel() 
final FileDescriptor    getFD()      // 返回“文件描述符”
void                    write(byte[] buffer, int byteOffset, int byteCount) // 将buffer写入到“文件输出流”中，从buffer的byteOffset开始写，写入长度是byteCount。
void                    write(int oneByte)  // 写入字节oneByte到“文件输出流”中
```

FileOutputStream中有三种方法写入一个换行符号

* 第一种：Windows环境下使用显示换号符号“\r\n”
* 第二种：Unix环境下使用显示换号符号“\n”
* 第三种：使用Java自定义的换行符号，这种方法具有良好的跨平台性，推荐使用。

# 3 源码分析



# 4 代码示例

```java
public class FileInOrOutputStreamTest {
    private final  static String FILE_NAME="tempfile.txt";

    public static void main(String[] args) {
//        testWrite();
        testRead();
    }

    private static void testWrite() {

        try(
            FileOutputStream out = new FileOutputStream(FILE_NAME,true);
        ){
            out.write("三个地方广东省发鬼地方个1".getBytes("UTF-8"));
            //Unix下的换行符为"\n"
            out.write("\n".getBytes());

            out.write("逗号分隔和法规和2".getBytes("UTF-8"));
            //Windows下的换行符为"\r\n"
            out.write("\r\n".getBytes());

            out.write("鬼地方个地方回复挂号费3".getBytes("UTF-8"));
            //推荐使用，具有良好的跨平台性
            out.write(System.lineSeparator().getBytes());

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void testRead() {
        try(
           // 这里要处理换行和中文的话，最好还是通过BufferedReader
           // 自己处理换行和中文非常麻烦
           FileInputStream in = new FileInputStream(FILE_NAME);
           InputStreamReader inr = new InputStreamReader(in,"UTF-8");
           BufferedReader br = new BufferedReader(inr);
        )
        {
            String line;
            while((line=br.readLine())!=null) {
                // 这里打印换行要使用println，使用printf会导致没有换行
                // 说明BufferedReader读取一行数据时，并没有将换行读出来。
                System.out.println(line);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

这里推荐使用Junit进行单元测试，遇到几个问题看如下解决

* [JUnit入门-1.在IDEA创建JUnit测试用例](https://zhuanlan.zhihu.com/p/42453559)

* [IDEA下maven工程找不到@Test](https://blog.csdn.net/seudongnan/article/details/78268508)
* [看完这个，Java IO从此不在难](https://www.jianshu.com/p/715659e4775f)