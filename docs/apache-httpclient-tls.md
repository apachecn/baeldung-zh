# 如何在 Apache HttpClient 中设置 TLS 版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-httpclient-tls>

## 1.介绍

Apache HttpClient 是一个低级、轻量级的客户端 HTTP 库，用于与 HTTP 服务器通信。在本教程中，我们将学习如何在使用 [HttpClient](/web/20220529030614/https://www.baeldung.com/httpclient-guide) 时配置支持的传输层安全性(TLS)版本。我们将首先概述 TLS 版本协商在客户机和服务器之间是如何工作的。之后，我们将看看使用 HttpClient 时配置受支持的 TLS 版本的三种不同方式**。**

## 2.TLS 版本协商

TLS 是一种互联网协议，可在双方之间提供安全、可信的通信。它封装了像 HTTP 这样的应用层协议。自 1999 年首次发布以来，TLS 协议已经过多次修订。**因此，对于客户端和服务器来说，在建立新的连接时，首先就使用哪个版本的 TLS 达成一致是很重要的。**客户端和服务器交换 hello 消息后，TLS 版本达成一致:

1.  客户端发送支持的 TLS 版本列表。
2.  服务器选择一个并在响应中包含所选版本。
3.  客户端和服务器使用所选版本继续连接设置。

由于存在[降级攻击](https://web.archive.org/web/20220529030614/https://en.wikipedia.org/wiki/Downgrade_attack)的风险，正确配置 web 客户端支持的 TLS 版本非常重要。**注意，为了使用最新版本的 TLS (TLS 1.3)，我们必须使用 Java 11 或更高版本。**

## 3.静态设置 TLS 版本

### 3.1.`SSLConnectionSocketFactory`

让我们使用由`HttpClients#custom`构建器方法公开的`HttpClientBuilder`来定制我们的`HTTPClient`配置。这个构建器模式允许我们传入我们自己的`SSLConnectionSocketFactory`，它将使用所需的一组受支持的 TLS 版本进行实例化:

```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
  SSLContexts.createDefault(),
  new String[] { "TLSv1.2", "TLSv1.3" },
  null,
  SSLConnectionSocketFactory.getDefaultHostnameVerifier());

CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
```

返回的`Httpclient`对象现在可以执行 HTTP 请求了。通过在`SSLConnectionSocketFactory` 构造函数中显式设置支持的协议，客户端将只支持 TLS 1.2 或 TLS 1.3 上的通信。注意，在 Apache HttpClient 之前的版本中，这个类被称为`SSLSocketFactory`。

### 3.2.Java 运行时参数

或者，我们可以使用 Java 的`https.protocols`系统属性来配置支持的 TLS 版本。这种方法避免了将值硬编码到应用程序代码中。相反，我们将配置`HttpClient`在建立连接时使用系统属性。HttpClient API 提供了两种方法来实现这一点。第一种是经由`HttpClients#createSystem`:

```java
CloseableHttpClient httpClient = HttpClients.createSystem();
```

如果需要更多的客户端配置，我们可以使用构建器方法:

```java
CloseableHttpClient httpClient = HttpClients.custom().useSystemProperties().build();
```

这两种方法都告诉`HttpClient`在连接配置期间使用系统属性。这允许我们在应用程序运行时使用命令行参数设置所需的 TLS 版本。例如:

```java
$ java -Dhttps.protocols=TLSv1.1,TLSv1.2,TLSv1.3 -jar webClient.jar
```

## 4.动态设置 TLS 版本

还可以根据主机名和端口等连接细节来设置 TLS 版本。我们将扩展`SSLConnectionSocketFactory`并覆盖`prepareSocket`方法。客户端在启动新连接之前调用`prepareSocket`方法。**这将让我们决定基于每个连接使用哪些 TLS 协议。**也可以启用对旧 TLS 版本的支持，但前提是远程主机有特定的子域:

```java
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(SSLContexts.createDefault()){

    @Override
    protected void prepareSocket(SSLSocket socket) {

        String hostname = socket.getInetAddress().getHostName();
        if (hostname.endsWith("internal.system.com")){
            socket.setEnabledProtocols(new String[] { "TLSv1", "TLSv1.1", "TLSv1.2", "TLSv1.3" });
        }
        else {
            socket.setEnabledProtocols(new String[] {"TLSv1.3"});
        }
    }
};<br />
CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build(); 
```

在上面的例子中，`prepareSocket`方法首先获取`SSLSocket`将要连接的远程主机名。**主机名随后被用于确定 TLS 协议的启用。**现在，我们的 HTTP 客户端将对每个请求执行 TLS 1.3，除非目的主机名是* `.internal.example.com.`的形式，并且能够在创建新的`SSLSocket`之前插入自定义逻辑，我们的应用程序现在可以自定义 TLS 通信细节。

## 5.结论

在本文中，我们研究了在使用 Apache HttpClient 库时配置受支持的 TLS 版本的三种不同方式。我们已经看到了如何为所有连接设置 TLS 版本，或者在每个连接的基础上设置。本文中使用的代码示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220529030614/https://github.com/eugenp/tutorials/tree/master/apache-httpclient-2)