# Java NIO 选择器简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-selector>

## 1。概述

在本文中，我们将探索 Java NIO 的`Selector`组件的介绍部分。

选择器提供了一种机制，用于监控一个或多个 NIO 通道，并识别一个或多个通道何时可用于数据传输。

这样，**单个线程可以用于管理多个通道**，从而管理多个网络连接。

## 2。为什么要使用选择器？

有了选择器，我们可以用一个线程而不是几个来管理多个通道。**线程间的上下文切换对于操作系统**来说开销很大，另外，**每个线程都要占用内存。**

因此，我们使用的线程越少越好。然而，重要的是要记住**现代操作系统和 CPU 在多任务处理方面越来越好**，因此多线程的开销随着时间的推移不断减少。

在这里，我们将讨论如何使用选择器用单线程处理多个通道。

还要注意，选择器不仅仅帮助你读取数据；它们还可以侦听传入的网络连接，并通过慢速通道写入数据。

## 3。设置

要使用选择器，我们不需要任何特殊的设置。我们需要的所有类都在核心`java.nio`包中，我们只需要导入我们需要的。

之后，我们可以用一个选择器对象注册多个通道。当任何通道上发生 I/O 活动时，选择器都会通知我们。这就是我们如何在单线程上读取大量数据源的方法。

我们用选择器注册的任何通道都必须是`SelectableChannel`的子类。这些是可以被置于非阻塞模式的特殊类型的通道。

## 4。创建选择器

可以通过调用`Selector` 类的静态`open` 方法来创建选择器，该方法将使用系统的默认选择器提供程序来创建新的选择器:

```java
Selector selector = Selector.open();
```

## 5。注册可选频道

为了让选择器监控任何通道，我们必须向选择器注册这些通道。我们通过调用可选通道的`register`方法来实现这一点。

但是在向选择器注册通道之前，它必须处于非阻塞模式:

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

这意味着我们不能使用带有选择器的`FileChannel` s，因为它们不能像我们使用套接字通道那样切换到非阻塞模式。

第一个参数是我们之前创建的`Selector`对象，第二个参数定义了一个兴趣集**，**表示我们有兴趣通过选择器在被监控的通道中监听什么事件。

我们可以监听四个不同的事件，每个事件都由`SelectionKey`类中的一个常数表示:

*   `Connect`**–**当客户端试图连接到服务器时。由`SelectionKey.OP_CONNECT`代表
*   `Accept`**–**当服务器接受客户端的连接时。由`SelectionKey.OP_ACCEPT`代表
*   `Read`**–**当服务器准备好读取通道时。由`SelectionKey.OP_READ`代表
*   `Write`**–**当服务器准备写入通道时。由`SelectionKey.OP_WRITE`代表

返回的对象`SelectionKey`代表可选通道在选择器中的注册。我们将在下一节中进一步研究它。

## 6。`SelectionKey`对象

正如我们在上一节中看到的，当我们用选择器注册一个通道时，我们得到一个`SelectionKey`对象。该对象保存表示通道注册的数据。

它包含一些重要的属性，我们必须很好地理解这些属性才能在通道上使用选择器。我们将在下面的小节中研究这些属性。

### 6.1。利息设定

兴趣集定义了我们希望选择器在这个通道上关注的事件集。它是一个整数值；我们可以用下面的方法得到这个信息。

首先，我们有由`SelectionKey`的`interestOps`方法返回的利息集。然后我们在前面看过的`SelectionKey`中有了事件常数。

当我们对这两个值进行 AND 运算时，我们会得到一个布尔值，它告诉我们事件是否被监视:

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

### 6.2。就绪设定

就绪集定义了通道准备好的事件集。它也是一个整数值；我们可以用下面的方法得到这个信息。

我们已经得到了由`SelectionKey`的`readyOps`方法返回的就绪集合。当我们将该值与事件常量进行 AND 运算时，就像我们在兴趣集的情况下所做的那样，我们会得到一个布尔值，表示通道是否准备好接受某个特定值。

另一个更简单的替代方法是使用`SelectionKey'`的便利方法来达到同样的目的:

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWriteable();
```

### 6.3。频道

从`SelectionKey`对象访问正在观看的频道非常简单。我们只是调用了`channel`方法:

```java
Channel channel = key.channel();
```

### 6.4。选择器

就像获取通道一样，从`SelectionKey`对象中获取`Selector`对象非常容易:

```java
Selector selector = key.selector();
```

### 6.5。附着物体

我们可以将一个对象附加到一个`SelectionKey.` 上，有时我们可能想要给一个通道一个自定义 ID，或者附加我们想要跟踪的任何种类的 Java 对象。

附加对象是一种简便的方法。下面是如何从`SelectionKey`中附加和获取对象:

```java
key.attach(Object);

