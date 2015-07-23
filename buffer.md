# 缓冲区

Java NIO 中的 Buffer 用于和 NIO 通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成 NIO Buffer 对象，并提供了一组方法，用来方便的访问该块内存。

下面是 NIO Buffer 相关的话题列表：

1. Buffer 的基本用法
2. Buffer 的 capacity,position 和 limit
3. Buffer 的类型
4. Buffer 的分配
5. 向 Buffer 中写数据
6. flip() 方法
7. 从 Buffer 中读取数据
8. clear() 与 compact() 方法
9. mark() 与 reset() 方法
10. equals() 与 compareTo() 方法

### Buffer 的基本用法

使用 Buffer 读写数据一般遵循以下四个步骤：

1. 写入数据到 Buffer
2. 调用flip()方法
3. 从 Buffer 中读取数据
4. 调用clear()方法或者compact()方法

当向 buffer 写入数据时，buffer 会记录下写了多少数据。一旦要读取数据，需要通过 flip() 方法将 Buffer 从写模式切换到读模式。在读模式下，可以读取之前写入到 buffer 的所有数据。

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用 clear() 或 compact() 方法。clear() 方法会清空整个缓冲区。compact() 方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

下面是一个使用 Buffer 的例子：

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();  
//create buffer with capacity of 48 bytes  
ByteBuffer buf = ByteBuffer.allocate(48);  
int bytesRead = inChannel.read(buf); //read into buffer.  
while (bytesRead != -1) {  
  buf.flip();  //make buffer ready for read  
  while(buf.hasRemaining()){  
      System.out.print((char) buf.get()); // read 1 byte at a time  
  }  
  buf.clear(); //make buffer ready for writing  
  bytesRead = inChannel.read(buf);  
}  
aFile.close();  
```

### Buffer 的 capacity,position 和 limit

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成 NIO Buffer 对象，并提供了一组方法，用来方便的访问该块内存。

为了理解 Buffer 的工作原理，需要熟悉它的三个属性：

- capacity
- position
- limit

position 和 limit 的含义取决于 Buffer 处在读模式还是写模式。不管 Buffer 处在什么模式，capacity 的含义总是一样的。

这里有一个关于 capacity，position 和 limit 在读写模式中的说明，详细的解释在插图后面。

![](/images/4.png)

**capacity**

作为一个内存块，Buffer 有一个固定的大小值，也叫 “capacity”。 你只能往里写 capacity 个 byte、long，char 等类型。一旦 Buffer 满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

**position**

当你写数据到 Buffer 中时，position 表示当前的位置。初始的 position 值为 0. 当一个 byte、long 等数据写到 Buffer 后， position 会向前移动到下一个可插入数据的 Buffer 单元。position 最大可为 capacity – 1。

当读取数据时，也是从某个特定位置读。当将 Buffer 从写模式切换到读模式，position 会被重置为 0。当从 Buffer 的 position 处读取数据时，position 向前移动到下一个可读的位置。

**limit**

在写模式下，Buffer 的 limit 表示你最多能往 Buffer 里写多少数据。 写模式下，limit 等于 Buffer 的 capacity。

当切换 Buffer 到读模式时，limit 表示你最多能读到多少数据。因此，当切换 Buffer 到读模式时，limit 会被设置成写模式下的 position 值。换句话说，你能读到之前写入的所有数据（limit 被设置成已写数据的数量，这个值在写模式下就是 position）

### Buffer 的类型

Java NIO 有以下 Buffer 类型  

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
 

如你所见，这些 Buffer 类型代表了不同的数据类型。换句话说，就是可以通过 char，short，int，long，float 或 double 类型来操作缓冲区中的字节。  

MappedByteBuffer 有些特别，在涉及它的专门章节中再讲。

### Buffer 的分配

要想获得一个 Buffer 对象首先要进行分配。 每一个 Buffer 类都有一个 allocate 方法。下面是一个分配 48 字节 capacity 的 ByteBuffer 的例子。

`ByteBuffer buf = ByteBuffer.allocate(48); ` 

这是分配一个可存储 1024 个字符的 CharBuffer：

`CharBuffer buf = CharBuffer.allocate(1024);`

### 向 Buffer 中写数据

写数据到 Buffer 有两种方式：

- 从 Channel 写到 Buffer。
- 通过 Buffer 的 put() 方法写到 Buffer 里。  

从 Channel 写到 Buffer 的例子

`int bytesRead = inChannel.read(buf); //read into buffer.`

