# 1 概述

本章主要介绍下在Java网络编程模型中，常用的BIO，NIO，AIO等3个相关的概念。

首先，网络编程的本质是**进程间通信**。通信的基础是IO模型。Java中的IO模型`java.io`前面已经进行过介绍了，而这些基本都是基于本地文件，内存等可以统称为磁盘IO。

而通过网络传输数据，可以称为网络IO。

## 1.1 网络模型

在讨论网络IO的过程中，不可避免的要涉及到五大网络模型的概念，本节就主要介绍下这个。

### 1.1.1 URL解析

协议://域名:端口:路径?参数

```http
http://www.google.com:80/search?q=test&safe=strict
```

### 1.1.2 DNS解析

又称为域名解析，就是指域名与IP地址的映射关系解析。域名解析式从右向左解析的。例如：www.google.com的完整域名应该是

```http
# 通常root会被省略掉
www.google.com.root
```

根域名： `.root`

顶级域名:  `.com`   `.edu`  `.org`

次级域名：`.google`  `.mit`       -->自己网站时申请

主机名：`www`                               -->自定义即可

#### 1.1.2.1 DNS递归查询

存储域名的服务，类似于分布式数据库。

![io-011](..\images\io-011.png)

#### 1.1.2.2 DNS迭代查询

![io-012](..\images\io-012.png)

### 1.1.3 协议

* 应用层：HTTP FTP SMTP 

应用层数据包格式：其数据是写在`TCP/UDP数据包`中的。

* 传输层：TCP UDP   解决传递主机的具体端口的连接

传输层数据包格式：

```json
# 最大1518个字节，传输01二进制数据
帧： [Ethernet标头（18个字节  网卡地址）] [Ethernet数据（最大1500字节）[IP标头（20字节）] [IP数据 （1480字节）[TCP/UDP标头（TCP20字节，UDP8字节）] [TCP/UDP数据 （TCP1460字节，UDP1472字节）] ] ]
```



* 网络层：IP    主要解决网络导航问题，更方便的将信息传输到特定主机

网络层数据包格式: （网络层与链路层管理紧密，所以数据包格式也相近）

```json
# 最大1518个字节，传输01二进制数据
帧： [Ethernet标头（18字节  网卡地址）] [Ethernet数据（最大1500字节）[IP标头（20字节）] [IP数据（1480字节）] ]     
```



* 链路层：Ethernet  主要解决网卡之间的连接

链路层数据包格式:

```json
# 最大1518个字节，传输01二进制数据
帧： [Ethernet标头（18个字节  网卡地址）] [Ethernet数据（最大1500字节）]     
```

* 实体层：电信号   主要解决计算之前的物理连接



