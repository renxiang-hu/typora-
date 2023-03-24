## NIO开发文档

### 一、NIO 概况

`NIO`（Non-blocking I/O）是 `Java` 提供的一种非阻塞` I/O` 操作方式，它可以让一个线程来管理多个` I/O` 操作，从而提高系统的并发能力和处理能力。下面分别介绍 `NIO` 服务端和客户端的执行流程。

### 二、NIO三大核心原理

#### 1、Buffer缓冲区

<img src="/Users/hurenxiang/Desktop/Typora文档中心/typora-document/pic/image-20230324152914034.png" alt="image-20230324152914034" style="zoom:50%;" />

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成`NIO Buffer`对象，并提供了 一组方法，用来方便的访问该块内存。相比较直接对数组的操作，`Buffer API`更加容易操作和管理。

##### 1.1、Buffer重要概念

* 容量(`capacity`)：作为一个内存块，`Buffer`具有一定的固定大小，也称为”容量”，缓冲区容量不能
  为负，并且创建后不能更改。
* 限制(`limit`)：表示缓冲区中可以操作数据的大小(`limit`后数据不能进行读写)。缓冲区的限制不
  能为负，并且不能大于其容量。写入模式，限制等于buffer的容量。读取模式下，`limit`等于写入
  的数据量。
* 位置(`position`)：下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限
  制。
* 标记(`mark`)与重置(`reset`)：标记是一个索弓l，通过`Buffer`中的`mark()`方法指定Buffer中一
  个特定的 `position`，之后可以通过调用`reset()`方法恢复到这个`position`。

##### 1.2、Buffer常见方法

```java
Buffer clear()          清空缓冲区并返回对缓冲区的引用
Buffer flip()           为将缓冲区的界限设置为当前位置，并将当前位置重置为0
int capacity()          返回Buffer的capacity大小
boolean hasRemaining()  判断缓冲区中是否还有元素
int limit()             返回Buffer的界限（limit）的位置
Buffer limit(int n）    将设置缓冲区界限为n，并返回一个具有新limit的缓冲区对象
Buffer mark()           对缓冲区设置标记
int position()          返回缓冲区的当前位置position
Buffer position(int n） 将设置缓冲区的当前位置为n，并返回修改后的Buffer对象
int remaining()         返回position和limit之间的元素个数
Buffer reset()          将位置position转到以前设置的mark所在的位置
Buffer rewind()         将位置设为为0．取消设置的mark
```

#### 2、Channel通道

通道(`Channel`)：由`java.nio.channels`包定义的。`Channel`表示`IO`源与目标打开的连接。`Channel`类似于传统的“流”。只不过`Channel`本身不能直接访问数据，`Channel`只能与`Buffer`进行交互。

1. NIO的通道类似于流，但有些区别如下

* 通道可以同时进行读写，而流只能读或者只能写；
* 通道可以实现异步读写数据；
* 通道可以从缓冲读数据，也可以写数据到缓冲；

2. `BlO`中的`stream`是单向的，例如`FilelnputStream`对象只能进行读取数据的操作，而`NIO`中的通道(`Channel`)是双向的，可以读操作，也可以写操作。

#### 3、Selector选择器

选择器(`Selector`)是`SeIectabIeChannIe`对象的多路复用器，`Selector`可以同时监控多个`SelectableChannel` 的`IO`状况，也就是说，利用`Selector`可使一个单独的线程管理多个`Channel`。

### 三、落地实践

#### 3.1、服务端接收客户端的连接请求(一对一)

##### NIO 服务端执行流程

1. 创建 `Selector` 对象：创建一个 `Selector` 对象来管理所有的 `Channel`。
2. 创建 `ServerSocketChannel`：通过 `ServerSocketChannel.open()` 方法创建一个 `ServerSocketChannel` 对象，用来监听客户端的连接请求。
3. 绑定端口：调用 `ServerSocketChannel.bind()` 方法，将 `ServerSocketChannel` 绑定到一个 `IP` 地址和端口号。
4. 配置非阻塞模式：调用 `ServerSocketChannel.configureBlocking(false)` 方法，将 `ServerSocketChannel` 设置为非阻塞模式。
5. 注册 `Selector`：将 `ServerSocketChannel` 注册到 `Selector` 对象中，并指定 `SelectionKey.OP_ACCEPT` 操作位，表示可以接收客户端的连接请求。
6. 处理连接请求：在一个死循环中调用 `Selector.select()` 方法来等待客户端的连接请求。当 `Selector.select()` 方法返回值大于 0 时，表示有客户端连接请求到来，可以通过 `SelectionKey` 获取对应的 `Channel`，并将该 `Channel` 注册到 `Selector` 中，以便后续处理读写操作。
7. 处理读写操作：在一个死循环中调用 `Selector.select()` 方法来等待 `Channel` 可读/可写事件。当 `Selector.select()` 方法返回值大于 0 时，表示有 `Channel` 可读/可写，可以通过 `SelectionKey` 获取对应的 `Channel`，并处理相应的读写操作。

