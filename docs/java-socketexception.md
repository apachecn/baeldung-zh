# 如何处理 Java SocketException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-socketexception>

## 1.介绍

在这个快速教程中，我们将通过一个例子来了解`SocketException`的成因。

当然，我们还将讨论如何处理异常。

## 2.`SocketException`的原因

**`SocketException`最常见的原因是向关闭的[套接字](/web/20221003154930/https://www.baeldung.com/a-guide-to-java-sockets)连接写入数据或从中读取数据。**另一个原因是在读取套接字缓冲区中的所有数据之前关闭连接。

让我们仔细看看一些常见的潜在原因。

### 2.1.慢速网络

网络连接不良可能是根本问题。设置较高的套接字连接超时可以降低慢速连接的`SocketException`速率:

```java
socket.setSoTimeout(30000); // timeout set to 30,000 ms
```

### 2.2.防火墙干预

一个[网络防火墙](/web/20221003154930/https://www.baeldung.com/cs/firewalls-intro)可以关闭套接字连接。如果我们可以进入防火墙，我们可以关闭它，看看它是否能解决问题。

否则，我们可以使用网络监控工具，如 [Wireshark](https://web.archive.org/web/20221003154930/https://www.wireshark.org/) 来检查防火墙活动。

### 2.3.长期空闲连接

空闲连接可能会被另一端遗忘(以节省资源)。如果我们必须长时间使用一个连接，我们可以发送[心跳消息](https://web.archive.org/web/20221003154930/https://en.wikipedia.org/wiki/Heartbeat_message)来防止空闲状态。

### 2.4.应用程序错误

最后但同样重要的是，`SocketException`可能是因为我们代码中的错误或 bug 而发生的。

为了演示这一点，让我们在端口 6699 上启动一个服务器:

```java
SocketServer server = new SocketServer();
server.start(6699);
```

当服务器启动时，我们将等待来自客户端的消息:

```java
serverSocket = new ServerSocket(port);
clientSocket = serverSocket.accept();
out = new PrintWriter(clientSocket.getOutputStream(), true);
in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
String msg = in.readLine();
```

一旦我们得到它，**我们将响应并关闭连接:**

```java
out.println("hi");
in.close();
out.close();
clientSocket.close();
serverSocket.close();
```

因此，假设一个客户端连接到我们的服务器并发送“嗨”:

```java
SocketClient client = new SocketClient();
client.startConnection("127.0.0.1", 6699);
client.sendMessage("hi");
```

到目前为止，一切顺利。

但是，如果客户端发送另一条消息:

```java
client.sendMessage("hi again");
```

由于客户端在连接中止后向服务器发送`“hi again”`，因此会发生`SocketException`。

## 3.`SocketException`的处理

处理`SocketException`非常简单明了。与任何其他检查过的异常类似，我们必须要么抛出它，要么用 try-catch 块包围它。

让我们来处理例子中的异常:

```java
try {
    client.sendMessage("hi");
    client.sendMessage("hi again");
} catch (SocketException e) {
    client.stopConnection();
}
```

这里，我们已经在异常发生后关闭了客户端连接。**重试不起作用，因为连接已经关闭。我们应该开始一个新的连接:**

```java
client.startConnection("127.0.0.1", 6699);
client.sendMessage("hi again");
```

## 4.结论

在本文中，我们看了一下`SocketException`的原因以及如何处理它。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20221003154930/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)