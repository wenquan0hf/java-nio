# 通道之间的数据传输

在 Java NIO 中，如果两个通道中有一个是 FileChannel，那你可以直接将数据从一个 channel（译者注：channel 中文常译作通道）传输到另外一个 channel。

**transferFrom()**

FileChannel 的 transferFrom() 方法可以将数据从源通道传输到 FileChannel 中（译者注：这个方法在 JDK 文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();
long position = 0;
long count = fromChannel.size();
toChannel.transferFrom(position, count, fromChannel);
```

方法的输入参数 position 表示从 position 处开始向目标文件写入数据，count 表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。  

此外要注意，在 SoketChannel 的实现中，SocketChannel 只会传输此刻准备好的数据（可能不足 count 字节）。因此，SocketChannel 可能不会将请求的所有数据 (count 个字节) 全部传输到 FileChannel 中。

**transferTo()**

transferTo() 方法将数据从 FileChannel 传输到其他的 channel 中。下面是一个简单的例子：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();
long position = 0;
long count = fromChannel.size();
fromChannel.transferTo(position, count, toChannel);
```

是不是发现这个例子和前面那个例子特别相似？除了调用方法的 FileChannel 对象不一样外，其他的都一样。

上面所说的关于 SocketChannel 的问题在 transferTo() 方法中同样存在。SocketChannel 会一直传输数据直到目标 buffer 被填满。

