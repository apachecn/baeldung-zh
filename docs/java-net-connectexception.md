# 处理 java.net.ConnectException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-net-connectexception>

## 1.介绍

在这个快速教程中，我们将讨论`java.net.ConnectException`的一些可能原因。然后，我们将展示如何借助两个公开可用的命令和一个小的 Java 示例来检查连接。

## 2.什么原因造成的`java.net.ConnectException`

`java.net.ConnectException`异常是与网络相关的最常见的 Java 异常之一。**当我们从客户端应用程序到服务器建立 TCP 连接时，我们可能会遇到这种情况。**由于它是一个检查异常，我们应该在代码中的`try-catch`块中正确处理它。

这种异常有许多可能的原因:

*   我们试图连接的服务器没有启动，因此，我们无法获得连接
*   我们用来连接服务器的主机和端口组合可能不正确
*   一个[防火墙](/web/20221129020246/https://www.baeldung.com/cs/firewalls-intro)可能会阻止来自特定 IP 地址或端口的连接

## 3.**以编程方式捕获异常**

我们通常使用`java.net.Socket`类以编程方式连接到服务器。为了建立 TCP 连接，我们必须**确保我们连接到正确的主机和端口组合:**

```java
String host = "localhost";
int port = 5000;

try {
    Socket clientSocket = new Socket(host, port);

    // do something with the successfully opened socket

    clientSocket.close();
} catch (ConnectException e) {
    // host and port combination not valid
    e.printStackTrace();
} catch (Exception e) {
    e.printStackTrace();
}
```

如果主机和端口组合不正确，`Socket`将抛出一个 `java.net.ConnectException.`。在上面的代码示例中，我们打印堆栈跟踪以更好地确定哪里出错了。没有可见的堆栈跟踪意味着我们的代码成功地连接到服务器。

在这种情况下，我们应该在继续之前检查连接细节。我们还应该检查我们的防火墙没有阻止连接。

## 4.**使用命令行界面检查连接**

通过 CLI，我们可以**使用` [ping](https://web.archive.org/web/20221129020246/https://linux.die.net/man/8/ping)`命令来检查我们试图连接的服务器是否正在运行**。

例如，我们可以检查 Baeldung 服务器是否已启动:

```java
ping baeldung.com
```

如果 Baeldung 服务器正在运行，我们应该会看到关于发送和接收的包的信息。

```java
PING baeldung.com (104.18.63.78): 56 data bytes
64 bytes from 104.18.63.78: icmp_seq=0 ttl=57 time=7.648 ms
64 bytes from 104.18.63.78: icmp_seq=1 ttl=57 time=14.493 ms
```

**`telnet` 是另一个有用的 CLI 工具，我们可以用它来连接到指定的主机或 IP 地址。**此外，我们还可以传递一个我们想要测试的精确端口。否则，将使用默认端口 23。

例如，如果我们想在端口 80 上连接到 Baeldung 网站，我们可以运行:

```java
telnet baeldung.com 80
```

值得注意的是，`ping`和`telnet`命令可能并不总是有效——即使我们试图访问的服务器正在运行，并且我们使用了正确的主机和端口组合。这在生产系统中是常见的情况，生产系统通常受到严密保护，以防止未经授权的访问。此外，防火墙阻止特定的互联网流量可能是另一个失败的原因。

## 5.结论

在这个快速教程中，我们讨论了常见的 Java 网络异常，`java.net.ConnectException`。

我们从解释该异常的可能原因开始。我们展示了如何以编程方式捕获异常，以及两个有用的 CLI 命令，它们可能有助于确定原因。

和往常一样，本文中显示的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129020246/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)