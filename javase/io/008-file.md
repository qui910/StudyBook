# 1 概述

File 是“**文件**”和“**目录路径名**”的抽象表示形式。

File 直接继承于Object，实现了Serializable接口和Comparable接口。实现Serializable接口，意味着File对象支持序列化操作。而实现Comparable接口，意味着File对象之间可以比较大小；File能直接被存储在有序集合(如TreeSet、TreeMap中)。



# 2 常用API

```java
// 静态成员
// 路径分割符":"
public static final String     pathSeparator  
// 路径分割符':'    
public static final char     pathSeparatorChar 
// 分隔符"/"    
public static final String     separator 
// 分隔符'/'    
public static final char     separatorChar        

// 构造函数
File(File dir, String name)
File(String path)
File(String dirPath, String name)
File(URI uri)

// 成员函数
// 测试应用程序是否可以执行此抽象路径名表示的文件。
boolean    canExecute() 
// 测试应用程序是否可以读取此抽象路径名表示的文件。    
boolean    canRead()
// 测试应用程序是否可以修改此抽象路径名表示的文件。    
boolean    canWrite() 
// 按字母顺序比较两个抽象路径名。    
int    compareTo(File pathname)
 // 当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。 
boolean    createNewFile()  
 // 在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称。    
static File    createTempFile(String prefix, String suffix) 
// 在指定目录中创建一个新的空文件，使用给定的前缀和后缀字符串生成其名称。    
static File    createTempFile(String prefix, String suffix, File directory) 
// 删除此抽象路径名表示的文件或目录。    
boolean    delete()  
// 在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。    
void    deleteOnExit()
// 测试此抽象路径名与给定对象是否相等。    
boolean    equals(Object obj)
// 测试此抽象路径名表示的文件或目录是否存在。    
boolean    exists() 
// 返回此抽象路径名的绝对路径名形式。    
File    getAbsoluteFile()
// 返回此抽象路径名的绝对路径名字符串。    
String    getAbsolutePath()
// 返回此抽象路径名的规范形式。    
File    getCanonicalFile()
// 返回此抽象路径名的规范路径名字符串。    
String    getCanonicalPath() 
// 返回此抽象路径名指定的分区中未分配的字节数。    
long    getFreeSpace()  
// 返回由此抽象路径名表示的文件或目录的名称。    
String    getName()
// 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。    
String    getParent()  
 // 返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回 null。    
File    getParentFile()
// 将此抽象路径名转换为一个路径名字符串。    
String    getPath() 
// 返回此抽象路径名指定的分区大小。    
long    getTotalSpace()
// 返回此抽象路径名指定的分区上可用于此虚拟机的字节数。    
long    getUsableSpace()
// 计算此抽象路径名的哈希码。    
int    hashCode() 
// 测试此抽象路径名是否为绝对路径名。    
boolean    isAbsolute()
// 测试此抽象路径名表示的文件是否是一个目录。    
boolean    isDirectory() 
// 测试此抽象路径名表示的文件是否是一个标准文件。    
boolean    isFile()
// 测试此抽象路径名指定的文件是否是一个隐藏文件。    
boolean    isHidden() 
// 返回此抽象路径名表示的文件最后一次被修改的时间。    
long    lastModified()
// 返回由此抽象路径名表示的文件的长度。    
long    length()       
// 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录。    
String[]    list()
// 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。    
String[]    list(FilenameFilter filter) 
// 返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。    
File[]    listFiles()
// 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。    
File[]    listFiles(FileFilter filter) 
// 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。    
File[]    listFiles(FilenameFilter filter)
// 列出可用的文件系统根。    
static File[]    listRoots() 
// 创建此抽象路径名指定的目录。    
boolean    mkdir() 
// 创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。    
boolean    mkdirs()
// 重新命名此抽象路径名表示的文件。    
boolean    renameTo(File dest)
// 设置此抽象路径名所有者执行权限的一个便捷方法。    
boolean    setExecutable(boolean executable) 
// 设置此抽象路径名的所有者或所有用户的执行权限。    
boolean    setExecutable(boolean executable, boolean ownerOnly) 
// 设置此抽象路径名指定的文件或目录的最后一次修改时间。    
boolean    setLastModified(long time) 
// 设置此抽象路径名所有者读权限的一个便捷方法。    
boolean    setReadable(boolean readable)  
// 设置此抽象路径名的所有者或所有用户的读权限。    
boolean    setReadable(boolean readable, boolean ownerOnly)  
// 标记此抽象路径名指定的文件或目录，从而只能对其进行读操作。    
boolean    setReadOnly()    
// 设置此抽象路径名所有者写权限的一个便捷方法。    
boolean    setWritable(boolean writable)    
// 设置此抽象路径名的所有者或所有用户的写权限。    
boolean    setWritable(boolean writable, boolean ownerOnly)  
// 返回此抽象路径名的路径名字符串。    
String    toString()  
// 构造一个表示此抽象路径名的 file: URI。    
URI    toURI()
// 已过时。 此方法不会自动转义 URL 中的非法字符。建议新的代码使用以下方式将抽象路径名转换为 URL：首先通过 toURI 方法将其转换为 URI，然后通过 URI.toURL 方法将 URI 装换为 URL。    
URL    toURL()    
```

