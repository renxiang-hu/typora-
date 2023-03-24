## NIO开发文档

### 一、NIO 概况

`NIO`（Non-blocking I/O）是 `Java` 提供的一种非阻塞` I/O` 操作方式，它可以让一个线程来管理多个` I/O` 操作，从而提高系统的并发能力和处理能力。下面分别介绍 `NIO` 服务端和客户端的执行流程。

### 二、NIO三大核心原理

#### 1、Buffer缓冲区

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成`NIO Buffer`对象，并提供了 一组方法，用来方便的访问该块内存。相比较直接对数组的操作，`Buffer API`更加容易操作和管理。

##### 1.1、Buffer重要概念

* 容量(`capacity`)：作为一个内存块，`Buffer`具有一定的固定大小，也称为”容量”，缓冲区容量不能
  为负，并且创建后不能更改。
* 限制(`limit`)：表示缓冲区中可以操作数据的大小(`limit`后数据不能进行读写)。缓冲区的限制不
  能为负，并且不能大于其容量。写入模式，限制等于buffer的容量。读取模式下，`limit`等于写入
  的数据量。
* 位置(`position`)：下一个要读取或写入的数据的索引。缓’中区的位置不能为负，并且不能大于其限
  制。
* 标记(`mark`)与重置(`reset`)：标记是一个索弓l，通过`Buffer`中的`mark()`方法指定Buffer中一
  个特定的 position，之后可以通过调用reset(）方法恢复到这个`position`。

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

`Java NIO`的通道类似流，但又有些不同：既可以从通道中读取数据，又可以写数据到通道。但流的`input`或 `output`读写通常是单向的。通道可以非阻塞读取和写入通道，通道可以支持读取或写入缓冲区，也支持异步地读写。

#### 3、Selector选择器

`Selector`是一个`Java NIO`组件，可以能够检查一个或多个`NIO`通道，并确定哪些通道已经准备好进行读取或写入。这样，一个单独的线程可以管理多个`channel`，从而管理多个网络连接，提高效率。

### NIO 服务端执行流程

1. 创建 `Selector` 对象：创建一个 `Selector` 对象来管理所有的 `Channel`。
2. 创建 `ServerSocketChannel`：通过 `ServerSocketChannel.open()` 方法创建一个 `ServerSocketChannel` 对象，用来监听客户端的连接请求。
3. 绑定端口：调用 `ServerSocketChannel.bind()` 方法，将 `ServerSocketChannel` 绑定到一个 `IP` 地址和端口号。
4. 配置非阻塞模式：调用 `ServerSocketChannel.configureBlocking(false)` 方法，将 `ServerSocketChannel` 设置为非阻塞模式。
5. 注册 `Selector`：将 `ServerSocketChannel` 注册到 `Selector` 对象中，并指定 `SelectionKey.OP_ACCEPT` 操作位，表示可以接收客户端的连接请求。
6. 处理连接请求：在一个死循环中调用 `Selector.select()` 方法来等待客户端的连接请求。当 `Selector.select()` 方法返回值大于 0 时，表示有客户端连接请求到来，可以通过 `SelectionKey` 获取对应的 `Channel`，并将该 `Channel` 注册到 `Selector` 中，以便后续处理读写操作。
7. 处理读写操作：在一个死循环中调用 `Selector.select()` 方法来等待 `Channel` 可读/可写事件。当 `Selector.select()` 方法返回值大于 0 时，表示有 `Channel` 可读/可写，可以通过 `SelectionKey` 获取对应的 `Channel`，并处理相应的读写操作。

### NIO 客户端执行流程

1. 创建 `SocketChannel` 对象：通过 `SocketChannel.open()` 方法创建一个 `SocketChannel` 对象。
2. 配置非阻塞模式：调用 `SocketChannel.configureBlocking(false)` 方法，将 `SocketChannel` 设置为非阻塞模式。
3. 连接服务器：调用 `SocketChannel.connect()` 方法连接服务器。由于设置为非阻塞模式，连接操作不会一直阻塞等待连接成功，而是会立即返回。
4. 判断连接是否成功：在一个死循环中调用 `SocketChannel.finishConnect()` 方法来判断连接是否成功。当该方法返回 `true` 时，表示连接成功，可以开始进行读写操作。
5. 处理读写操作：在一个死循环中调用 `Selector.select()` 方法来等待 `Channel` 可读/可写事件。当 `Selector.select()` 方法返回值大于 0 时，表示有 `Channel` 可读/可写，可以通过 `SelectionKey` 获取对应的 `Channel`，并处理相应的读写操作。

总体来说，`NIO` 的执行流程比较复杂，需要对多个对象进行管理，包括 `Selector`、`Channel` 和 `SelectionKey` 等，需要掌握相应的 API 和使用方法。但是，`NIO` 的优点在于它能够高效地处理大量的并发连接，提高系统的性能和稳定性。