从上面的分析来看，在普通的局域网环境下，UDP的数据最大为1472字节最好(避免分片重组)。但在网络编程中，Internet中的路由器可能有设置成不同的值(小于默认值)，Internet上的标准MTU值为576，所以Internet的UDP编程时数据长度最好在576－20－8＝548字节以内。具体可以参考下:[TCP、UDP数据包大小的限制](

## 1.2 Socket

而在网络IO中，Socket也是一种数据源，可以通过socket（套接字）获取的IO流。Socket是网络通信的端点。在TCP/IP协议中，IP和PORT可以唯一的确定一台主机，那我们可以将`IP+PORT` 与`Socket`绑定，称为端点，可以分为客户端或服务端端点。

![io-013](..\images\io-013.png)

Unix系统中一切皆是文件，Socket可以理解为是**套接字**文件。而文件描述符表示的就是已打开文件的索引（即第8章中的FileDescriptor），文件描述符是Int型的数值，比0表示标准输入，1表示标准输出，2表示标准错误输出。其他的诸如网络等“文件”的打开都会产生一个Int型的文件描述符。

每个进程都会维护一个文件描述符表（后续详解，单独在Linux章节说明）。

### 1.2.1 通过Socket发送数据

![io-014](..\images\io-014.png)

（1）创建Socket端点

（2）将IP+端口 与Socket端点绑定，并通知网卡驱动程序

（3）应用程序发送数据到Socket端点

（4）Socket端点转发到网卡驱动程序

### 1.2.2 通过Socket接收数据

![io-015](..\images\io-015.png)

## 1.3 同步、异步和阻塞、非阻塞

同步synchronous、异步asynchronous，他们的区别就是发起任务后，本身的一个状态——如果是一直等待结果，那就是同步；如果立即返回，并采用其他的方式得到结果就是异步（比如，状态、通知、回调）。

阻塞blocking、非阻塞non-blocking，则聚焦的是CPU在等待结果的过程中的状态。在等待数据的过程中，CPU并不做什么事情是阻塞；如果CPU同时还处理其他事情就是非阻塞的。

### 1.3.1 用户空间与内核空间

理解IO底层的通信模型，要了解操作系统是如何处理IO，同步阻塞是如何造成的？异步非阻塞又是什么原理？要理解这些需要先了解下用户空间和内核空间的概念。

这个概念就涉及操作系统了，为了保护操作系统的安全，会将内存分为**用户空间**和**内核空间**两个部分。如果用户想要操作内核空间的数据，需要把数据从内和空间拷贝到用户控件。

#### 1.3.1.1 磁盘IO

在读取和写入文件的IO操作都是调用操作系统提供的接口（读写分别对应前面的`read`和`write`两个接口），因为磁盘设备是由操作系统管理的，应用程序要访问物理设备只能通过系统调用的方式来工作。这时磁盘IO的工作方式就是：

![io-004](..\images\io-004.png)

#### 1.3.1.2 网络IO

磁盘IO在前面的19个章节已经讲解过了，这里我们的重点是网络IO。举例说明：服务器接收客户端发过来的请求，想要进行处理，大致会经过下面几个步骤：

1. 服务器的网络驱动接收到消息，去内核上申请空间；并等待完整的数据包到达（有可能分组传送，没传完...），复制到内核空间；
2. 数据从内核空间拷贝到用户空间
3. 用户程序进行处理

因此大致可以把接收消息理解为两个阶段：1. 等待数据到达 2. 拷贝到用户空间

这就是网络IO的原理。

https://blog.csdn.net/chunqingtai2922/article/details/101029629)

### 1.3.2 同步阻塞IO

就是在应用程序调用网络接口时，等待网络传输数据，等待数据从内核拷贝至用户空间这段时间中，`read`或`write`请求均是阻塞状态。只有等数据都准备好后，才执行下步操作。示意图如下：

![io-005](..\images\io-005.png)

### 1.3.3 同步非阻塞IO

就是应用程序在等待数据这个过程中，如果遇到等待就直接让出CPU，去做其他操作，过后会获取CPU取询问数据是否送达。因此非阻塞IO基于状态轮训的方式，虽然能让程序在等待的过程中做点其他的事情，但是频繁的切换运行程序，反而会造成很大的压力。示意图如下：

![io-006](..\images\io-006.png)

### 1.3.4 IO多路复用/事件驱动

其实NIO或者Netty就是基于这种模式，一个线程就可以监听很多IO操作，这样在IO等待上就高效多了。

具体实现是依赖于操作系统的，windows和linux都有不同的实现方式。最初的select或者poll，都有并发数的限制，并且NIO的select还有空轮训的问题；epool则突破了连接数的限制，一个线程就可以监听大量的IO操作。这个感兴趣的朋友，可以深入了解下select、poll、epool的原理。![io-007](..\images\io-007.png)

### 1.3.5 信号驱动IO

UNIX网络编程里面的信号驱动，可没这么简单，这个信号是依赖于操作系统底层的，捕获信号或者处理都很麻烦，所以现在应用的也不是很广泛。

![io-008](..\images\io-008.png)

### 1.3.6 异步非阻塞IO

异步非阻塞就是应用程序先发送数据请求后，直接返回不做等待处理。等数据准备好后再调用回调函数来处理业务。这个过程就是异步非阻塞的，消息的等待和处理都在服务器端完成，用户只要最后接收到消息处理完的通知就行了。

![io-009](..\images\io-009.png)

### 1.3.7 总结几种方式

![io-010](..\images\io-010.png)

# 1.4 网络通信处理多个请求的方法

### 1.4.1 多个线程并发处理

![io-016](..\images\io-016.png)

缺点：多线程处理中不断的新建，关闭线程会耗费系统资源

![io-017](..\images\io-017.png)

### 1.4.2 复用线程

针对上小节的缺点，改进就是 复用线程。避免重复创建，销毁线程的浪费，只保留一定数据量的线程，称为线程池。Java提供的线程池`ExecutorService`。`Executors`。

```
Runnable/Callable -> ExecutorServuice->Future (isDone(),get())
```



`newSingleThreadExecutor`：只有1个线程，不希望重复创建线程，实现线程复用时使用。

`newFixedThreadPool`：创建固定大小的线程池，例如对应主机的核数大小，创建同样大小的线程池。

`newCachedThreadPool`：尽量复用线程，如果线程都用完，则创建新线程。

`newScheduledThreadPool`：固定频率执行



## 1.5 Socket与ServerSocket通信机制

![io-018](..\images\io-018.png)



# 2 Java IO

Java网络IO的演化，从最开始JDK1.4之前是基于阻塞的IO；发展到1.4发布后的Nio提供了selector多路复用的机制以及channel和buffer，再到1.7的NIO升级提供了真正的异步api；再发展到后来崭露头角的MINA和Netty。

## 2.1 BIO编程模式

### 2.1.1 BIO单线程示例

最简单的SocketServer编程，服务端单线程等待客户端连接，在一个客户端与服务端连接后，只要当前客户端没有端口连接，其他客户端是无法接入的。

服务端：

```java
    /**
     * 演示单个线程处理客户端连接
     */
    @Test
    public void singleThreadServer() {
        ServerSocket socketServer = null;
        try {
            // bind绑定监听端口
            socketServer = new ServerSocket(DEFAULT_PORT);
            System.out.println("启动服务器，监听端口"+DEFAULT_PORT);

            while (true) {
                // 等待客户端连接
                Socket socket = socketServer.accept();
                System.out.println("客户端["+socket.toString()+"]已连接");

                // BIO处理数据
                BufferedReader reader = new BufferedReader(
                        new InputStreamReader(socket.getInputStream()));
                BufferedWriter writer = new BufferedWriter(
                        new OutputStreamWriter(socket.getOutputStream()));

                // 读取客户端消息，发送的消息中有行分隔符时，才能用readLine()接收。
                // 不然就都是一行了
                String msg = null;
                while ((msg = reader.readLine())!=null) {
                    System.out.println("客户端["+socket.toString()+"]:"+msg);

                    // 回复消息
                    writer.write("服务器："+ msg + "\n");
                    writer.flush();

                    //查看客户端是否退出
                    if(Client.QUIT.equals(msg)){
                        System.out.println("客户端["+socket.toString()+"]已退出");
                        break;
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (socketServer!=null) {
                try {
                    socketServer.close();
                    System.out.println("关闭server");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

客户端：

```java
 public class Client {
    public static final String QUIT="quit";
    public static final String DEFAULT_SERVER_HOST = "127.0.0.1";
    public static final int DEFAULT_SERVER_PORT=8888;

    @Test
    public void createClient(int i) {
        Socket socket = null;
        BufferedReader  reader = null;
        BufferedWriter writer = null;
        try {
            // 创建socket
            socket = new Socket(DEFAULT_SERVER_HOST,DEFAULT_SERVER_PORT);
            System.out.println("客户端["+socket+"]与服务器建立连接。");
            // 创建IO流
            // System.out.println("请输入信息：");
            reader = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            writer = new BufferedWriter(
                    new OutputStreamWriter(socket.getOutputStream()));

            // 等待用户输入信息
            BufferedReader consoleReader = new BufferedReader(
                    new InputStreamReader(System.in));
            // 一直发送消息，直到从屏幕接收到quit，才退出
            while (true) {
                String input = consoleReader.readLine();

                // 发送消息给服务器
                writer.write(input+"\n");
                writer.flush();

                // 读取服务器返回消息
                System.out.println(reader.readLine());

                // 查看用户是否退出
                if (QUIT.equals(input)) {
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }  finally {
            //这里关闭writer的话，会自动关闭它封装的OutputStreamWriter 和socket
            if (writer!=null) {
                try {
                    writer.close();
                    System.out.println("关闭client");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

运行结果：这里多次运行client，因第一个client没输入quit，第2个client是无法连接的。

服务器：

```shell
启动服务器，监听端口8888
客户端[Socket[addr=/127.0.0.1,port=1361,localport=8888]]已连接
客户端[Socket[addr=/127.0.0.1,port=1361,localport=8888]]:测试1
客户端[Socket[addr=/127.0.0.1,port=1361,localport=8888]]:测试2
客户端[Socket[addr=/127.0.0.1,port=1361,localport=8888]]:quit
客户端[Socket[addr=/127.0.0.1,port=1361,localport=8888]]已退出
客户端[Socket[addr=/127.0.0.1,port=1388,localport=8888]]已连接
客户端[Socket[addr=/127.0.0.1,port=1388,localport=8888]]:测1
```

客户端1：

```shell
客户端[Socket[addr=/127.0.0.1,port=8888,localport=1361]]与服务器建立连接。
测试1
服务器：测试1
测试2
服务器：测试2
quit
服务器：quit
关闭client
```

客户端2：

```shell
客户端[Socket[addr=/127.0.0.1,port=8888,localport=1388]]与服务器建立连接。
测1
服务器：测1
```

### 2.1.2 BIO多线程

单线程处理的效率比较第，在服务端引入多线程，针对每个client在服务端创建单独的线程处理，这样处理效率会高很多。

![io-019](..\images\io-019.png)

### 2.1.3 示例

多人聊天室：

基于BIO模型

支持多人同时在线

每个用户的发言都被转发给其他用户

```java
/**
 * BIO实现多人聊天室,功能
 * （1）基于BIO模型
 * （2）支持多人同时在线
 * （3）每个用户的发言都被转发给其他用户
 */
public class BIOChatServer {
    public static final int DEFAULT_PORT = 8888;
    public static final String DEFAULT_HOST = "127.0.0.1";
    public static final String QUIT ="quit";

    private ServerSocket serverSocket;
    private Map<Integer, Writer> connectedClients = new HashMap<>();

    /**
     * 新接入客户端，并将客户端加入到连接Map中
     * 多个线程客户端调用，需要线程安全
     * @param socket
     * @throws IOException
     */
    private synchronized void addClient(Socket socket) throws IOException {
        if (socket!=null) {
            int port = socket.getPort();
            BufferedWriter writer = new BufferedWriter(
                    new OutputStreamWriter(socket.getOutputStream()));
            connectedClients.put(port,writer);
            System.out.println("客户端["+socket.getInetAddress()+socket.getPort()+"]已连接到服务器");
        }
    }

    /**
     * 关闭客户端，并将客户端从连接Map中清除
     * 多个线程客户端调用，需要线程安全
     * @param socket
     * @throws IOException
     */
    private synchronized void removeClient(Socket socket) throws IOException {
        if (socket!=null) {
            int port = socket.getPort();
            if (connectedClients.containsKey(port)) {
                // socket是通过writer进行封装的，所以关闭writer就会关闭socket
                connectedClients.get(port).close();
                connectedClients.remove(port);
                System.out.println("客户端["+socket.getInetAddress()+socket.getPort()+"]已断开连接");
            }
        }
    }

    /**
     * 将客户端socket发生来的消息，转发到其他已连接的所有客户端
     * @param socket
     * @param fwdMsg
     */
    private synchronized void forwardMessage(Socket socket,String fwdMsg) {
        connectedClients.forEach((port,writer)->{
            if (port!=socket.getPort()) {
                try {
                    writer.write(fwdMsg);
                    writer.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    /**
     * 启动服务器
     */
    public void start() {
        try {
            // 绑定监听端口
            serverSocket = new ServerSocket(DEFAULT_PORT);
            System.out.println("启动服务器，监听端口："+DEFAULT_PORT);

            while (true){
                // 等待客户端连接
                Socket socket = serverSocket.accept();
                // 创建ChatHandler线程
                new Thread(new ChatHandler(socket,this)).start();
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    public synchronized void close() {
        if (serverSocket!=null) {
            try {
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 检查用户是否退出
     * @param msg
     * @return
     */
    public boolean readyToQuit(String msg) {
        return QUIT.equals(msg);
    }

    /**
     * 每个客户端连接，建立一个线程来处理
     */
    private static class ChatHandler implements Runnable{
        private Socket socket;

        private BIOChatServer server;

        public ChatHandler(Socket socket, BIOChatServer server) {
            this.socket = socket;
            this.server = server;
        }

        @Override
        public void run() {
            try {
                // 保存新连接用户
                server.addClient(socket);

                //读取用户发送的消息
                BufferedReader reader = new BufferedReader(
                        new InputStreamReader(socket.getInputStream()));

                while(true) {
                    String msg=reader.readLine();
                    String fwdMsg = "客户端["+socket.getInetAddress()+socket.getPort()+"]:"+msg;
                    System.out.println(fwdMsg);

                    // 将消息转发给聊天室里在线的其他用户
                    // 添加 \n 方便接收方readline方法处理
                    server.forwardMessage(socket,fwdMsg+"\n");

                    // 检查用户是否准备退出
                    if (server.readyToQuit(msg)) {
                        break;
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    server.removeClient(socket);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        BIOChatServer server = new BIOChatServer();
        server.start();
    }
}
```

```java
/**
 * BIO实现多人聊天室,功能
 * （1）基于BIO模型
 * （2）支持多人同时在线
 * （3）每个用户的发言都被转发给其他用户
 *
 * 这里因为使用consoleReader.readLine();不能使用JUint调试
 * 只能使用main方法，否则会到consoleReader.readLine();无反应卡死
 */
public class BIOChatClient {

    private Socket socket;
    private BufferedReader reader;
    private BufferedWriter writer;

    /**
     * 发送消息到服务端
     * @param msg
     * @throws IOException
     */
    private void send(String msg) throws IOException {
        if (!socket.isOutputShutdown()) {
            writer.write(msg+"\n");
            writer.flush();
        }
    }

    private String receive() throws IOException {
        return socket.isInputShutdown()?null:reader.readLine();
    }

    /**
     * 检查用户是否退出
     * @param msg
     * @return
     */
    public boolean readyToQuit(String msg) {
        return BIOChatServer.QUIT.equals(msg);
    }

    public void close() {
        System.out.println("关闭socket");
        if (writer!=null) {
            try {
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (reader!=null) {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public void start() {
        try {
            // 创建socket
            socket = new Socket(BIOChatServer.DEFAULT_HOST,
                    BIOChatServer.DEFAULT_PORT);

            //创建IO流
            reader = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));

            writer = new BufferedWriter(
                    new OutputStreamWriter(socket.getOutputStream()));

            //处理用户输入，这里需要开启另外的线程
            //因为等待用户输入时阻塞式的，如果不开启额外的线程，会导致读数据被阻塞，无法接受消息
            new Thread(new UserInputHandler(this)).start();

            //读取服务器转发的消息
            String msg = null;
            while ((msg = receive())!=null) {
                System.out.println(msg);
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    /**
     * 用户输入处理线程
     * 单独开启线程是因为读取控制台输入，是会造成阻塞的。
     * 防止阻塞从服务器获取消息。
     */
    private static class UserInputHandler implements Runnable{

        private BIOChatClient chatClient;

        public UserInputHandler(BIOChatClient chatClient) {
            this.chatClient = chatClient;
        }


        @Override
        public void run() {
            try {
                System.out.println("请输入:");
                // 等待用户输入消息
                BufferedReader consoleReader =
                        new BufferedReader(
                                new InputStreamReader(System.in));

                while(true) {
                    //一般从控制台获取输入，是会有回车的，所以这里用readline
                    String input = consoleReader.readLine();
                    // 向服务器发送消息
                    chatClient.send(input);

                    // 检查用户是否准备推出
                    if (chatClient.readyToQuit(input)) {
                        break;
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                System.out.println("关闭");
            }
        }
    }

    public static void main(String[] args) {
        BIOChatClient client = new BIOChatClient();
        client.start();
    }
}
```





### 2.1.4 伪异步BIO多线程

为每个客户端创建连接线程是很消耗资源的事情，这里可以改造为使用线程池来实现，也就是可以理解是伪异步模式。

![io-020](..\images\io-020.png)

```java
private ExecutorService executorService= Executors.newFixedThreadPool(10);

    /**
     * 启动服务器
     */
    public void start() {
        try {
            // 绑定监听端口
            serverSocket = new ServerSocket(DEFAULT_PORT);
            System.out.println("启动服务器，监听端口："+DEFAULT_PORT);

            while (true){
                // 等待客户端连接
                Socket socket = serverSocket.accept();
//                // 创建ChatHandler线程
//                new Thread(new ChatHandler(socket,this)).start();

                // 使用线程池
                executorService.execute(new ChatHandler(socket,this));
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }
```

### 2.1.5 BIO中的阻塞

```java
//以下操作都会产生阻塞
ServerSocket.accept();
InputStream.read();
OutputSream.write();
//无法在同一个线程里处理多个Stream I/O
```

## 2.2 NIO编程模型

NIO即非阻塞IO。

* 使用Channel代替Stream
* 使用Selector监控多条Channel
* 可以在一个线程里处理多个Channel I/O

### 2.2.1 Channel与Buffer

在NIO中Channel的作用是替代IO中的Stream，且Channel不分输入和输出流。而对Channel的操作是无法离开Buffer的。

![io-021](..\images\io-021.png)

#### 2.2.1.1 向Buffer写入数据

![io-022](..\images\io-022.png)

![io-023](..\images\io-023.png)

调用flip()函数进入读模式

#### 2.2.1.2 读Buffer数据

![io-024](..\images\io-024.png)

这时有两种情况，

* **全部读完**

![io-025](..\images\io-025.png)

调用clear()函数，恢复到写模式

![io-022](..\images\io-022.png)

* 未全部读完的情况

![io-026](..\images\io-026.png)

调用compact()，会把上次未读完的数据（图中红色不分）拷贝到最开始的位置,恢复写模式

![io-027](..\images\io-027.png)

### 2.2.2 Channel基本操作

Channel间也可以进行数据交换

![io-028](..\images\io-028.png)

### 2.2.3 几个重要的Channel

* FileChannel
* ServerSocketChannel
* SocketChannel

### 2.2.4 Selector与Channel

Selector可以监视多个Channel的状态。首先将Channel注册到Selector中。

![io-029](..\images\io-029.png)

### 2.2.5 Channel的状态变化

这里主要就是关注 ServerSocketChannel 和 SocketChannel

`CONNECT   ,  ACCEPT  ， READ  ， WRITE`

Channel ----> Selector  ---> SelectionKey （interestOpes()，readyOps()，channel()，selector()，attachment()）

当Selector处理完一个Channel，并不会改变Channel状态，需要我们手动处理。