# BIO，NIO，AIO总结
先回顾几个概念：同步与异步，阻塞和非阻塞

1、同步与异步：
同步：发起一个调用之后，被调用者未处理完请求之前，调用不返回
异步：发起一个调用之后，立刻得到被调用者的回应表示已接受到请求，但是被调用者并没有返回结果，此时我们可以处理其他请求，被调用者通常依靠时间，回调等机制来通知调用者其返回结果。
最大区别：异步的话调用者不需要等待处理结果，被调用者会通过回调等机制来通知调用者其返回结果。

2、阻塞和非阻塞：
阻塞：阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当前条件就绪才能继续
非阻塞：非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先干其他事情。

例子：
故事：老王烧开水。
出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。
老王想了想，有好几种等待方式

1.老王用水壶煮水，并且站在那里，不管水开没开，每隔一定时间看看水开了没。－同步阻塞

老王想了想，这种方法不够聪明。

2.老王还是用水壶煮水，不再傻傻的站在那里看水开，跑去寝室上网，但是还是会每隔一段时间过来看看水开了没有，水没有开就走人。－同步非阻塞

老王想了想，现在的方法聪明了些，但是还是不够好。

3.老王这次使用高大上的响水壶来煮水，站在那里，但是不会再每隔一段时间去看水开，而是等水开了，水壶会自动的通知他。－异步阻塞

老王想了想，不会呀，既然水壶可以通知我，那我为什么还要傻傻的站在那里等呢，嗯，得换个方法。

4.老王还是使用响水壶煮水，跑到客厅上网去，等着响水壶自己把水煮熟了以后通知他。－异步非阻塞

老王豁然，这下感觉轻松了很多。

同步和异步：同步就是烧开水，需要自己去轮询，异步就是水开了，然后水壶会通知你水已经开了。同步异步针对的是操作结果来说，会不会等待结果返回。
阻塞非阻塞：阻塞是说在煮水的过程中，你不可以去干其他事情，非阻塞是在同样的情况下，可以同时去干别的事情，阻塞是相对于线程是否被阻塞来说的。

## 1.BIO（Blocking I/O）
同步阻塞I/O模式，数据的读写必须阻塞在一个线程中等待其完成。
### 1.1传统BIO
BIO通信（一请求一应答），模型图如下（图源网络，源出处不明）
![BIO通信](img/BIO.png)
采用BIO通信模型的服务端，通常有一个独立的Acceptor线程负责监听客户端的连接。我们一般通过在while(true)循环中服务端会调用accept()方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就建立通信套接字进行读写，此时不能再接收其他的客户端的请求，只能等待同当前连接客户的操作执行完成，通过多线程方式可以支持多个客户端的连接。如上图所示。

如果BIO通信模型能够同时处理多个客户端的请求，就必须使用多线程（主要原因是socket.accept(),socket.read(),socket.write()涉及的三个主要函数时同步阻塞的），也就是说它在接受到客户端连接请求之后为每个客户端创建了一个新的线程进行链路处理。处理完成之后，通过输出流返回应答器给客户端，线程销毁。这就是典型的请求——应答通信模型。
设想如果这个通信连接不作任何事情的话，就会造成不必要的线程开销。不过可以使用线程池机制来改善，线程池还可以让线程的创建和回收成本相对较低。使用FixedThreadPool可以有效的控制了线程的最大数量。保证资源的有效控制，实现了N（客户请求数量）：M（处理请求的线程数量）的伪异步I/O模型（N可以远远大于M）

### 1.2伪异步IO
为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，后端对他的线程模型进行优化，使用线程池制度令狐调配线程资源。
（图源网络，原出处不明）
![伪异步IO](img/伪异步IO.png)
采用线程池和任务队列可以实现一个叫做伪异步的IO通信框架，他的模型图如上图所示。当有新的客户端接入时，将客户端的Socket封装成一个Task（实现Runnable接口）投递到后端的线程池中进行处理。JDK的线程池维护一个消息队列和N个活跃线程，对消息队列的任务进行处理。资源可控。

### 1.3代码示例
演示BIO模型，客户端发送“时间+：hello world” ，客户端处理。
###### 客户端

    public class IOClient {
        public static void main(String[] args){

            new Thread(() -> {
                try{
                    Socket socket = new Socket("127.0.0.1",3333);
    //                int len;
    //                byte[] data = new byte[1024];
    //                InputStream inputStream = socket.getInputStream();
    //                while((len = inputStream.read(data)) != -1){
    //                    System.out.println(new String(data,0,len));
    //                }
                    while (true) {
                        try{
                            //向socket发送请求
                            socket.getOutputStream().write((new Date() + ": hello world").getBytes());
                            Thread.sleep(2000);
                        }catch (Exception e){

                        }
                    }
                }catch (IOException e){

                }
            }).start();
        }
    }

