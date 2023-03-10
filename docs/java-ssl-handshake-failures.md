# SSL 握手失败

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ssl-handshake-failures>

## 1。概述

安全套接字层(SSL)是一种加密协议，为网络通信提供安全性。在本教程中，我们将讨论可能导致 SSL 握手失败的各种情况以及如何解决。

请注意，我们对使用 JSSE 的 SSL 的介绍更详细地涵盖了 SSL 的基础知识。

## 2.术语

值得注意的是，由于安全漏洞，SSL 作为一种标准被传输层安全性(TLS)所取代。大多数编程语言，包括 Java，都有支持 SSL 和 TLS 的库。

自从 SSL 出现以来，许多产品和语言，如 OpenSSL 和 Java，都引用了 SSL，甚至在 TLS 接管以后，它们还保留着 SSL。因此，在本教程的剩余部分，我们将使用术语 SSL 来泛指加密协议。

## 3。设置

出于本教程的目的，我们将创建一个简单的服务器和客户端应用程序，使用 Java Socket API 模拟网络连接。

### 3.1。创建客户端和服务器

在 Java 中，我们可以使用 **s** **ockets 在[服务器和客户端](/web/20221205181618/https://www.baeldung.com/cs/client-vs-server-terminology)之间通过网络**建立通信通道。套接字是 Java 中 Java 安全套接字扩展(JSSE)的一部分。

让我们从定义一个简单的服务器开始:

```java
int port = 8443;
ServerSocketFactory factory = SSLServerSocketFactory.getDefault();
try (ServerSocket listener = factory.createServerSocket(port)) {
    SSLServerSocket sslListener = (SSLServerSocket) listener;
    sslListener.setNeedClientAuth(true);
    sslListener.setEnabledCipherSuites(
      new String[] { "TLS_DHE_DSS_WITH_AES_256_CBC_SHA256" });
    sslListener.setEnabledProtocols(
      new String[] { "TLSv1.2" });
    while (true) {
        try (Socket socket = sslListener.accept()) {
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            out.println("Hello World!");
        }
    }
}
```

上面定义的服务器返回消息“Hello World！”连接到一个客户端。

接下来，让我们定义一个基本客户端，我们将把它连接到我们的`SimpleServer:`

```java
String host = "localhost";
int port = 8443;
SocketFactory factory = SSLSocketFactory.getDefault();
try (Socket connection = factory.createSocket(host, port)) {
    ((SSLSocket) connection).setEnabledCipherSuites(
      new String[] { "TLS_DHE_DSS_WITH_AES_256_CBC_SHA256" });
    ((SSLSocket) connection).setEnabledProtocols(
      new String[] { "TLSv1.2" });

    SSLParameters sslParams = new SSLParameters();
    sslParams.setEndpointIdentificationAlgorithm("HTTPS");
    ((SSLSocket) connection).setSSLParameters(sslParams);

    BufferedReader input = new BufferedReader(
      new InputStreamReader(connection.getInputStream()));
    return input.readLine();
}
```

我们的客户机打印服务器返回的消息。

### 3.2。用 Java 创建证书

SSL 在网络通信中提供了保密性、完整性和真实性。证书在确定真实性方面起着重要的作用。

通常，这些证书是由证书颁发机构购买和签名的，但是对于本教程，我们将使用自签名证书。

**要实现这一点，我们可以使用`keytool, `JDK 的哪些船只:**

```java
$ keytool -genkey -keypass password \
                  -storepass password \
                  -keystore serverkeystore.jks
```

上面的命令启动一个交互式 shell 来收集证书的信息，如公用名(CN)和可分辨名(DN)。当我们提供所有相关细节时，它会生成文件`serverkeystore.jks`，该文件包含服务器的私钥及其公共证书。

注意`serverkeystore.jks `是以 Java 密钥库(JKS)格式存储的，这是 Java 专有的。**这些天，`keytool `会提醒我们应该考虑使用 PKCS#12，它也支持它。**

我们可以进一步使用`keytool `从生成的密钥库文件中提取公共证书:

```java
$ keytool -export -storepass password \
                  -file server.cer \
                  -keystore serverkeystore.jks
```

上面的命令将公钥证书作为文件`server.cer`从 keystore 导出。让我们通过将导出的证书添加到客户端的信任库中来使用它:

```java
$ keytool -import -v -trustcacerts \
                     -file server.cer \
                     -keypass password \
                     -storepass password \
                     -keystore clienttruststore.jks
```

我们现在已经为服务器生成了一个密钥库，并为客户机生成了相应的信任库。当我们讨论可能的握手失败时，我们将回顾这些生成的文件的用法。

关于 Java 的 keystore 用法的更多细节可以在我们之前的教程中找到。

## 4。SSL 握手

SSL 握手是一种机制，客户端和服务器通过这种机制建立信任和逻辑，以保护它们在网络上的连接。

这是一个非常协调的过程，理解其中的细节有助于理解为什么它经常失败，我们打算在下一节中介绍这一点。

SSL 握手的典型步骤是:

1.  客户端提供了可能使用的 SSL 版本和密码套件的列表
2.  服务器同意特定的 SSL 版本和密码套件，并使用其证书进行响应
3.  客户端从证书中提取公钥，用加密的“预主密钥”进行响应
4.  服务器使用其私钥解密“预主密钥”
5.  客户端和服务器使用交换的“预主密钥”计算“共享密钥”
6.  客户端和服务器交换消息，确认使用“共享秘密”成功加密和解密

虽然大多数步骤对于任何 SSL 握手都是相同的，但是单向和双向 SSL 之间还是有细微的差别。让我们快速回顾一下这些差异。

### 4.1。单向 SSL 中的握手

如果我们参考上面提到的步骤，第二步提到了证书交换。单向 SSL 要求客户端可以通过其公共证书信任服务器。这个**让服务器信任所有请求连接的客户端**。服务器无法从客户端请求和验证公共证书，这可能会带来安全风险。

### 4.2。双向 SSL 中的握手

对于单向 SSL，服务器必须信任所有客户端。但是，双向 SSL 也增加了服务器建立可信客户端的能力。在[双向握手](/web/20221205181618/https://www.baeldung.com/cs/handshakes)、**过程中，客户端和服务器都必须出示并接受对方的公共证书**才能成功建立连接。

## 5。握手失败场景

完成快速回顾后，我们可以更清晰地看待故障场景。

单向或双向通信中的 SSL 握手可能由于多种原因而失败。我们将逐一分析这些原因，模拟故障，并了解如何避免这种情况。

在每个场景中，我们将使用我们之前创建的`SimpleClient`和`SimpleServer`。

### 5.1。缺少服务器证书

让我们试着运行`SimpleServer`并通过`SimpleClient`连接它。当我们期待看到“你好，世界！”，我们会看到一个例外:

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: 
  Received fatal alert: handshake_failure
```

这表明出了问题。上面的`SSLHandshakeException`抽象地说，**表示客户端在连接到服务器时没有收到任何证书。**

为了解决这个问题，我们将使用之前生成的密钥库，将它们作为系统属性传递给服务器:

```java
-Djavax.net.ssl.keyStore=serverkeystore.jks -Djavax.net.ssl.keyStorePassword=password
```

值得注意的是，密钥库文件路径的系统属性应该是绝对路径，或者密钥库文件应该放在调用 Java 命令来启动服务器的同一个目录中。**密钥库的 Java 系统属性不支持相对路径。**

这有助于我们获得预期的结果吗？让我们在下一小节中找出答案。

### 5.2。不受信任的服务器证书

当我们使用前面小节中的更改再次运行`SimpleServer`和`SimpleClient`时，我们得到的输出是什么:

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: 
  sun.security.validator.ValidatorException: 
  PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: 
  unable to find valid certification path to requested target
```

嗯，它并不完全像我们预期的那样工作，但看起来它失败的原因不同。

这个特别的失败是由于我们的服务器使用了一个不是由证书颁发机构(CA)签名的`self-signed`证书。

实际上，只要证书是由默认信任库中之外的人签署的，我们就会看到这个错误。JDK 的默认信任库通常附带有关正在使用的常见 ca 的信息。

为了解决这个问题，我们必须强制`SimpleClient`信任`SimpleServer`提供的证书。让我们使用前面生成的信任库，将它们作为系统属性传递给客户机:

```java
-Djavax.net.ssl.trustStore=clienttruststore.jks -Djavax.net.ssl.trustStorePassword=password
```

请注意，这不是一个理想的解决方案。在理想的情况下，我们不应该使用自签名证书，而应该使用由客户端默认信任的证书颁发机构(CA)认证的证书。

让我们转到下一小节，看看我们现在是否得到了预期的输出。

### 5.3。缺少客户端证书

让我们应用前面小节中的更改，再次尝试运行 SimpleServer 和 SimpleClient:

```java
Exception in thread "main" java.net.SocketException: 
  Software caused connection abort: recv failed
```

同样，这不是我们所期望的。这里的`SocketException` 告诉我们服务器不能信任客户端。这是因为我们已经建立了一个双向 SSL。在我们的`SimpleServer `中，我们有:

```java
((SSLServerSocket) listener).setNeedClientAuth(true);
```

**上面的代码表示通过公共证书进行客户端身份验证需要一个`SSLServerSocket` 。**

我们可以为客户机创建一个密钥库，并为服务器创建一个相应的信任库，其方式类似于我们在创建以前的密钥库和信任库时使用的方式。

我们将重新启动服务器，并向其传递以下系统属性:

```java
-Djavax.net.ssl.keyStore=serverkeystore.jks \
    -Djavax.net.ssl.keyStorePassword=password \
    -Djavax.net.ssl.trustStore=clienttruststore.jks \
    -Djavax.net.ssl.trustStorePassword=password
```

然后，我们将通过传递这些系统属性来重新启动客户端:

```java
-Djavax.net.ssl.keyStore=serverkeystore.jks \
    -Djavax.net.ssl.keyStorePassword=password \
    -Djavax.net.ssl.trustStore=clienttruststore.jks \
    -Djavax.net.ssl.trustStorePassword=password
```

最后，我们得到了我们想要的输出:

```java
Hello World!
```

### 5.4。不正确的证书

除了上述错误之外，握手可能由于与我们如何创建证书相关的各种原因而失败。一个常见的错误与不正确的 CN 有关。让我们研究一下我们之前创建的服务器密钥库的详细信息:

```java
keytool -v -list -keystore serverkeystore.jks
```

当我们运行上面的命令时，我们可以看到密钥库的详细信息，特别是所有者:

```java
...
Owner: CN=localhost, OU=technology, O=baeldung, L=city, ST=state, C=xx
...
```

该证书所有者的 CN 设置为 localhost。所有者的 CN 必须与服务器的主机完全匹配。如果有任何不匹配，将导致`SSLHandshakeException`。

让我们尝试用 CN 重新生成服务器证书，而不是本地主机。当我们现在使用重新生成的证书运行`SimpleServer`和`SimpleClient`时，它立即失败:

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: 
    java.security.cert.CertificateException: 
    No name matching localhost found
```

上面的异常跟踪清楚地表明，客户端需要一个名称为 localhost 的证书，但它没有找到。

请注意，默认情况下， **JSSE 不会强制进行主机名验证。**我们已经通过明确使用 HTTPS 在`SimpleClient`中启用了主机名验证:

```java
SSLParameters sslParams = new SSLParameters();
sslParams.setEndpointIdentificationAlgorithm("HTTPS");
((SSLSocket) connection).setSSLParameters(sslParams);
```

主机名验证是失败的一个常见原因，一般来说，为了更好的安全性，应该总是强制执行。有关主机名验证及其在 TLS 安全性中的重要性的详细信息，请参考本文中的[。](https://web.archive.org/web/20221205181618/https://tersesystems.com/blog/2014/03/23/fixing-hostname-verification/)

### 5.5。不兼容的 SSL 版本

当前，有各种加密协议在运行，包括不同版本的 SSL 和 TLS。

如前所述，由于其加密强度，SSL 通常已经被 TLS 取代。加密协议和版本是客户端和服务器在握手期间必须达成一致的附加元素。

例如，如果服务器使用 SSL3 加密协议，而客户端使用 TLS1.3 加密协议，则它们无法就加密协议达成一致，将会生成一个`SSLHandshakeException`。

在我们的`SimpleClient`中，让我们将协议更改为与为服务器设置的协议不兼容的协议:

```java
((SSLSocket) connection).setEnabledProtocols(new String[] { "TLSv1.1" });
```

当我们再次运行我们的客户端时，我们将得到一个`SSLHandshakeException`:

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: 
  No appropriate protocol (protocol is disabled or cipher suites are inappropriate)
```

这种情况下的异常跟踪是抽象的，并没有告诉我们确切的问题。为了解决这些类型的问题，有必要验证客户端和服务器是否使用相同或兼容的加密协议。

### 5.6。不兼容的密码套件

客户端和服务器还必须就它们将用来加密消息的密码套件达成一致。

在握手过程中，客户端将提供一个可能使用的密码列表，服务器将从列表中选择一个密码作为响应。如果无法选择合适的密码，服务器将生成一个`SSLHandshakeException `。

在我们的`SimpleClient`中，让我们将密码套件更改为与我们的服务器使用的密码套件不兼容:

```java
((SSLSocket) connection).setEnabledCipherSuites(
  new String[] { "TLS_RSA_WITH_AES_128_GCM_SHA256" });
```

当我们重启客户端时，我们将得到一个`SSLHandshakeException`:

```java
Exception in thread "main" javax.net.ssl.SSLHandshakeException: 
  Received fatal alert: handshake_failure
```

同样，异常跟踪非常抽象，并没有告诉我们确切的问题。解决此类错误的方法是验证客户端和服务器使用的已启用密码套件，并确保至少有一个通用套件可用。

通常，客户端和服务器被配置为使用各种各样的密码套件，因此这种错误不太可能发生。如果我们遇到此错误，通常是因为服务器被配置为使用选择性很强的密码。出于安全原因，服务器可以选择实施一组选择性的密码。

## 6。结论

在本教程中，我们学习了如何使用 Java 套接字设置 SSL。然后我们讨论了单向和双向 SSL 的 SSL 握手。最后，我们讨论了 SSL 握手失败的可能原因，并讨论了解决方案。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221205181618/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)