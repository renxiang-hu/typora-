## NIO开发文档

### NIO 概况

`NIO`（Non-blocking I/O）是 `Java` 提供的一种非阻塞` I/O` 操作方式，它可以让一个线程来管理多个` I/O` 操作，从而提高系统的并发能力和处理能力。下面分别介绍 `NIO` 服务端和客户端的执行流程。

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