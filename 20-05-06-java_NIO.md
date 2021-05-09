# NIO

引用：https://ifeve.com/



Java NIO由一下集合核心部分构成:

- Channels
- Buffers
- Selectors

## Channels

Java NIO的通道类似流,但又有些不同:

- 既可以从通道读取数据,又可以写数据到通道.而流通常时单向的.
- 通道可以异步的读写 (?)
- 通道中的数据总是到先读到一个Buffer,或者是从一个Buffer中写入.

### Channel的重要实现:

- FileChannel 从文件中读写数据
- DatagramChannel 能通过UDP读写网络中的数据
- SocketChannel 能通过TCP读写网络中的数据
- ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel

### Channel使用示例:

```java
    public static void main(String[] args) throws IOException {
        String path = "D:\\ideaProject\\myspring\\src\\com\\riden\\spring5\\MainTest.java";
        RandomAccessFile file = new RandomAccessFile(path, "rw");
        // 获取管道
        FileChannel channel = file.getChannel();
        // 创建缓存
        ByteBuffer buf = ByteBuffer.allocate(40);
        int hasRead;
        while ((hasRead = channel.read(buf)) != -1) {
            // make buffer ready for read
            buf.flip();
            while (buf.hasRemaining()) {
                // read 1 byte at a time
                System.out.print((char) buf.get());
            }
            // make buf ready for writing
            buf.clear();
        }
    }
```

## Buffer

Java NIO中Buffer用于和NIO通道进行交互.数据是从管道读入缓存,从缓存写到通道中.

缓冲区本质是一块可以写入数据,然后可以从中读取数据的内存.

### Buffer的基本用法

使用Buffer读写数据一般遵循一下四个步骤:

1. 写入数据到Buffer
2. 调用flip()方法
3. 从Buffer中读取数据
4. 调用clear()方法活这compact方法

当向buffer写入数据时,buffer会记录写下了多少数据(通过position).如果要读取数据,需要调用flip()方法让Buffer从写模式切换到读模式.

一旦读完所有数据,需要清空缓冲区,让它可以再次被写入.有两种方式可以清空缓冲区:

1. 调用clear()方法: 改方法会清空整个缓冲区.
2. 调用compact()方法: 该方法只会清除已经读过的数据.

### Buffer的三个重要属性

上面说buffer就是一个块内存.这块内存被包装成NIO Buffer对象,并提供了一些方法,用来访问这块内内存.

为了理解Buffer的工作原理,需要熟悉它的三个属性:

- capacity: 固定值,代表这块内存的容量大小
- position: 代表这块内存已经使用(读或写)了多少
- limit: 内存可以被访问的最大区域值.

### Buffer的类型

JavaNIO有一下Buffer类型

- ByteBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
- CharBuffer
- MappedByteBuffer

### Buffer的分配

想要创建一个Buffer对象,通常调用Buffer对象的静态方法
`xxxBuffer allocate(int capacity)`,这里的capacity就是上文中Buffer三个重要属性中的capacity,代表这个缓冲块的容量.

### 向Buffer中写数据

写数据到Buffer有两种方式:

- 从Channel中写道Buffer
- 通过Buffer的put()方法写到Buffer里.例如`buf.put(128);`

### flip方法

flip方法将Buffer从写模式切换到读模式.它的实质是将limit设置为position的值,再将position设为0. 

flip方法源代码:

```java
    public Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```

### compact()方法

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

### 从Buffer中读取数据

从Buffer中读取数据有两种方式:

1. 从Buffer读取数据到Channel. 例如:`int hasRead = channel.write(buf)`
2. 使用get()方法从Buffer中读取数据. 例如:`byte val = buf.get()`

## Selector

Selector是Java NIO中能够检测一到多个NIO通道,并能够知晓通道是否为诸如读写事件做好准备的组件.这样一个单独的线程可一个管理多个channel.

### Selector优势

可以使用单线程处理多个Channel,减少线程的使用量.

### Selector的创建

通过调用Selector.open()方法创建一个Selector,如下:

```java
Selector selector = Selector.open();
```

向Selector注册通道.

为了将Channel和Selector配合使用,必须将channel注册到selector上.通过SelectableChannel.register()方法来实现,如下:

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

1. Connect: 某个channel成功连接到另一个服务器称为“连接就绪”
2. Accept: 一个server socket channel准备好接收新进入的连接称为“接收就绪”
3. Read: 一个有数据可读的通道可以说是“读就绪”
4. Write: 等待写数据的通道可以说是“写就绪”。

这四种事件用SelectionKey的四个常量来表示：

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

### SelectionKey

当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些属性：

- interest集合
- ready集合
- Channel
- Selector
- 附加的对象（可选）

#### interest集合

就像[向Selector注册通道](https://ifeve.com/selectors/#Registering)一节中所描述的，interest集合是你所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合，像这样：

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;

boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;

boolean isInterestedInRead = interestSet & SelectionKey.OP_READ;

boolean isInterestedInWrite = interestSet & SelectionKey.OP_WRITE;
```

可以看到，用“位与”操作interest 集合和给定的SelectionKey常量，可以确定某个确定的事件是否在interest 集合中。

但是，也可以使用以下四个方法，它们都会返回一个布尔类型：

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

#### ready集合

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合：

```java
int readySet = selectionKey.readyOps();
```

#### Channel + Selector

从SelectionKey访问Channel和Selector很简单。如下：

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

#### 附加的对象

可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加 与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法如下：

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

还可以在用register()方法向Selector注册Channel的时候附加对象。如：

```java
SelectionKey key = channel.register(selector,SelectionKey.OP_READ, theObject);
```

### 通过Selector选择通道

一旦向Selector注册了一个或多个通道，就可以调用集合重载select（）方法。这些方法会返回“感兴趣”事件已经准备就绪的那些通道。

下面是select()方法：

- int select() ： 阻塞到至少有一个通道在你注册的事件上就绪了
- int select(long timeout)：和select()一样，除了最长会阻塞timeout毫秒(参数)。
- int selectNow() ：不会阻塞，不管什么通道就绪都立刻返回（此方法执行**非阻塞**的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）

select()方法返回的int值表示有多少通道已经就绪。

### selectedKeys()

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
Set selectedKeys = selector.selectedKeys();
```

当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下：

```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
 // 这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。

### wakeUp()

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回.

### close()

用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。

## Scatter/Gather

##  通道之间的数据传输