###### 服务端

    public class IOServer {
        public static void main(String[] args) throws IOException{
            ServerSocket serverSocket = new ServerSocket(3333);

            new Thread(() -> {
                while(true) {
                    try{
                        //阻塞方法获取新的连接
                        Socket socket = serverSocket.accept();
    //                    OutputStream outputStream = socket.getOutputStream();
    //                    outputStream.write(("success").getBytes());
                        //接收客户端请求之后为每个客户端创建一个新的线程进行链路处理
                        new Thread(() -> {
                            try {
                                int len;
                                byte[] data = new byte[1024];
                                InputStream inputStream = socket.getInputStream();
                                //按照字节流的方式进行读取数据
                                while((len = inputStream.read(data)) != -1){
                                    System.out.println(new String(data,0,len));
                                }
                            }catch (IOException e){

                            }
                        }).start();
                    }catch (Exception e){

                    }
                }
            }).start();
        }
    }

### 1.4小结
活动连接数不是特别高的情况下，大概小于1000时，编程模型简单。

## 2.NIO(New I/O)
### 2.1 NIO简介
NIO是一种同步非阻塞（要轮询结果但可以做其他事）的IO模型，java1.4中引入这种NIO框架，提供了Channel，Selector，Buffer等抽象。
NIO中的N还可以理解为Non-blocking，不单纯是New。支持面向缓冲的，给予通道的IO操作方法。NIO提供了与传统BIO模型中的Socket和ServerSocket相对应的SocketChannel和ServerSocketChannel两种不同的套接字通道实现。两种通道都支持阻塞和非阻塞两种模式，阻塞模式使用就像传统的支持一样，简单但性能可靠性不好。非阻塞模式正好相反，对于低阻塞、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络)应用，应使用NIO的非阻塞模式来开发。

### 2.2NIO的特性/NIO与IO的区别
面试问题的话，首先NIO流是非阻塞IO，而IO流是阻塞IO说起。然后可以从NIO的3个核心组件/特性为NIO带来的改进进行分析。

**1)Non-blockingIO（非阻塞IO）**
IO流是阻塞的，NIO流是不阻塞的
Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
Java IO的各种流是阻塞的。这意味着，当一个线程调用 read() 或 write() 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了

**2)Buffer（缓冲区）**
IO面向流，而NIO面向缓冲区
Buffer是一个对象，包含着要写入或读出的数据。在NIO类库中加入Buffer对象，体现了新库和原IO的重要区别。在面向流的IO中，可以将数据直接写入或者数据直接读到Stream对象中。虽然Stream中也有Buffer开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而NIO却是直接读到Buffer中进行操作。
在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。
最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer,还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。

**3)Channel（通道）**
NIO 通过Channel（通道） 进行读写。
通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。

**4）Selector（选择器）**
NIO有选择器，而IO没有
选择器用于使用单个线程处理多个通道。因此他需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说太昂贵了
![Selector](img/Selector.png)

### 2.3 NIO读数据和写数据方法
通常来说NIO中所有的IO都是从Chnnel开始的
从通道进行数据读取：创建一个缓冲区，然后请求通道读取数据
从通道进行数据写入：创建一个缓冲区，填充数据，并要求通道写入数据
![NIO读写数据的方式](img/NIO读写数据的方式.png)

### 2.4 NIO核心组件简单介绍
NIO 包含下面几个核心的组件：

Channel(通道)
Buffer(缓冲区)
Selector(选择器)

### 2.5 代码示例

    public class NIOServer {
        public static void main(String[] args) throws IOException {
            // 1. serverSelector负责轮询是否有新的连接，服务端监测到新的连接之后，不再创建一个新的线程，
            // 而是直接将新连接绑定到clientSelector上，这样就不用 IO 模型中 1w 个 while 循环在死等
            Selector serverSelector = Selector.open();
            // 2. clientSelector负责轮询连接是否有数据可读
            Selector clientSelector = Selector.open();

            new Thread(() -> {
                try {
                    // 对应IO编程中服务端启动
                    ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                    listenerChannel.socket().bind(new InetSocketAddress(3333));
                    listenerChannel.configureBlocking(false);
                    listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                    while (true) {
                        // 监测是否有新的连接，这里的1指的是阻塞的时间为 1ms
                        if (serverSelector.select(1) > 0) {
                            Set<SelectionKey> set = serverSelector.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();

                            while (keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();

                                if (key.isAcceptable()) {
                                    try {
                                        // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                        SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                        clientChannel.configureBlocking(false);
                                        clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                    } finally {
                                        keyIterator.remove();
                                    }
                                }

                            }
                        }
                    }
                } catch (IOException ignored) {
                }
            }).start();
            new Thread(() -> {
                try {
                    while (true) {
                        // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为 1ms
                        if (clientSelector.select(1) > 0) {
                            Set<SelectionKey> set = clientSelector.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();

                            while (keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();

                                if (key.isReadable()) {
                                    try {
                                        SocketChannel clientChannel = (SocketChannel) key.channel();
                                        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                        // (3) 面向 Buffer
                                        clientChannel.read(byteBuffer);
                                        byteBuffer.flip();
                                        System.out.println(
                                                Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());
                                    } finally {
                                        keyIterator.remove();
                                        key.interestOps(SelectionKey.OP_READ);
                                    }
                                }

                            }
                        }
                    }
                } catch (IOException ignored) {
                }
            }).start();

        }
    }

