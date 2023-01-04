# Java NIO 数据通道

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nio-datagramchannel>

## 1.概观

在本教程中，我们将探索允许我们[发送和接收 UDP 数据包](/web/20220524034344/https://www.baeldung.com/udp-in-java)的`DatagramChannel`类。

## 2.`DatagramChannel`

在互联网支持的各种协议中， [TCP 和 UDP](/web/20220524034344/https://www.baeldung.com/cs/udp-vs-tcp) 是最常见的。

TCP 是面向连接的协议， **UDP 是面向数据报的协议，性能很高，但不太可靠**。由于 UDP 的不可靠性，它经常被用于**发送广播或组播数据传输**。

Java 的 NIO 模块的 [`DatagramChannel`](https://web.archive.org/web/20220524034344/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/channels/DatagramChannel.html) 类为面向数据报的套接字提供了一个可选通道**。换句话说，它允许创建一个数据报通道来发送和接收数据报(UDP 包)。**

让我们使用`DatagramChannel`类创建一个通过本地 IP 地址发送数据报的客户机和一个接收数据报的服务器。

## 3.打开并绑定

首先，让我们用`openChannel`方法创建`DatagramChannelBuilder`类，它提供一个开放但未连接的数据报通道:

```
public class DatagramChannelBuilder {
    public static DatagramChannel openChannel() throws IOException {
        DatagramChannel datagramChannel = DatagramChannel.open();
        return datagramChannel;
    }
}
```

然后，我们需要将一个打开的通道绑定到本地地址，以侦听入站 UDP 数据包。

因此，我们将添加将`DatagramChannel`绑定到所提供的本地地址的`bindChannel`方法:

```
public static DatagramChannel bindChannel(SocketAddress local) throws IOException {
    return openChannel().bind(local); 
}
```

现在，我们可以使用 `DatagramChannelBuilder` 类来创建在配置的套接字地址上发送/接收 UDP 包的客户机/服务器。

## 4.客户

首先，让我们用`startClient`方法创建`DatagramClient`类，它使用了已经讨论过的`DatagramChannelBuilder`类的`bindChannel`方法

```
public class DatagramClient {
    public static DatagramChannel startClient() throws IOException {
        DatagramChannel client = DatagramChannelBuilder.bindChannel(null);
        return client;
    }
}
```

由于**客户端不需要监听入站 UDP 包，我们在绑定通道时为地址**提供了一个`null`值。

然后，让我们添加`sendMessage`方法来发送服务器地址上的数据报:

```
public static void sendMessage(DatagramChannel client, String msg, SocketAddress serverAddress) throws IOException {
    ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
    client.send(buffer, serverAddress);
} 
```

就是这样！现在，我们准备使用客户端发送消息:

```
DatagramChannel client = startClient();
String msg = "Hello, this is a Baeldung's DatagramChannel based UDP client!";
InetSocketAddress serverAddress = new InetSocketAddress("localhost", 7001);

sendMessage(client, msg, serverAddress);
```

注意:因为我们已经将消息发送到了`localhost:7001`地址，所以我们必须使用相同的地址启动我们的服务器。

## 5.计算机网络服务器

类似地，让我们用`startServer`方法创建`DatagramServer`类，在`localhost:7001`地址上启动一个服务器:

```
public class DatagramServer {
    public static DatagramChannel startServer() throws IOException {
        InetSocketAddress address = new InetSocketAddress("localhost", 7001);
        DatagramChannel server = DatagramChannelBuilder.bindChannel(address);
        System.out.println("Server started at #" + address);
        return server;
    }
}
```

然后，让我们添加从客户端接收数据报、提取消息并打印出来的`receiveMessage`方法:

```
public static void receiveMessage(DatagramChannel server) throws IOException {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    SocketAddress remoteAdd = server.receive(buffer);
    String message = extractMessage(buffer);
    System.out.println("Client at #" + remoteAdd + "  sent: " + message);
}
```

此外，为了从接收到的缓冲区中提取客户端的消息，我们需要添加`extractMessage`方法:

```
private static String extractMessage(ByteBuffer buffer) {
    buffer.flip();
    byte[] bytes = new byte[buffer.remaining()];
    buffer.get(bytes);
    String msg = new String(bytes);

    return msg;
} 
```

这里，我们在 [`ByteBuffer`](https://web.archive.org/web/20220524034344/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/ByteBuffer.html) 实例上使用了`flip`方法，将其从从 I/O 读取转换为从 I/O 写入。此外，`flip`方法将限制设置为当前位置，并将位置设置为零，以便让我们从头开始读取。

现在，我们可以启动服务器并接收来自客户端的消息:

```
DatagramChannel server = startServer();
receiveMessage(server);
```

因此，当服务器收到我们的消息时，打印输出将是:

```
Server started at #localhost/127.0.0.1:7001
Client at #/127.0.0.1:52580  sent: Hello, this is a Baeldung's DatagramChannel based UDP client!
```

## 6.`DatagramChannelUnitTest`

现在我们已经准备好了客户端和服务器，我们可以编写一个单元测试来验证端到端数据报(UDP 数据包)的交付:

```
@Test
public void whenClientSendsAndServerReceivesUDPPacket_thenCorrect() throws IOException {
    DatagramChannel server = DatagramServer.startServer();
    DatagramChannel client = DatagramClient.startClient();
    String msg1 = "Hello, this is a Baeldung's DatagramChannel based UDP client!";
    String msg2 = "Hi again!, Are you there!";
    InetSocketAddress serverAddress = new InetSocketAddress("localhost", 7001);

    DatagramClient.sendMessage(client, msg1, serverAddress);
    DatagramClient.sendMessage(client, msg2, serverAddress);

    assertEquals("Hello, this is a Baeldung's DatagramChannel based UDP client!", DatagramServer.receiveMessage(server));
    assertEquals("Hi again!, Are you there!", DatagramServer.receiveMessage(server));
}
```

首先，我们启动了绑定数据报通道的服务器来监听`localhost:7001`上的入站消息。然后，我们启动客户端，发送两条消息。

最后，我们在服务器上接收了入站消息，并将其与通过客户端发送的消息进行了比较。

## 7.其他方法

到目前为止，我们已经使用了由`DatagramChannel`类提供的`open`、`bind`、`send`和`receive`等**方法。现在，让我们快速浏览一下它的其他方便的方法。**

### 7.1.`configureBlocking`

默认情况下，数据报通道是阻塞的。当传递`false`值时，我们可以使用`configureBlocking`方法使通道不阻塞:

```
client.configureBlocking(false);
```

### 7.2.`isConnected`

`isConnected`方法返回数据报通道的状态——也就是说，它是连接还是断开。

### 7.3.`socket`

`socket`方法返回与数据报通道关联的 [`DatagramSocket`](https://web.archive.org/web/20220524034344/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/DatagramSocket.html) 类的对象。

### 7.4.`close`

此外，我们可以通过调用`DatagramChannel`类的`close`方法来关闭通道。

## 8.结论

在这个快速教程中，我们探索了 Java NIO 的`DatagramChannel`类，它允许创建数据报通道来发送/接收 UDP 包。

首先，我们检查了一些方法，如`open`和`bind`，它们同时允许数据报通道监听入站 UDP 数据包。

然后，我们创建了一个客户端和一个服务器**，使用`DatagramChannel`类**来探索端到端的 UDP 包交付。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524034344/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-nio-2)