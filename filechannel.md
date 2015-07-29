# FileChannel

Java NIO 中的 FileChannel 是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel 无法设置为非阻塞模式，它总是运行在阻塞模式下。

### 打开 FileChannel

在使用 FileChannel 之前，必须先打开它。但是，我们无法直接打开一个 FileChannel，需要通过使用一个 InputStream、OutputStream 或 RandomAccessFile 来获取一个 FileChannel 实例。下面是通过 RandomAccessFile 打开 FileChannel 的示例：

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```

### 从 FileChannel 读取数据

调用多个 read() 方法之一从 FileChannel 中读取数据。如：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

首先，分配一个 Buffer。从 FileChannel 中读取的数据将被读到 Buffer 中。

然后，调用 FileChannel.read() 方法。该方法将数据从 FileChannel 读取到 Buffer 中。read() 方法返回的 int 值表示了有多少字节被读到了 Buffer 中。如果返回 - 1，表示到了文件末尾。

### 向 FileChannel 写数据

使用 FileChannel.write() 方法向 FileChannel 写数据，该方法的参数是一个 Buffer。如：

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

注意 FileChannel.write() 是在 while 循环中调用的。因为无法保证 write() 方法一次能向 FileChannel 写入多少字节，因此需要重复调用 write() 方法，直到 Buffer 中已经没有尚未写入通道的字节。

### 关闭 FileChannel

用完 FileChannel 后必须将其关闭。如：

`channel.close();`

### FileChannel 的 position 方法

有时可能需要在 FileChannel 的某个特定位置进行数据的读 / 写操作。可以通过调用 position() 方法获取 FileChannel 的当前位置。

也可以通过调用 position(long pos) 方法设置 FileChannel 的当前位置。

这里有两个例子:

```
long pos = channel.position();
channel.position(pos +123);
```

如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回 - 1 —— 文件结束标志。

如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致 “文件空洞”，磁盘上物理文件中写入的数据间有空隙。

### FileChannel 的 size 方法

FileChannel 实例的 size() 方法将返回该实例所关联文件的大小。如:

`long fileSize = channel.size();`

### FileChannel 的 truncate 方法

可以使用 FileChannel.truncate() 方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：

`channel.truncate(1024);`

这个例子截取文件的前 1024 个字节。

### FileChannel 的 force 方法

FileChannel.force() 方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到 FileChannel 里的数据一定会即时写到磁盘上。要保证这一点，需要调用 force() 方法。

force() 方法有一个 boolean 类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上：

`channel.force(true);`