# 3 示例

## 3.1 新建目录的常用方法

**方法1：根据相对路径新建目录。**

示例代码如下(在当前路径下新建目录“dir”)：

```java
File dir = new File("dir");
dir.mkdir();
```

**方法2：根据绝对路径新建目录。**

示例代码如下(新建目录“/home/test/dir”)：

```java
File dir = new File("/home/test/dir");
dir.mkdirs();
```

**说明**：上面是在linux系统下新建目录“/home/test/dir”的源码。在windows下面，若要新建目录“D:/dir”，源码如下：

```java
File dir = new File("D:/dir");
dir.mkdir();
```

方法3

```java
URI uri = new URI("file:/home/test/dir"); 
File dir = new File(uri);
sub.mkdir();
```

说明： 和“方法2”类似，只不过“方法2”中传入的是完整路径，而“方法3”中传入的是完整路径对应URI。


## 3.2 新建子目录的几种常用方法

例如，我们想要在当前目录的子目录“dir”下，再新建一个子目录。有一下几种方法:

**方法1**

```java
File sub1 = new File("dir", "sub1");
sub1.mkdir();
```

**说明**：上面的方法作用是，在当前目录下 "dir/sub1"。它能正常运行的前提是“sub1”的父目录“dir”已经存在！

**方法2**

```java
File sub2 = new File(dir, "sub2");
sub2.mkdir();
```

**说明**：上面的方法作用是，在当前目录下 "dir/sub2"。它能正常运行的前提是“sub2”的父目录“dir”已经存在！

**方法3**

```java
File sub3 = new File("dir/sub3");
sub3.mkdirs();
```

**说明**：上面的方法作用是，在当前目录下 "dir/sub3"。它不需要dir已经存在，也能正常运行；若“sub3”的父母路不存在，mkdirs()方法会自动创建父目录。

**方法4**

```java
File sub4 = new File("/home/test/dir/sub4");
sub4.mkdirs();
```

**说明**：上面的方法作用是，新建目录"/home/test/dir/sub3"。它不需要dir已经存在，也能正常运行；若“sub4”的父母路不存在，mkdirs()方法会自动创建父目录。

**方法5**

```java
URI uri = new URI("file:/home/test/dir/sub5"); 
File sub5 = new File(uri);
sub5.mkdirs();
```

**说明**： 和“方法4”类似，只不过“方法4”中传入的是完整路径，而“方法5”中传入的是完整路径对应URI。

 

## 3.3 新建文件的几种常用方法

例如，我们想要在当前目录的子目录“dir”下，新建一个文件。有一下几种方法

**方法1**

```java
try {
    File dir = new File("dir");    // 获取目录“dir”对应的File对象
    File file1 = new File(dir, "file1.txt");
    file1.createNewFile();
} catch (IOException e) {
    e.printStackTrace();
}
```

**说明**：上面代码作用是，在“dir”目录(相对路径)下新建文件“file1.txt”。

**方法2**

```java
try {
    File file2 = new File("dir", "file2.txt");
    file2.createNewFile();
} catch (IOException e) {
    e.printStackTrace();
}
```

**说明**：上面代码作用是，在“dir”目录(相对路径)下新建文件“file2.txt”。

**方法3**

```java
try {
    File file3 = new File("/home/test/dir/file3.txt");
    file3.createNewFile();
} catch (IOException e) {
    e.printStackTrace();
}
```

**说明**：上面代码作用是，下新建文件“/home/test/dir/file3.txt”(绝对路径)。这是在linux下根据绝对路径的方法，在windows下可以通过以下代码新建文件"D:/dir/file4.txt"。

```java
try {
    File file3 = new File("D:/dir/file4.txt");
    file3.createNewFile();
} catch (IOException e) {
    e.printStackTrace();
}
```

**方法4**

```java
try {
    URI uri = new URI("file:/home/test/dir/file4.txt"); 
    File file4 = new File(uri);
    file4.createNewFile();
} catch (IOException e) {
    e.printStackTrace();
}
```



# 4 FileDescriptor 

FileDescriptor可以看做一种指向文件引用的抽象化概念。它能表示一个开放文件，一个开放的socket或者一个字节的源。它最主要的用途就是去创建FileInputStream或者FileOutputStream。并且也说了不应该创建应用自己的文件描述符。

## 4.1 成员变量

```java
//封装了一个int型的值fd，每个打开的文件，socket等都会给你一个fd值，通过该值可以对文件，socket等进行相关操作，有点儿类似操作对象的索引
private int fd;

//封装了一个boolean类型变量closed，用来判断fd是否被释放
private boolean closed;

//定义的一个标准输入(键盘)，fd值为0
public static final FileDescriptor in = new FileDescriptor(0);
 
//定义的一个标准输出(屏幕)，fd值为1
public static final FileDescriptor out = new FileDescriptor(1);
 
//定义的一个标准错误输出(屏幕)，fd值为2，我想在不同的环境下，标准流都能工作的原因应该就是因为已经在这里限定好了吧
public static final FileDescriptor err = new FileDescriptor(2);

```