```java
/**
 * 目标：NIO非阻塞通信下的入门案例
 */
public class Server {
    public static void main(String[] args) throws IOException {
        //获取通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //切换非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定连接
        serverSocketChannel.bind(new InetSocketAddress(9999));
        //获取选择器
        Selector selector = Selector.open();
        //将通道注册到选择器上，并且指定"监听接收事件"
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        //轮询式的获取选择器上已经"准备就绪"的事件
        while (selector.select() > 0) {
            System.out.println("轮一轮");
            //获取当前选择器中所有注册的"选择键(已就绪的监听事件)"）
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()){
                //获取准备"就绪"的事件
                SelectionKey next = iterator.next();
                //判断具体是什么事件准备就绪
                if (next.isAcceptable()){
                    //若"接收就绪"，获取客户端连接
                    SocketChannel accept = serverSocketChannel.accept();
                    //切换非阻塞模式
                    accept.configureBlocking(false);
                    //将通道注册到选择器上
                    accept.register(selector,SelectionKey.OP_READ);
                } else if (next.isReadable()){
                    //获取当前选择器上"读就绪"状态的通道
                    SocketChannel channel = (SocketChannel) next.channel();
                    //读取数据
                    ByteBuffer allocate = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = channel.read(allocate)) > 0) {
                        allocate.flip();
                        System.out.println(new String(allocate.array(),0,len));
                        allocate.clear();
                    }
                }
                iterator.remove();
            }
        }
    }
}
```

##### NIO 客户端执行流程

1. 创建 `SocketChannel` 对象：通过 `SocketChannel.open()` 方法创建一个 `SocketChannel` 对象。
2. 配置非阻塞模式：调用 `SocketChannel.configureBlocking(false)` 方法，将 `SocketChannel` 设置为非阻塞模式。
3. 连接服务器：调用 `SocketChannel.connect()` 方法连接服务器。由于设置为非阻塞模式，连接操作不会一直阻塞等待连接成功，而是会立即返回。
4. 判断连接是否成功：在一个死循环中调用 `SocketChannel.finishConnect()` 方法来判断连接是否成功。当该方法返回 `true` 时，表示连接成功，可以开始进行读写操作。
5. 处理读写操作：在一个死循环中调用 `Selector.select()` 方法来等待 `Channel` 可读/可写事件。当 `Selector.select()` 方法返回值大于 0 时，表示有 `Channel` 可读/可写，可以通过 `SelectionKey` 获取对应的 `Channel`，并处理相应的读写操作。

```java
public class Client {
    public static void main(String[] args) throws Exception {
        //获取通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9999));
        //切换非阻塞模式
        socketChannel.configureBlocking(false);
        //分配指定大小的缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        //发送数据给服务端
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String str = scanner.nextLine();
            byteBuffer.put((new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(System.currentTimeMillis())+"  "+str).getBytes());
            byteBuffer.flip();
            socketChannel.write(byteBuffer);
            byteBuffer.clear();
        }
        socketChannel.close();
    }
}
```

总体来说，`NIO` 的执行流程比较复杂，需要对多个对象进行管理，包括 `Selector`、`Channel` 和 `SelectionKey` 等，需要掌握相应的 API 和使用方法。但是，`NIO` 的优点在于它能够高效地处理大量的并发连接，提高系统的性能和稳定性。

#### 3.2、服务端接收客户端的连接请求(一对多)

**服务端代码**

```java
public class Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务端启动~~~");
        //获取通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //切换为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定连接端口
        serverSocketChannel.bind(new InetSocketAddress(9999));
        //获取选择器
        Selector selector = Selector.open();
        //将通道注册到选择器上，并且开始指定监听接收事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        //使用selector选择器轮询已经就绪好的事件
        while (selector.select() > 0) {
            System.out.println("开始一轮事件处理~");
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            //开始遍历准备好的事件
            while (iterator.hasNext()){
                //提取当前事件
                SelectionKey selectionKey = iterator.next();
                if (selectionKey.isAcceptable()){
                    //直接获取当前接入的客户端通道
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    //将客户端的通道设置为非阻塞的
                    socketChannel.configureBlocking(false);
                    //将客户端的通道注册到selector上
                    socketChannel.register(selector,SelectionKey.OP_READ);
                } else if (selectionKey.isReadable()){
                    //获取当前选择器上的"读就绪事件"
                    SocketChannel socketChannel = (SocketChannel)selectionKey.channel();
                    //开始读取数据
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = socketChannel.read(byteBuffer)) > 0){
                        byteBuffer.flip();
                        System.out.println(new String(byteBuffer.array(),0,len));
                        byteBuffer.clear();
                    }
                }
                //处理完毕当前事件后，需要移除当前事件，否则会重复处理
                iterator.remove();
            }
        }
    }
}
```

**客户端代码**

```java
public class Client {
    public static void main(String[] args) throws Exception {
        //获取通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1",9999));
        //切换为非阻塞模式
        socketChannel.configureBlocking(false);
        //分配指定缓存大小
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        //发送数据给服务器端
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("请输入：");
            String s = scanner.nextLine();
            byteBuffer.put(s.getBytes());
            byteBuffer.flip();
            socketChannel.write(byteBuffer);
            byteBuffer.clear();
        }
    }
}
```

### 四、答疑

#### 4.1、serverSocketChannel为什么要切换为非阻塞模式

`ServerSocketChannel` 切换为非阻塞模式的主要原因是为了提高服务器的性能和可伸缩性。

在阻塞模式下，当 `ServerSocketChannel` 调用 `accept()` 方法时，如果没有客户端连接到服务器，则 `accept()` 方法将一直阻塞等待，直到有连接到来或者发生异常才会返回。这种阻塞式的等待会浪费服务器的资源，降低服务器的响应速度和并发处理能力。

而在非阻塞模式下，`ServerSocketChannel` 的 `accept()` 方法会立即返回，无论是否有客户端连接到服务器。如果没有连接到来，则 `accept()` 方法将返回 `null`。这种非阻塞的方式允许服务器在等待连接的同时，可以处理其他任务，从而提高了服务器的并发处理能力和响应速度。

另外，非阻塞模式还可以实现多路复用，即在单线程中同时处理多个连接。这种方式可以减少线程的创建和销毁，节省服务器资源，提高服务器的可伸缩性。

