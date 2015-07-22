# SocketChannel

Java NIO 中的 SocketChannel 是一个连接到 TCP 网络套接字的通道。可以通过以下 2 种方式创建 SocketChannel：

1. 打开一个 SocketChannel 并连接到互联网上的某台服务器。
2. 一个新连接到达 ServerSocketChannel 时，会创建一个 SocketChannel。

### 打开 SocketChannel

下面是 SocketChannel 的打开方式：

```
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

### 关闭 SocketChannel

当用完 SocketChannel 之后调用 SocketChannel.close() 关闭 SocketChannel：

`socketChannel.close();`

### 从 SocketChannel 读取数据

要从 SocketChannel 中读取数据，调用一个 read() 的方法之一。以下是例子：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);
```

首先，分配一个 Buffer。从 SocketChannel 读取到的数据将会放到这个 Buffer 中。

然后，调用 SocketChannel.read()。该方法将数据从 SocketChannel 读到 Buffer 中。read() 方法返回的 int 值表示读了多少字节进 Buffer 里。如果返回的是 - 1，表示已经读到了流的末尾（连接关闭了）。

### 写入 SocketChannel

写数据到 SocketChannel 用的是 SocketChannel.write() 方法，该方法以一个 Buffer 作为参数。示例如下：

```
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
while(buf.hasRemaining()) {
    channel.write(buf);
}
```

注意 SocketChannel.write() 方法的调用是在一个 while 循环中的。Write() 方法无法保证能写多少字节到 SocketChannel。所以，我们重复调用 write() 直到 Buffer 没有要写的字节为止。

### 非阻塞模式

可以设置 SocketChannel 为非阻塞模式（non-blocking mode）. 设置之后，就可以在异步模式下调用 connect()， read() 和 write() 了。

**connect()**

如果 SocketChannel 在非阻塞模式下，此时调用 connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用 finishConnect() 的方法。像这样：

```
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}
```

**write()**

非阻塞模式下，write() 方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用 write()。前面已经有例子了，这里就不赘述了。

**read()**

非阻塞模式下, read() 方法在尚未读取到任何数据时可能就返回了。所以需要关注它的 int 返回值，它会告诉你读取了多少字节。

### 非阻塞模式与选择器

非阻塞模式与选择器搭配会工作的更好，通过将一或多个 SocketChannel 注册到 Selector，可以询问选择器哪个通道已经准备好了读取，写入等。Selector 与 SocketChannel 的搭配使用会在后面详讲。