通过 put 方法写 Buffer 的例子：

`buf.put(127);`

put 方法有很多版本，允许你以不同的方式把数据写入到 Buffer 中。例如， 写到一个指定的位置，或者把一个字节数组写入到 Buffer。 更多 Buffer 实现的细节参考 JavaDoc。

### flip() 方法

flip 方法将 Buffer 从写模式切换到读模式。调用 flip() 方法会将 position 设回 0，并将 limit 设置成之前 position 的值。

换句话说，position 现在用于标记读的位置，limit 表示之前写进了多少个 byte、char 等 —— 现在能读取多少个 byte、char 等。

### 从 Buffer 中读取数据

从 Buffer 中读取数据有两种方式：

- 从 Buffer 读取数据到 Channel。
- 使用 get() 方法从 Buffer 中读取数据。

从 Buffer 读取数据到 Channel 的例子：

```
//read from buffer into channel.  
int bytesWritten = inChannel.write(buf);
```

使用 get() 方法从 Buffer 中读取数据的例子

`byte aByte = buf.get();`

get 方法有很多版本，允许你以不同的方式从 Buffer 中读取数据。例如，从指定 position 读取，或者从 Buffer 中读取数据到字节数组。更多 Buffer 实现的细节参考 JavaDoc。

### rewind() 方法

Buffer.rewind() 将 position 设回 0，所以你可以重读 Buffer 中的所有数据。limit 保持不变，仍然表示能从 Buffer 中读取多少个元素（byte、char 等）。

### clear() 与 compact() 方法

一旦读完 Buffer 中的数据，需要让 Buffer 准备好再次被写入。可以通过 clear() 或 compact() 方法来完成。

如果调用的是 clear() 方法，position 将被设回 0，limit 被设置成 capacity 的值。换句话说，Buffer 被清空了。Buffer 中的数据并未清除，只是这些标记告诉我们可以从哪里开始往 Buffer 里写数据。

如果 Buffer 中有一些未读的数据，调用 clear() 方法，数据将 “被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果 Buffer 中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用 compact() 方法。

compact() 方法将所有未读的数据拷贝到 Buffer 起始处。然后将 position 设到最后一个未读元素正后面。limit 属性依然像 clear() 方法一样，设置成 capacity。现在 Buffer 准备好写数据了，但是不会覆盖未读的数据。

### mark() 与 reset() 方法

通过调用 Buffer.mark() 方法，可以标记 Buffer 中的一个特定 position。之后可以通过调用 Buffer.reset() 方法恢复到这个 position。例如：

```
buffer.mark();
//call buffer.get() a couple of times, e.g. during parsing.
buffer.reset();  //set position back to mark.
```

### equals() 与 compareTo() 方法

可以使用 equals() 和 compareTo() 方法两个 Buffer。

### equals()

当满足下列条件时，表示两个 Buffer 相等：

1. 有相同的类型（byte、char、int 等）。
2. Buffer 中剩余的 byte、char 等的个数相等。
3. Buffer 中所有剩余的 byte、char 等都相同。

如你所见，equals 只是比较 Buffer 的一部分，不是每一个在它里面的元素都比较。实际上，它只比较 Buffer 中的剩余元素。

### compareTo() 方法

compareTo() 方法比较两个 Buffer 的剩余元素 (byte、char 等)， 如果满足下列条件，则认为一个 Buffer“小于” 另一个 Buffer：

1. 第一个不相等的元素小于另一个 Buffer 中对应的元素 。
2. 所有元素都相等，但第一个 Buffer 比另一个先耗尽 (第一个 Buffer 的元素个数比另一个少)。

（译注：剩余元素是从 position 到 limit 之间的元素）





