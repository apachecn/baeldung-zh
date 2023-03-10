# 在核心 Java 中通过代理服务器连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-connect-via-proxy-server>

## 1。简介

代理服务器充当客户端应用程序和其他服务器之间的中介。在企业环境中，我们经常使用它们来帮助控制用户消费的内容，通常是跨网络边界的。

在本教程中，我们将看看**如何通过 Java** 中的代理服务器进行连接。

首先，我们将探索更老、更全球化的方法，它是 JVM 范围的，并配置了系统属性。之后，我们将引入`Proxy`类，它允许基于每个连接进行配置，从而给了我们更多的控制。

## 2。设置

要运行本文中的示例，我们需要访问代理服务器。Squid 是一个流行的实现，适用于大多数操作系统。Squid 的默认配置对于我们的大多数例子来说已经足够好了。

## 3。使用全局设置

Java 公开了一组系统属性，可以用来配置 JVM 范围的行为。如果适合用例，这种“一刀切”的方法通常是最容易实现的。

当调用 JVM 时，我们可以**从命令行设置所需的属性。作为替代，我们也可以通过在运行时调用**来设置它们。

### 3.1。可用的系统属性

Java 为 HTTP、HTTPS、FTP 和 SOCKS 协议提供代理处理程序。可以将每个处理程序的代理定义为主机名和端口号:

*   `**http.proxyHost**`–HTTP 代理服务器的主机名
*   **`http.proxyPort`**–HTTP 代理服务器的端口号–属性是可选的，如果不提供，默认为 80
*   **`http.nonProxyHosts`**–应该绕过代理的主机模式的管道分隔(“|”)列表–如果设置，则适用于 HTTP 和 HTTPS 处理程序
*   **`socksProxyHost`**–SOCKS 代理服务器的主机名
*   `**socksProxyPort**` –SOCKS 代理服务器的端口号

如果指定`nonProxyHosts`，主机模式可能以通配符(" * ")开始或结束。在 Windows 平台上，可能需要对分隔符“|”进行转义。所有可用的代理相关系统属性的详尽列表可以在 [Oracle 关于网络属性的官方 Java 文档](https://web.archive.org/web/20220926181620/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/doc-files/net-properties.html)中找到。

### 3.2。通过命令行参数设置

我们可以通过将设置作为系统属性传入来在命令行上定义代理:

```java
java -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=3128 com.baeldung.networking.proxies.CommandLineProxyDemo
```

当以这种方式启动一个流程时，我们能够简单地在`URL`上使用`openConnection()`，而不需要任何额外的工作:

```java
URL url = new URL(RESOURCE_URL);
URLConnection con = url.openConnection();
```

### 3.3。使用`System.setProperty(String, String)`设置

如果我们不能在命令行上设置代理属性，我们可以在程序中调用`System.setProperty()`来设置它们:

```java
System.setProperty("http.proxyHost", "127.0.0.1");
System.setProperty("http.proxyPort", "3128");

URL url = new URL(RESOURCE_URL);
URLConnection con = url.openConnection();
// ...
```

如果我们稍后手动取消设置相关的系统属性，则代理将不再使用:

```java
System.setProperty("http.proxyHost", null);
```

### 3.4。全局配置的局限性

尽管使用带有系统属性的全局配置很容易实现，但是这种方法**限制了我们可以做的事情，因为这些设置适用于整个 JVM** 。出于这个原因，为特定协议定义的设置在 JVM 的生命周期内是活动的，或者直到它们被取消设置。

为了绕过这个限制，根据需要打开和关闭设置可能很有诱惑力。为了在多线程程序中安全地做到这一点，有必要引入一些措施来防止并发问题。

作为替代，代理 API 对代理配置提供了更细粒度的控制。

## 4。使用`Proxy` API

`Proxy`类为我们提供了一种基于每个连接配置代理的灵活方式。如果存在任何现有的 JVM 范围的代理设置，使用`Proxy`类的基于连接的代理设置将覆盖它们。

我们可以通过`Proxy.Type`定义三种类型的代理:

*   **HTTP**–使用 HTTP 协议的代理
*   **SOCKS**–使用 SOCKS 协议的代理
*   **直接**–明确配置的没有代理的直接连接

### 4.1。使用 HTTP 代理

为了使用 HTTP 代理，我们首先用`Proxy`和类型`Proxy.Type.HTTP`T6**包装一个`SocketAddress`实例。接下来，我们简单地将`Proxy`实例传递给`URLConnection.openConnection():`**

```java
URL weburl = new URL(URL_STRING);
Proxy webProxy 
  = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("127.0.0.1", 3128));
HttpURLConnection webProxyConnection 
  = (HttpURLConnection) weburl.openConnection(webProxy);
```

简单地说，这意味着我们将连接到`URL_STRING`，然后通过托管在`127.0.0.1:3128`的代理服务器路由该连接。

### 4.2。使用直接代理

我们可能需要直接连接到主机。在这种情况下，我们可以通过使用静态的`Proxy.NO_PROXY` 实例，显式地**绕过一个可以全局配置的代理。在幕后，API 为我们构建了一个新的`Proxy`实例，使用`Proxy.Type.DIRECT` 作为类型`:`**

```java
HttpURLConnection directConnection 
  = (HttpURLConnection) weburl.openConnection(Proxy.NO_PROXY);
```

基本上，如果没有全局配置的代理，那么这与不带参数调用`openConnection() `是一样的。

### 4.3。使用 SOCKS 代理

在使用`URLConnection.`时，使用 SOCKS 代理类似于 HTTP 变体，我们从使用`Proxy.Type.SOCKS` 类型的**用`Proxy`包装`SocketAddress`实例开始。之后，我们将`Proxy`实例传递给`URLConnection.openConnection`:**

```java
Proxy socksProxy 
  = new Proxy(Proxy.Type.SOCKS, new InetSocketAddress("127.0.0.1", 1080));
HttpURLConnection socksConnection 
  = (HttpURLConnection) weburl.openConnection(socksProxy); 
```

当连接到 TCP 套接字时，**也可以使用 SOCKS 代理。首先，我们使用`Proxy`实例来构造一个`Socket`。之后，我们**将目的地`SocketAddress`实例传递给`Socket.connect()`** :**

```java
Socket proxySocket = new Socket(socksProxy);
InetSocketAddress socketHost 
  = new InetSocketAddress(SOCKET_SERVER_HOST, SOCKET_SERVER_PORT);
proxySocket.connect(socketHost);
```

## 5。结论

在本文中，我们研究了如何在核心 Java 中使用代理服务器。

首先，我们研究了使用系统属性通过代理服务器进行连接的更古老、更全球化的方式。然后，我们看到了如何使用`Proxy`类，它在通过代理服务器连接时提供了细粒度的控制。

和往常一样，本文中使用的所有源代码都可以在 GitHub 上的[中找到。](https://web.archive.org/web/20220926181620/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)