# 在 Java 7 中启用 TLS v1.2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-7-tls-v12>

## 1.概观

**说到 SSL 连接，我们应该使用 tlsv 1.2。**事实上，它是 Java 8 的默认 SSL 协议。

而 Java 7 支持 TLSv1.2，**默认是 TLS v1.0，这几天太弱了。**

在本教程中，我们将讨论配置 Java 7 使用 TLSv1.2 的各种选项。

## 2.使用 Java VM 参数

如果我们使用的是 Java 1.7.0_95 或更高版本，我们可以添加`jdk.tls.client.protocols`属性作为一个`java`命令行参数来支持 TLSv1.2:

```
java -Djdk.tls.client.protocols=TLSv1.2 <Main class or the Jar file to run>
```

**但是 Java 1.7.0_95 只提供给从 Oracle** 购买支持的客户。因此，我们将在下面回顾在 Java 7 上启用 TLSv1.2 的其他选项。

## 3.使用 SSLSocket

在第一个例子中，我们将使用 [`SSLSocketFactory`](/web/20220625232401/https://www.baeldung.com/java-ssl) 来启用 TLSv1.2。

首先，**我们可以通过调用`SSLSocketFactory#` `getDefault`工厂方法创建一个默认的`SSLSocketFactory`对象。**

**然后，我们简单地将我们的主机和端口传递给`SSLSocket#` `createSocket`** :

```
SSLSocketFactory socketFactory = (SSLSocketFactory) SSLSocketFactory.getDefault();
SSLSocket sslSocket = (SSLSocket) socketFactory.createSocket(hosturl, port);
```

上面创建的默认`SSLSocket`没有任何关联的 SSL 协议。我们可以用几种方式将 SSL 协议与我们的`SSLSocket`关联起来。

在第一种方法中，**我们可以将一组受支持的 SSL 协议传递给我们的`SSLSocket `实例**上的`setEnabledProtocols`方法:

```
sslSocket.setEnabledProtocols(new String[] {"TLSv1.2"});
```

或者，我们可以使用`SSLParameters`，使用相同的数组:

```
SSLParameters params = new SSLParameters();
params.setProtocols(new String[] {"TLSv1.2"});
sslSocket.setSSLParameters(params);
```

## 4.使用 SSLContext

直接设置`SSLSocket `只会改变一个连接。我们可以使用`SSLContext `来改变我们创建`SSLSocketFactory.`的方式

所以，不要用`SSLSocketFactory#getInstance`，**，让我们用`SSLContext#getInstance, `给它“`TLSv1.2`”作为参数。我们现在可以从那里得到我们的`SSLSocketFactory `:**

```
SSLContext sslContext = SSLContext.getInstance("TLSv1.2");
sslContext.init(null, null, new SecureRandom());
SSLSocketFactory socketFactory = sslContext.getSocketFactory();
SSLSocket socket = (SSLSocket) socketFactory.createSocket(url, port);
```

作为一个小提示，在使用 SSL 时，永远记得使用`SecureRandom`。

## 5.使用`HttpsURLConnection`

当然，我们并不总是直接创建套接字。通常，我们处于应用协议级别。

那么，最后我们来看看如何在 [`HttpsURLConnection`](/web/20220625232401/https://www.baeldung.com/java-http-request) 上启用 TLSv1.2。

首先，我们需要一个`URL`的实例。让我们想象我们正在连接到[https://example.org](https://web.archive.org/web/20220625232401/https://example.org/):

```
URL url = new URL("https://" + hosturl + ":" + port);
```

现在，我们可以像以前一样设置我们的`SSLContext`:

```
SSLContext sslContext = SSLContext.getInstance("TLSv1.2"); 
sslContext.init(null, null, new SecureRandom());
```

**然后，我们的最后一步是创建连接，并为其提供一个`SSLSocketFactory` :**

```
HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
connection.setSSLSocketFactory(sslContext.getSocketFactory());
```

## 6.结论

在这篇简短的文章中，我们展示了在 Java 7 上启用 TLSv1.2 的几种方法。

本文中使用的代码示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220625232401/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)