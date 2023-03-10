# 使用 Java 获取当前机器的 IP 地址

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-ip-address>

## 1.概观

[IP 地址](/web/20220709110357/https://www.baeldung.com/cs/ipv4-vs-ipv6)或互联网协议地址唯一标识互联网上的设备。因此，知道运行我们的应用程序的设备的身份是一些应用程序的关键部分。

在本教程中，我们将检查使用 Java 检索计算机 IP 地址的各种方法。

## 2.找到本地 IP 地址

首先，我们来看看获取当前机器的本地 IPv4 地址的一些方法。

### 2.1.Java 网络库的本地地址

此方法使用 Java Net 库建立 UDP 连接:

```java
try (final DatagramSocket datagramSocket = new DatagramSocket()) {
    datagramSocket.connect(InetAddress.getByName("8.8.8.8"), 12345);
    return datagramSocket.getLocalAddress().getHostAddress();
}
```

这里，为了简单起见，我们使用 Google 的主 DNS 作为我们的目的主机，并提供 IP 地址`8.8.8.8.`。Java Net 库在这一点上只检查地址格式的有效性，因此地址本身可能是不可到达的。此外，我们使用随机端口`12345`通过`socket.connect()`方法创建 UDP 连接。在幕后，它设置发送和接收数据所需的所有变量，包括机器的本地地址，而实际上不向目的主机发送任何请求。

虽然这种解决方案在 Linux 和 Windows 机器上运行得很好，但在 macOS 上却有问题，并且不能返回预期的 IP 地址。

### 2.2.使用套接字连接的本地地址

或者，**我们可以通过可靠的互联网连接使用** **套接字连接来查找 IP 地址**:

```java
try (Socket socket = new Socket()) {
    socket.connect(new InetSocketAddress("google.com", 80));
    return socket.getLocalAddress().getHostAddress();
}
```

这里，同样为了简单起见，我们使用端口`80`上的连接`google.com`来获取主机地址。我们可以使用任何其他 URL 来创建套接字连接，只要它是可到达的。

### 2.3.关于复杂网络情况的警告

上面列出的方法在简单的网络情况下非常有效。然而，在机器有更多网络接口的情况下，行为可能不可预测。

换句话说，**从上述函数返回的 IP 地址将是机器上首选网络接口的地址。因此，它可能与我们预期的不同。**对于特定的需求，我们可以[找到连接到服务器](/web/20220709110357/https://www.baeldung.com/java-client-get-ip-address)的客户端的 IP 地址。

## 3.找到公共 IP 地址

类似于本地 IP 地址，我们可能想知道当前机器的公共 IP 地址。**公共 IP 地址是可从互联网到达的 IPv4 地址。**此外，它可能无法唯一识别查找地址的机器。例如，同一台路由器下的多台主机拥有相同的公有 IP 地址。

简单地说，我们可以连接到 Amazon AWS `checkip.amazonaws.com` URL 并读取响应:

```java
String urlString = "http://checkip.amazonaws.com/";
URL url = new URL(urlString);
try (BufferedReader br = new BufferedReader(new InputStreamReader(url.openStream()))) {
    return br.readLine();
}
```

这在大多数时候都很有效。然而，我们明确地依赖于一个可靠性无法保证的外部来源。因此，作为后备，我们可以使用这些 URL 中的任何一个来检索公共 IP 地址`:`

*   `https://ipv4.icanhazip.com/`
*   `http://myexternalip.com/raw`
*   `http://ipecho.net/plain`

## 4.结论

在本文中，我们学习了如何找到当前机器的 IP 地址，以及如何使用 Java 检索它们。我们还研究了检查本地和公共 IP 地址的各种方法。

和往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220709110357/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)