# Java 中的 SSL 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ssl>

## 1。概述

在本教程中，我们将介绍 SSL，并探索如何使用 JSSE (Java 安全套接字扩展)API 在 Java 中使用它。

## 2。简介

简单地说， **`Secured Socket Layer (SSL) `实现了两方**之间的安全连接，通常是客户端和服务器。

SSL 在通过网络连接运行的两个设备之间提供安全通道。SSL 的一个常见例子是支持 web 浏览器和 web 服务器之间的安全通信。

在这种特定情况下，web 浏览器将使用 HTTPS(代表安全)连接来访问不同 web 服务器提供的资源。

**SSL 是支持三个主要信息安全原则所必需的:**

*   **加密**:保护各方之间的数据传输

*   **认证**:确保我们连接的服务器确实是正确的服务器

*   **数据完整性**:保证所请求的数据是有效交付的

Java 提供了几个基于安全性的 API，帮助开发人员建立与客户端的安全连接，以加密格式接收和发送消息:

*   Java 安全套接字扩展(JSSE)
*   Java 加密体系结构(JCA)
*   Java 加密扩展(JCE)

在下一节中，我们将介绍 Java 用来实现安全通信的安全套接字扩展。

## 3.JSSE API

Java 安全 API 广泛利用了`Factory`设计模式。

事实上，一切都是通过 JSSE 的工厂来实例化的。

### 3.1.`SSLSocketFactory`

`javax.net.ssl.SSLSocketFactory`用于创建`SSLSocket`对象。

这个类包含三组 API。

第一组包含一个静态的`getDefault()`方法，用于检索默认实例，该方法反过来可以创建 *SSLSocket* 实例。

第二组由五种方法组成，可用于创建`SSLSocket`实例:

*   `Socket createSocket(String host, int port)`
*   `Socket createSocket(String host, int port, InetAddress clientHost, int clientPort)`
*   `Socket createSocket(InetAddress host, int port)`
*   `Socket createSocket(InetAddress host, int port, InetAddress clientHost, int clientPort)`
*   `Socket createSocket(Socket socket, String host, int port, boolean autoClose)`

我们可以通过获取默认实例或者使用包含获取`SSLSocketFactory`实例的方法的`javax.net.ssl.` SSLContext 对象来直接使用这个类。

### 3.2.`SSLSocket`

这个类扩展了`Socket`类并提供了安全套接字。这样的套接字是普通的流套接字。

此外，它们还在底层网络传输协议上增加了一层安全保护。

`SSLSocket`实例在指定端口建立到指定主机的 SSL 连接。

这允许将连接的客户端绑定到给定的地址和端口。

### 3.3.`SSLServerSocketFactory`

`SSLServerSocketFactory`类与`SSLSocketFactory`非常相似，不同之处在于它创建了`SSLServerSocket`实例来代替`SSLSocket`实例。

通过相似性，这些方法被称为`createServerSocket`，类似于`SSLSocketFactory`类。

### 3.4.`SSLServerSocket`

`SSLServerSocket`类类似于`SSLSocket`类。SSLServerSocket 类上的方法是`SSLSocket`类方法的子集。它们作用于 SSL 连接的另一端

## 4.SSL 示例

让我们提供一个如何创建到服务器的安全连接的示例:

```java
String host = getHost(...);
Integer port = getPort(...);
SSLSocketFactory sslsocketfactory = SSLSocketFactory.getDefault();
SSLSocket sslsocket = (SSLSocket) sslsocketfactory
  .createSocket(host, port);
InputStream in = sslsocket.getInputStream();
OutputStream out = sslsocket.getOutputStream();

out.write(1);
while (in.available() > 0) {
    System.out.print(in.read());
}

System.out.println("Secured connection performed successfully"); 
```

如果我们得到错误`“javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target while establishing the SSL connection”`，**，这表明我们在 Java 信任库中没有我们试图连接的服务器的公共证书。**

信任库是包含可信证书的文件，Java 使用该文件来验证安全连接。

为了解决这个问题，我们有几个选择:

*   将服务器的公共证书添加到 Java 使用的默认 **`cacerts`信任库**中。启动 SSL 连接时
*   javax.net 设定了。`ssl`。`trustStore` 指向信任库文件的环境变量，这样应用程序就可以获取包含我们正在连接的服务器的公共证书的文件。

将新证书安装到 Java 默认信任库中的步骤如下:

1.  从服务器提取证书:`openssl s_client -connect server:443`
2.  使用 keytool 将证书导入信任库:`keytool -import -alias alias.server.com -keystore $JAVA_HOME/jre/lib/security/cacerts`

一旦我们完成了这些，我们应该能够再次运行这个例子并获得`Secured connection performed successfully`消息。

## 5.结论

在本文中，我们介绍了 SSL 和为 Java 实现 SSL 的 JSSE API。通过使用 SSL 和 JSSE，我们可以使我们的 Java 应用程序以及应用程序之间和应用程序内部的通信更加安全。

和往常一样，本文中的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221126223104/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)