## 4.2 构造器

```java
//不带参构造，默认fd为-1，目测-1没什么意义，符合官方说的不建议直接创建自己的对象
public FileDescriptor() {
	fd = -1;
}

//一个带参私有的构造方法，将fd值设为传入的参数
private static FileDescriptor standardStream(int fd) {
    FileDescriptor desc = new FileDescriptor();
    desc.handle = set(fd);
    return desc;
}
```



## 4.3 源码解析

### 4.3.1 out的作用和原理

FileDescriptor.out就是机器的“标准输出(屏幕)”的文件标识符。例如我们在linux中进行shell编程时，日志输出就是用的

```shell
标准输入stdin：对应的文件描述符是0，符号是<和<<，/dev/stdin -> /proc/self/fd/0

标准输出stdout：对应的文件描述符是1，符号是>和>>，/dev/stdout -> /proc/self/fd/1

标准错误stderr：对应的文件描述符是2，符号是2>和2>>，/dev/stderr -> /proc/self/fd/2
```

所以这里通俗理解，out就代表了标准输出(屏幕)。若我们要输出信息到屏幕上，即可通过out来进行操作；但是，out又没有提供输出信息到屏幕的接口(因为out本质是FileDescriptor对象，而FileDescriptor没有输出接口)。

怎么办呢？

很简单，我们创建out对应的“输出流对象”，然后通过“输出流”的write()等输出接口就可以将信息输出到屏幕上。如下代码：

```java
try {
    FileOutputStream out = new FileOutputStream(FileDescriptor.out);
    out.write('A');
    out.close();
} catch (IOException e) {
}
```

执行上面的程序，会在屏幕上输出字母'A'。

为了方便我们操作，java早已为我们封装好了“能方便的在屏幕上输出信息的接口”：通过System.out，我们能方便的输出信息到屏幕上。
因此，我们可以等价的将上面的程序转换为如下代码：`System.out.print('A');`

下面讲讲上面两段代码的原理

* 查看看out的定义。它的定义在**FileDescriptor.java**中，相关源码如下：

```java
public final class FileDescriptor {

    private int fd;

    public static final FileDescriptor out = new FileDescriptor(1);
    
    private FileDescriptor(int fd) {
        this.fd = fd;
        useCount = new AtomicInteger();
    }

    ...
}
```

从中，可以看出

* out就是一个FileDescriptor对象。它是通过构造函数FileDescriptor(int fd)创建的。
* FileDescriptor(int fd)的操作：就是给fd对象(int类型)赋值，并新建一个使用计数变量useCount。
* fd对象是非常重要的一个变量，“fd=1”就代表了“标准输出”，“fd=0”就代表了“标准输入”，“fd=2”就代表了“标准错误输出”。

`System.in`，`System.out`，`System.err`就是通过FileDescriptor的`in`，`out`，`err`来实现的。

## 4.4 示例

```java
    /**
     * 证明了System.out,就是使用FileDescriptor.out来创建的
     * 二者的信息都在控制台输出
     */
    @Test
    public void testSystemOutBuildByOut() throws IOException {
        FileOutputStream out = new FileOutputStream(FileDescriptor.out);
        out.write("测试FileDescriptor.out输出\n".getBytes());
        System.out.println("测试System.out输出");
    }

    /**
     * 验证对同一文件建立的两个不同的流，其fd值是不同的
     */
    @Test
    public void testFileFFDNum() throws IOException {
        FileOutputStream fos1 = new FileOutputStream(FILE_NAME);
        FileOutputStream fos2 = new FileOutputStream(FILE_NAME);
        System.out.println("fos1:"+fos1.getFD());
        System.out.println("fos2:"+fos2.getFD());
    }

    /**
     * 验证"通过文件名创建FileOutputStream"与“通过文件描述符创建FileOutputStream”对象是等效的
     * 该程序会在“该源文件”所在目录新建文件"file.txt"，并且文件内容是"Aa"。
     * 并且流关闭后，FileDescriptor不可用了
     */
    @Test
    public void testConstructor() throws IOException {
        // 新建文件“file.txt”对应的FileOutputStream对象
        FileOutputStream out1 = new FileOutputStream(FILE_NAME);
        // 获取文件“file.txt”对应的“文件描述符”
        FileDescriptor fdout = out1.getFD();
        // 根据“文件描述符”创建“FileOutputStream”对象
        FileOutputStream out2 = new FileOutputStream(fdout);

        out1.write('A');    // 通过out1向“file.txt”中写入'A'
        out2.write('a');    // 通过out2向“file.txt”中写入'a'

        if (fdout!=null)
            System.out.printf("关闭前fdout(%s) is %s\n",fdout, fdout.valid());

        out1.close();
        out2.close();

        if (fdout!=null)
            System.out.printf("关闭后fdout(%s) is %s\n",fdout, fdout.valid());

    }
```

