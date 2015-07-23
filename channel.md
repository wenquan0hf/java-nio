# 通道

Java NIO 的通道类似流，但又有些不同：

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个 Buffer，或者总是要从一个 Buffer 中写入。

正如上面所说，从通道读取数据到缓冲区，从缓冲区写入数据到通道。如下图所示：

![](/images/3.png)

### Channel 的实现

这些是 Java NIO 中最重要的通道的实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

FileChannel 从文件中读写数据。

DatagramChannel 能通过 UDP 读写网络中的数据。

SocketChannel 能通过 TCP 读写网络中的数据。

ServerSocketChannel 可以监听新进来的 TCP 连接，像 Web 服务器那样。对每一个新进来的连接都会创建一个 SocketChannel。

### 基本的 Channel 示例

下面是一个使用 FileChannel 读取数据到 Buffer 中的示例：

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");     
FileChannel inChannel = aFile.getChannel();  
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
System.out.println(&quot;Read &quot; + bytesRead);
buf.flip();
while(buf.hasRemaining()){
System.out.print((char) buf.get());
}
buf.clear();
bytesRead = inChannel.read(buf);
}
aFile.close();
```

注意 buf.flip() 的调用，首先读取数据到 Buffer，然后反转 Buffer, 接着再从 Buffer 中读取数据。下一节会深入讲解 Buffer 的更多细节。