Object object = key.attachment();
```

或者，我们可以选择在通道注册期间附加一个对象。我们将它作为第三个参数添加到 channel 的`register`方法中，如下所示:

```java
SelectionKey key = channel.register(
  selector, SelectionKey.OP_ACCEPT, object);
```

## 7。频道键选择

到目前为止，我们已经了解了如何创建一个选择器，向它注册通道，并检查代表通道向选择器注册的`SelectionKey`对象的属性。

这只是该过程的一半，现在我们必须执行一个连续的过程来选择我们之前看到的就绪集。我们使用选择器的`select`方法进行选择，就像这样:

```java
int channels = selector.select();
```

此方法会一直阻止，直到至少有一个通道准备好进行操作。返回的整数表示其通道已准备好进行操作的键的数量。

接下来，我们通常检索所选键的集合进行处理:

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

我们获得的集合是由`SelectionKey`个对象组成的，每个键代表一个已注册的通道，该通道已准备好进行操作。

在这之后，我们通常迭代这个集合，对于每个键，我们获取通道并执行我们感兴趣的集合中出现的任何操作。

在一个通道的生命周期中，它可能被选择多次，因为它的键出现在不同事件的就绪集中。这就是为什么我们必须有一个连续的循环来捕获和处理发生的通道事件。

## 8。完整示例

为了巩固我们在前面几节中获得的知识，我们将构建一个完整的客户机-服务器示例。

为了便于测试我们的代码，我们将构建一个 echo 服务器和一个 echo 客户机。在这种设置中，客户端连接到服务器并开始向其发送消息。服务器回显每个客户端发送的消息。

当服务器遇到特定的消息，比如`end`，它会将其解释为通信结束，并关闭与客户端的连接。

### 8.1。服务器

这是我们的`EchoServer.java`代码:

```java
public class EchoServer {

    private static final String POISON_PILL = "POISON_PILL";

    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.bind(new InetSocketAddress("localhost", 5454));
        serverSocket.configureBlocking(false);
        serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        ByteBuffer buffer = ByteBuffer.allocate(256);

        while (true) {
            selector.select();
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();
            while (iter.hasNext()) {

                SelectionKey key = iter.next();

                if (key.isAcceptable()) {
                    register(selector, serverSocket);
                }

                if (key.isReadable()) {
                    answerWithEcho(buffer, key);
                }
                iter.remove();
            }
        }
    }

    private static void answerWithEcho(ByteBuffer buffer, SelectionKey key)
      throws IOException {

        SocketChannel client = (SocketChannel) key.channel();
        client.read(buffer);
        if (new String(buffer.array()).trim().equals(POISON_PILL)) {
            client.close();
            System.out.println("Not accepting client messages anymore");
        }
        else {
            buffer.flip();
            client.write(buffer);
            buffer.clear();
        }
    }

    private static void register(Selector selector, ServerSocketChannel serverSocket)
      throws IOException {

        SocketChannel client = serverSocket.accept();
        client.configureBlocking(false);
        client.register(selector, SelectionKey.OP_READ);
    }

    public static Process start() throws IOException, InterruptedException {
        String javaHome = System.getProperty("java.home");
        String javaBin = javaHome + File.separator + "bin" + File.separator + "java";
        String classpath = System.getProperty("java.class.path");
        String className = EchoServer.class.getCanonicalName();

        ProcessBuilder builder = new ProcessBuilder(javaBin, "-cp", classpath, className);

        return builder.start();
    }
}
```

这就是正在发生的事情；我们通过调用静态的`open`方法来创建一个`Selector`对象。然后我们也通过调用它的静态`open`方法，特别是一个`ServerSocketChannel`实例来创建一个通道。

这是因为 **`ServerSocketChannel`是可选的，对于面向流的监听套接字**来说是好的。

然后，我们将它绑定到我们选择的端口。还记得我们之前说过，在将可选通道注册到选择器之前，我们必须首先将其设置为非阻塞模式。接下来我们这样做，然后将通道注册到选择器。

我们现阶段不需要这个通道的`SelectionKey`实例，所以我们不会记住它。

Java NIO 使用面向缓冲区的模型，而不是面向流的模型。所以套接字通信通常是通过读写缓冲区来实现的。

因此，我们创建了一个新的`ByteBuffer`，服务器将对其进行读写。我们将其初始化为 256 字节，这只是一个任意值，取决于我们计划来回传输的数据量。

最后，我们执行选择过程。我们选择准备好的通道，检索它们的选择键，迭代这些键，并执行每个通道准备好的操作。

我们在无限循环中这样做，因为无论是否有活动，服务器通常都需要保持运行。

一个`ServerSocketChannel`可以处理的唯一操作是一个`ACCEPT`操作。当我们接受来自客户端的连接时，我们获得了一个`SocketChannel`对象，我们可以在其上进行读写操作。我们将其设置为非阻塞模式，并将其注册为对选择器的读操作。

在随后的一次选择中，该新通道将变为可读状态。我们检索它并将它的内容读入缓冲区。作为 echo 服务器，我们必须将这些内容写回客户端。

**当我们想要写入一个我们一直在读取的缓冲区时，我们必须调用`flip()`方法**。

我们最后通过调用`flip`方法将缓冲区设置为写模式，并简单地写入它。

定义了 `start()`方法，以便 echo 服务器可以在单元测试期间作为一个单独的进程启动。

### 8.2。客户端

这是我们的`EchoClient.java`代码:

```java
public class EchoClient {
    private static SocketChannel client;
    private static ByteBuffer buffer;
    private static EchoClient instance;

