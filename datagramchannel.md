# DatagramChannel

Java NIO 中的 DatagramChannel 是一个能收发 UDP 包的通道。因为 UDP 是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

### 打开 DatagramChannel

下面是 DatagramChannel 的打开方式：

```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

这个例子打开的 DatagramChannel 可以在 UDP 端口 9999 上接收数据包。

### 接收数据

通过 receive() 方法从 DatagramChannel 接收数据，如：

```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```

receive() 方法会将接收到的数据包内容复制到指定的 Buffer. 如果 Buffer 容不下收到的数据，多出的数据将被丢弃。

### 发送数据

通过 send() 方法从 DatagramChannel 发送数据，如:

```
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

这个例子发送一串字符到”jenkov.com” 服务器的 UDP 端口 80。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为 UDP 在数据传送方面没有任何保证。

### 连接到特定的地址

可以将 DatagramChannel“连接” 到网络中的特定地址的。由于 UDP 是无连接的，连接到特定地址并不会像 TCP 通道那样创建一个真正的连接。而是锁住 DatagramChannel ，让其只能从特定地址收发数据。

这里有个例子:

`channel.connect(new InetSocketAddress("jenkov.com", 80));`

当连接后，也可以使用 read() 和 write() 方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子：

```
int bytesRead = channel.read(buf);
int bytesWritten = channel.write(but);
```

