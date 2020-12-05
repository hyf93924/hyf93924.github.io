---
title: Java NIO有关笔记
date: 2017-08-31 10:43:12
tags: java
---
### 首先先来了解一下NIO和传统IO的区别：
传统的IO是基于Byte(字节)和Stream(数据流)的，相应的IO操作都是阻塞的，主要问题就是系统资源的浪费。从流中一次可以读取一个或多个字节，这里没有任何缓存而且只能顺序从流中读取数据，如果需要跳过一些字节或者再读取已经读过的字节，你必须将从流中读取的数据缓存起来
而NIO是面向块的，数据会先被读/写到Buffer中，而且可以控制读取什么位置的数据。不过你需要增加额外的工作就是检查你需要的数据是否已经全部到了Buffer中，还需要保证当有更多数据进入Buffer中时，原先Buffer中还未被处理的数据不会被覆盖<!--more-->

### 阻塞IO和非阻塞IO
所有Java IO流操作都是阻塞的，当一条线程执行read()或write()方法时，这条线程会一直阻塞直到读取到了一些数据或者要写出去的数据已经全部写出，这期间这条线程不会做任何其他操作
Java NIO的非阻塞模式（NIO也有阻塞和非阻塞，阻塞模式除了使用Buffer存储数据外和IO没什么区别）允许一条线程从channel(通道)中读取数据，通过返回值来判断Buffer中是否还有数据，如果没有数据，NIO不会阻塞，过段时间再回来判断一下有没有数据。写操作也一样，一条线程将Buffer中的数据写入channel中，不会等待数据全部写完才会返回，而是调用完write()就会继续向下执行

### selector
Java NIO的selector允许一条线程去监听多个channel的输入，可以向一个selector注册多个channel，然后调用selector的select()方法判断是否有新的连接进来或者已经在selector上注册时channel是否有数据进入。selector的机制是让一个线程管理多个channel变得简单

---

### 总结
NIO允许你用一个线程或多个线程管理很多个channel，代价是程序的处理和处理IO相比更加复杂。
如果你需要同时管理成千上万的连接，但是每个连接只发送少量的数据，用NIO实现会更好一点
如果你只有少量的连接，但是每个连接都占有很高的带宽，同时发送很多的数据，传统IO会更好点

---
继续，NIO有几个核心的部分：
> Channels
> Buffer
> Selectors
   

### Channels和Buffer
基本上所有的IO在NIO中都从一个channel开始，channel有点像流，数据可以从channel读到buffer中，也可以从buffer中写到channel中，如图：
<img src="/images/java/2017083101.png" style="width: 400px;"/>
不过一个channel既可以读又可以写，而一个stream是单向的（所以分InputStream和OutputStream）。
NIO的主要Channel有4中：
> FileChannel：读写文件
> DatagramChannel：UDP协议网络通信
> SocketChannel：TCP协议网络通信
> ServerSocketChannel：监听TCP连接
   

Buffer：
> ByteBuffer
> CharBuffer
> DoubleBuffer
> FloatBuffer
> IntBuffer
> LongBuffer
> ShortBuffer
   

这些buffer涵盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char
Buffer中有3个很重要的变量：
> capacity：总容量
> position：指针当前位置
> limit：读/写边界位置
   

<img src="/images/java/2017083102.jpg" style="width: 400px;"/>
在对Buffer进行读/写的过程中，position会往后移动，而 limit 就是 position 移动的边界。由此不难想象，在对Buffer进行写入操作时，limit应当设置为capacity的大小，而对Buffer进行读取操作时，limit应当设置为数据的实际结束位置。（注意：将Buffer数据 写入 通道是Buffer 读取 操作，从通道 读取 数据到Buffer是Buffer 写入 操作）
同时Buffer提供了一些方法来正确设置position和limit的值，常用的：
> flip()：设置limit为position的值，然后position设置为0，在Buffer读取前操作
> rewind()：仅仅是将position设置为0，一般在重新读取Buffer前调用
> clear()：回到初始状态，即limit等于capacity，position为0
> compact()：将未读取完的数据移动到缓冲区开头，并将position设置为这段数据末尾的下一个位置，等价于重新向缓冲区写入这么一段数据
   

### Selector：
selector允许单线程处理多个channel，用于采集各个通道的状态，我们先将通道注册到selector，并设置好关心的事件，然后再通过select()方法静静的等待事件的发生。四种不同类型的监听：
> Connect
> Accept
> Read
> write
   

这四种监听用常量表示：
> SelectionKey.OP_CONNECT
> SelectionKey.OP_ACCEPT
> SelectionKey.OP_READ
> SelectionKey.OP_WRITE
   

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：
```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```
将Channel注册到Selector首先需要将Channel设置成非阻塞模式，否则会抛出异常：
```
//创建一个selector
Selector selector = Selector.open();
channel.configureBlocking(false);
//注册到selector（监听其ACCEPT事件）
listenChannel.register(selector, SelectionKey.OP_ACCEPT);
```

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。可以遍历这个已选择的键集合来访问就绪的通道
```
selector.select();  //阻塞，直到有监听事件发生
Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
//通过迭代器依次访问select出来的channel时间
while (keyIterator.hasNext()){
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
 		//TODO
 	}else if(key.isConnectable()){
		//TODO
 	}else if (key.isReadable()){
		//TODO
 	}else if (key.isWritable()){
		//TODO
 	}
 	keyIterator.remove();
 }
```

---
完整实例：
```
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
    	//TODO
    } else if (key.isConnectable()) {
    	//TODO
    } else if (key.isReadable()) {
    	//TODO
    } else if (key.isWritable()) {
    	//TODO
    }
    keyIterator.remove();
  }
}
```