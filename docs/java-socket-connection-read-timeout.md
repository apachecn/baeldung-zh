# Java 套接字的连接超时与读取超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-socket-connection-read-timeout>

## 1.介绍

在本教程中，**我们将重点介绍 [Java 套接字编程](/web/20220626204309/https://www.baeldung.com/a-guide-to-java-sockets)** 的超时异常。我们的目标是理解为什么会出现这些异常，以及如何处理它们。

## 2.Java 套接字和超时

套接字是两个计算机应用程序之间逻辑链接的一个端点。换句话说，它是应用程序用来通过网络发送和接收数据的逻辑接口。

一般来说，套接字是 IP 地址和端口号的组合。每个套接字都分配有一个特定的端口号，用于标识服务。

基于连接的服务使用基于 [TCP-](/web/20220626204309/https://www.baeldung.com/cs/udp-vs-tcp) 的流套接字。为此， **Java 为客户端编程**提供了`java.net.Socket`类。另一方面，**服务器端 TCP/IP 编程利用了`java.net.ServerSocket` 类**。

另一种套接字是基于 [UDP-](/web/20220626204309/https://www.baeldung.com/udp-in-java) 的数据报套接字，用于无连接服务。 **Java 为 UDP 操作**提供`java.net.DatagramSocket `。然而，在本教程中，我们将重点放在 TCP/IP 套接字上。

## 3.连接超时

### 3.1.什么是“连接超时”？

为了从客户端建立到服务器的连接，调用**套接字构造函数**，它实例化一个套接字对象。**构造器将远程主机地址和端口号作为输入参数**。之后，它会尝试根据给定的参数建立到远程主机的连接。

**操作阻塞所有其他进程，直到成功连接**。但是，如果在某个时间后连接不成功，程序会抛出一个`ConnectionException`并显示“连接超时”消息:

```java
java.net.ConnectException: Connection timed out: connect
```

从服务器端来看，`ServerSocket`类持续监听传入的连接请求。**当`ServerSocket`收到连接请求时，它调用`accept()`方法实例化一个新的套接字对象**。同样地，这个方法也会封锁，直到它与远端用户端建立成功的连接。

如果 TCP 握手没有完成，连接仍然不成功。因此，**程序抛出一个`IOException` ，表示在建立一个新的连接**时发生了一个错误。

### 3.2.为什么会发生？

连接超时错误可能有多种原因:

*   没有服务正在侦听远程主机上的给定端口
*   远程主机不接受任何连接
*   远程主机不可用
*   互联网连接速度慢
*   没有到远程主机的转发路径

### 3.3.怎么处理？

阻塞时间没有限制，程序员可以为客户端和服务器操作预设超时选项。对于客户端，我们将首先创建一个空的套接字。之后，我们将使用`connect(SocketAddress endpoint, int timeout)`方法并设置超时参数:

```java
Socket socket = new Socket(); 
SocketAddress socketAddress = new InetSocketAddress(host, port); 
socket.connect(socketAddress, 30000);
```

**超时单位为毫秒**，应大于 0。但是，如果在方法调用返回之前超时，它将抛出一个`SocketTimeoutException`:

```java
Exception in thread "main" java.net.SocketTimeoutException: Connect timed out
```

**对于服务器端，我们将使用`setSoTimeout(int timeout)`方法**来设置超时值。`timeout`值定义了`ServerSocket.accept()`方法将阻塞多长时间:

```java
ServerSocket serverSocket = new new ServerSocket(port);
serverSocket.setSoTimeout(40000);
```

同样，`timeout`单位应该是毫秒，并且应该大于 0。如果在方法返回之前超时，它将抛出一个`SocketTimeoutException`。

有时，**防火墙会因为安全原因**封锁某些端口。因此，当客户端尝试建立与服务器的连接时，可能会出现“连接超时”错误。因此，**我们应该检查防火墙设置**，在将它绑定到服务之前，看看它是否阻塞了一个端口。

## 4.读取超时

### 4.1.什么是“读取超时”？

**`read()`方法调用在`InputStream`中阻塞，直到它完成从套接字读取数据字节。**操作等待，直到从套接字中读取至少一个数据字节。然而，如果该方法在未指定的时间后没有返回任何东西，**它抛出一个带有“读取超时”错误消息的`InterrupedIOException`**:

```java
java.net.SocketTimeoutException: Read timed out
```

### 4.2.为什么会发生？

从客户端来看，如果服务器花费更长的时间来响应和发送信息，则会发生“读取超时”错误**。这可能是由于互联网连接速度慢，或者主机可能脱机。**

从服务器端来看，当**服务器读取数据的时间比预设的超时时间**长时，就会发生这种情况。

### 4.3.怎么处理？

对于 TCP 客户端和服务器，我们可以用`setSoTimeout(int timeout)`方法指定`socketInputStream.read()`方法 **阻塞的时间:**

```java
Socket socket = new Socket(host, port);
socket.setSoTimeout(30000);
```

但是，如果在方法返回之前超时，程序将抛出一个`SocketTimeoutException`。

## 5.结论

在本文中，我们讨论了 Java 套接字编程中的超时异常，并学习了如何处理它们。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220626204309/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)