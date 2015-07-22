# Pipe

Java NIO 管道是 2 个线程之间的单向数据连接。Pipe有一个 source 通道和一个 sink 通道。数据会被写到 sink 通道，从 source 通道读取。

这里是 Pipe 原理的图示：

![](/images/8.png)

### 创建管道

通过Pipe.open()方法打开管道。例如：

`Pipe pipe = Pipe.open();`

### 向管道写数据

要向管道写数据，需要访问 sink 通道。像这样：

`Pipe.SinkChannel sinkChannel = pipe.sink();`

通过调用 SinkChannel 的write()方法，将数据写入SinkChannel, 像这样：

```
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

### 从管道读取数据

从读取管道的数据，需要访问 source 通道，像这样：

`Pipe.SourceChannel sourceChannel = pipe.source();`

调用 source 通道的read()方法来读取数据，像这样：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```

read()方法返回的 int 值会告诉我们多少字节被读进了缓冲区。