    public static EchoClient start() {
        if (instance == null)
            instance = new EchoClient();

        return instance;
    }

    public static void stop() throws IOException {
        client.close();
        buffer = null;
    }

    private EchoClient() {
        try {
            client = SocketChannel.open(new InetSocketAddress("localhost", 5454));
            buffer = ByteBuffer.allocate(256);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String sendMessage(String msg) {
        buffer = ByteBuffer.wrap(msg.getBytes());
        String response = null;
        try {
            client.write(buffer);
            buffer.clear();
            client.read(buffer);
            response = new String(buffer.array()).trim();
            System.out.println("response=" + response);
            buffer.clear();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return response;

    }
}
```

客户端比服务器简单。

我们使用单例模式在`start`静态方法中实例化它。我们从这个方法调用私有构造函数。

在私有构造函数中，我们在绑定服务器通道的同一个端口上打开一个连接，并且仍然在同一个主机上。

然后我们创建一个可以读写的缓冲区。

最后，我们有一个`sendMessage`方法，该方法将我们传递给它的任何字符串包装到一个字节缓冲区中，该缓冲区通过通道传输到服务器。

然后，我们从客户端通道读取，以获取服务器发送的消息。我们将此作为我们信息的回应。

### 8.3。测试

在一个名为`EchoTest.java`的类中，我们将创建一个测试用例，它启动服务器，向服务器发送消息，只有当从服务器收到相同的消息时才通过。作为最后一步，测试用例在完成之前停止服务器。

我们现在可以运行测试:

```java
public class EchoTest {

    Process server;
    EchoClient client;

    @Before
    public void setup() throws IOException, InterruptedException {
        server = EchoServer.start();
        client = EchoClient.start();
    }

    @Test
    public void givenServerClient_whenServerEchosMessage_thenCorrect() {
        String resp1 = client.sendMessage("hello");
        String resp2 = client.sendMessage("world");
        assertEquals("hello", resp1);
        assertEquals("world", resp2);
    }

    @After
    public void teardown() throws IOException {
        server.destroy();
        EchoClient.stop();
    }
}
```

## 9.`Selector.wakeup()`

正如我们前面看到的，调用`selector.select()`会阻塞当前线程，直到其中一个被监视的通道准备好运行。我们可以通过从另一个线程调用`selector.wakeup()`来覆盖它。

结果是**阻塞线程立即返回，而不是继续等待，无论通道是否准备就绪**。

我们可以使用`[CountDownLatch](/web/20220922104547/https://www.baeldung.com/java-countdown-latch)`并跟踪代码执行步骤来演示这一点:

```java
@Test
public void whenWakeUpCalledOnSelector_thenBlockedThreadReturns() {
    Pipe pipe = Pipe.open();
    Selector selector = Selector.open();
    SelectableChannel channel = pipe.source();
    channel.configureBlocking(false);
    channel.register(selector, OP_READ);

    List<String> invocationStepsTracker = Collections.synchronizedList(new ArrayList<>());

    CountDownLatch latch = new CountDownLatch(1);

    new Thread(() -> {
        invocationStepsTracker.add(">> Count down");
        latch.countDown();
        try {
            invocationStepsTracker.add(">> Start select");
            selector.select();
            invocationStepsTracker.add(">> End select");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();

    invocationStepsTracker.add(">> Start await");
    latch.await();
    invocationStepsTracker.add(">> End await");

    invocationStepsTracker.add(">> Wakeup thread");
    selector.wakeup();
    //clean up
    channel.close();

    assertThat(invocationStepsTracker)
      .containsExactly(
        ">> Start await",
        ">> Count down",
        ">> Start select",
        ">> End await",
        ">> Wakeup thread",
        ">> End select"
    );
}
```

在这个例子中，我们使用 Java NIO 的`Pipe`类打开一个通道进行测试。我们在线程安全列表中跟踪代码执行步骤。通过分析这些步骤，我们可以看到`selector.wakeup()`是如何释放被`selector.select()`阻塞的线程的。

## 10。结论

在本文中，我们介绍了 Java NIO 选择器组件的基本用法。

本文的完整源代码和所有代码片段可以在我的 [GitHub 项目](https://web.archive.org/web/20220922104547/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio-2)中找到。