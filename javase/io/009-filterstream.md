# 1 概述

**FilterOutputStream** 的作用是用来“**封装其它的输出流，并为它们提供额外的功能**”。它主要包括BufferedOutputStream, DataOutputStream和PrintStream。

* BufferedOutputStream的作用就是为“输出流提供缓冲功能”。
* DataOutputStream 是用来装饰其它输出流，将DataOutputStream和DataInputStream输入流配合使用，“允许应用程序以与机器无关方式从底层输入流中读写基本 Java 数据类型”。
* PrintStream 是用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。

**FilterInputStream** 的作用是用来“**封装其它的输入流，并为它们提供额外的功能**”。它的常用的子类有BufferedInputStream和DataInputStream。

* BufferedInputStream的作用就是为“输入流提供缓冲功能，以及mark()和reset()功能”。

* DataInputStream是用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。应用程序可以使用DataOutputStream(数据输出流)写入由DataInputStream(数据输入流)读取的数据。

由此可以得出，FilterOutputStream和FilterInputStream及其子类，都是**过滤流**，但是需要配合基础流才能发挥功能。



# 2 常用API

## 2.1 FilterInputStream

```java
//需要被过滤的流对象，由构造器赋值
protected volatile InputStream in;
// 只有一个构造器，就是传入
protected FilterInputStream(InputStream in)
```



## 2.2 FilterOutputStream

```java
protected OutputStream out;

public FilterOutputStream(OutputStream out);
```



# 3 说明

FilterOutputStream和FilterInputStream的子类作用就是通过嵌套过滤器来添加多重功能。例如：流默认情况下是不被缓存区缓存的，即每个对read的调用都会被请求操作系统再分发一个字节。相比之下，请求一个数据库并将其置于缓冲区中会显得更加高效。

示例1：如果想要视同缓存机制，以及用于文件的数据输入方法，可以参考如下构造：

```java
DataInputStream din = new DataInputStream(
	new BufferedInputStream(
		new FileInputStream("text.txt")));
```

示例2：从一个ZIP压缩文件中读取数字

```javascript
DataInputStream din = new DataInputStream(
    new ZipInputStream(new FileInputStream("employee.zip")));
```

