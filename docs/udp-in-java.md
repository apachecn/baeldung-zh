# Java 中的 UDP 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/udp-in-java>

## 1。概述

在本文中，我们将探索通过用户数据报协议( [UDP](https://web.archive.org/web/20220628112931/https://en.wikipedia.org/wiki/User_Datagram_Protocol) )使用 Java 进行网络通信。

UDP 是一种通信协议，**通过网络传输独立的数据包，不保证到达，也不保证交付顺序**。

互联网上的大多数通信都是通过传输控制协议(TCP)进行的，然而，UDP 也有它的位置，我们将在下一节探讨它。

## 2。为什么使用 UDP？

UDP 与更常见的 TCP 有很大不同。但是在考虑 UDP 的表面缺点之前，重要的是要理解开销的减少会使它比 TCP 快得多。

除了速度，我们还需要记住，一些类型的通信不需要 TCP 的可靠性，而是重视低延迟。该视频是一个很好的例子，说明在 UDP 而不是 TCP 上运行的应用程序可能会从中受益。

## 3。构建 UDP 应用程序

构建 UDP 应用程序与构建 TCP 系统非常相似；唯一的区别是我们没有在客户端和服务器之间建立点对点的连接。

设置也非常简单。Java 附带了对 UDP 的内置网络支持——这是`java.net`包的一部分。因此，要在 UDP 上执行网络操作，我们只需要从`java.net`包中导入类:`java.net.DatagramSocket`和`java.net.DatagramPacket`。

在下面的章节中，我们将学习如何设计通过 UDP 通信的应用程序；对于这个应用程序，我们将使用流行的 echo 协议。

首先，我们将构建一个回送发送给它的任何消息的回送服务器，然后构建一个只向服务器发送任意消息的回送客户端，最后，我们将测试应用程序以确保一切正常。

## 4。服务器

在 UDP 通信中，单个消息被封装在通过`DatagramSocket`发送的`DatagramPacket`中。

让我们从设置一个简单的服务器开始:

```java
public class EchoServer extends Thread {

    private DatagramSocket socket;
    private boolean running;
    private byte[] buf = new byte[256];

    public EchoServer() {
        socket = new DatagramSocket(4445);
    }

    public void run() {
        running = true;

        while (running) {
            DatagramPacket packet 
              = new DatagramPacket(buf, buf.length);
            socket.receive(packet);

            InetAddress address = packet.getAddress();
            int port = packet.getPort();
            packet = new DatagramPacket(buf, buf.length, address, port);
            String received 
              = new String(packet.getData(), 0, packet.getLength());

            if (received.equals("end")) {
                running = false;
                continue;
            }
            socket.send(packet);
        }
        socket.close();
    }
}
```

我们创建一个全局变量`DatagramSocket`，我们将使用它来发送数据包，创建一个字节数组来包装我们的消息，创建一个状态变量`running`。

为了简单起见，服务器扩展了`Thread`，所以我们可以在`run`方法中实现一切。

在`run`内部，我们创建了一个 while 循环，该循环一直运行到`running`因某种错误或来自客户端的终止消息而变为 false。

在循环的顶部，我们实例化了一个`DatagramPacket`来接收传入的消息。

接下来，我们调用套接字上的`receive`方法。该方法会一直阻塞，直到有消息到达，并将消息存储在传递给它的`DatagramPacket`的字节数组中。

收到消息后，我们检索客户端的地址和端口，因为我们将发送回响应
。

接下来，我们创建一个`DatagramPacket`来向客户端发送消息。请注意接收数据包的签名差异。这个还需要我们将消息发送到的客户端的地址和端口。

## 5。客户端

现在，让我们为这个新服务器部署一个简单的客户端:

```java
public class EchoClient {
    private DatagramSocket socket;
    private InetAddress address;

    private byte[] buf;

    public EchoClient() {
        socket = new DatagramSocket();
        address = InetAddress.getByName("localhost");
    }

    public String sendEcho(String msg) {
        buf = msg.getBytes();
        DatagramPacket packet 
          = new DatagramPacket(buf, buf.length, address, 4445);
        socket.send(packet);
        packet = new DatagramPacket(buf, buf.length);
        socket.receive(packet);
        String received = new String(
          packet.getData(), 0, packet.getLength());
        return received;
    }

    public void close() {
        socket.close();
    }
}
```

代码与服务器的代码没有太大的不同。我们有服务器的全局`DatagramSocket`和地址。我们在构造函数中实例化这些。

我们有一个单独的方法向服务器发送消息并返回响应。

我们首先将字符串消息转换成一个字节数组，然后创建一个`DatagramPacket`来发送消息。

接下来，我们发送消息。我们立即将`DatagramPacket`转换为接收。

当回显到达时，我们将字节转换为字符串并返回该字符串。

## 6。测试

在类`UDPTest.java`中，我们简单地创建一个测试来检查我们两个应用程序的响应能力:

```java
public class UDPTest {
    EchoClient client;

    @Before
    public void setup(){
        new EchoServer().start();
        client = new EchoClient();
    }

    @Test
    public void whenCanSendAndReceivePacket_thenCorrect() {
        String echo = client.sendEcho("hello server");
        assertEquals("hello server", echo);
        echo = client.sendEcho("server is working");
        assertFalse(echo.equals("hello server"));
    }

    @After
    public void tearDown() {
        client.sendEcho("end");
        client.close();
    }
}
```

在`setup`中，我们启动服务器并创建客户端。而在`tearDown`方法中，我们向服务器发送一个终止消息，以便它可以关闭，同时我们关闭客户端。

## 7。结论

在本文中，我们学习了用户数据报协议，并成功地构建了我们自己的通过 UDP 通信的客户机-服务器应用程序。

要获得本文中使用的示例的完整源代码，可以查看 GitHub 项目。