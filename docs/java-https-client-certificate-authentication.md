# Java HTTPS 客户端证书认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-https-client-certificate-authentication>

## 1.概观

HTTPS 是 HTTP 的扩展，允许计算机网络中两个实体之间的安全通信。HTTPS 使用 [TLS](https://web.archive.org/web/20221126231223/https://en.wikipedia.org/wiki/Transport_Layer_Security) (传输层安全)协议来实现安全连接。

**TLS 可以通过单向或双向证书验证**来实现。在单向方式中，服务器共享其公共证书，因此客户端可以验证它是受信任的服务器。替代方案是**双向验证。客户端和服务器共享它们的公共证书来验证彼此的身份**。

**本文将关注双向证书验证，其中服务器也将检查客户端的证书**。

## 2.Java 和 TLS 版本

TLS 1.3 是该协议的最新版本。这个版本性能更好，更安全。它有一个更有效的握手协议，并使用现代加密算法。

Java 从 Java 11 开始支持这个版本的协议。我们将使用这个版本来生成证书，并实现一个简单的客户机-服务器对，它使用 TLS 来相互认证。

## 3.用 Java 生成证书

由于我们正在进行双向 [TLS 认证](https://web.archive.org/web/20221126231223/https://baeldung.com/spring-tls-setup)，我们需要为客户端和服务器生成证书。

在生产环境中，建议从证书颁发机构购买证书。然而，出于测试或演示的目的，使用[自签名证书](/web/20221126231223/https://www.baeldung.com/spring-boot-https-self-signed-certificate)就足够了。对于本文，我们将使用 Java 的`keytool`来生成自签名证书。

## 3.1.服务器证书

首先，**我们生成服务器密钥存储库**:

```java
keytool -genkey -alias serverkey -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -keystore serverkeystore.p12 -storepass password -ext san=ip:127.0.0.1,dns:localhost
```

**我们使用`keytool -ext`选项来设置主题备用名称(SAN ),以定义标识服务器的本地主机名/IP 地址。**一般来说，我们可以用这个选项指定多个地址。但是，客户端将被限制使用这些地址之一来连接到服务器。

接下来，我们将证书导出到文件`server-certificate.pem`:

```java
keytool -exportcert -keystore serverkeystore.p12 -alias serverkey -storepass password -rfc -file server-certificate.pem
```

最后，**我们将服务器证书添加到客户端的信任存储库**:

```java
keytool -import -trustcacerts -file server-certificate.pem -keypass password -storepass password -keystore clienttruststore.jks
```

## 3.2.客户证书

类似地，**我们生成客户端密钥存储库**并导出其证书:

```java
keytool -genkey -alias clientkey -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -keystore clientkeystore.p12 -storepass password -ext san=ip:127.0.0.1,dns:localhost

keytool -exportcert -keystore clientkeystore.p12 -alias clientkey -storepass password -rfc -file client-certificate.pem

keytool -import -trustcacerts -file client-certificate.pem -keypass password -storepass password -keystore servertruststore.jks
```

在最后一个命令中，**我们将客户端的证书添加到服务器信任存储库**。

## 4.服务器 Java 实现

使用 java 套接字，服务器实现是简单的。`SSLSocketEchoServer`类获得一个`SSLServerSocket`来轻松支持 TLS 认证。我们只需要指定密码和协议，剩下的只是一个标准的 echo 服务器，它回复客户端发送的相同消息:

```java
public class SSLSocketEchoServer {

    static void startServer(int port) throws IOException {

        ServerSocketFactory factory = SSLServerSocketFactory.getDefault();
        try (SSLServerSocket listener = (SSLServerSocket) factory.createServerSocket(port)) {
            listener.setNeedClientAuth(true);
            listener.setEnabledCipherSuites(new String[] { "TLS_AES_128_GCM_SHA256" });
            listener.setEnabledProtocols(new String[] { "TLSv1.3" });
            System.out.println("listening for messages...");
            try (Socket socket = listener.accept()) {

                InputStream is = new BufferedInputStream(socket.getInputStream());
                byte[] data = new byte[2048];
                int len = is.read(data);

                String message = new String(data, 0, len);
                OutputStream os = new BufferedOutputStream(socket.getOutputStream());
                System.out.printf("server received %d bytes: %s%n", len, message);
                String response = message + " processed by server";
                os.write(response.getBytes(), 0, response.getBytes().length);
                os.flush();
            }
        }
    }
}
```

服务器监听客户端连接。**调用`listener.setNeedClientAuth(true)`需要客户端与服务器**共享其证书。在后台，`SSLServerSocket`实现使用 TLS 协议认证客户端。

**在我们的例子中，自签名的客户端证书在服务器信任存储中，因此套接字将接受连接**。服务器继续使用`InputStream`读取消息。然后，它使用`OuputStream`回显附加了确认的传入消息。

## 5.客户端 Java 实现

与我们对服务器所做的一样，客户端实现是一个简单的`SSLScocketClient`类:

```java
public class SSLScocketClient {

    static void startClient(String host, int port) throws IOException {

        SocketFactory factory = SSLSocketFactory.getDefault();
        try (SSLSocket socket = (SSLSocket) factory.createSocket(host, port)) {

            socket.setEnabledCipherSuites(new String[] { "TLS_AES_128_GCM_SHA256" });
            socket.setEnabledProtocols(new String[] { "TLSv1.3" });

            String message = "Hello World Message";
            System.out.println("sending message: " + message);
            OutputStream os = new BufferedOutputStream(socket.getOutputStream());
            os.write(message.getBytes());
            os.flush();

            InputStream is = new BufferedInputStream(socket.getInputStream());
            byte[] data = new byte[2048];
            int len = is.read(data);
            System.out.printf("client received %d bytes: %s%n", len, new String(data, 0, len));
        }
    }
}
```

首先，我们创建一个`SSLSocket`来建立与服务器的连接。在后台，套接字将建立 TLS 连接建立握手。**作为握手的一部分，客户端将验证服务器的证书，并检查它是否在客户端`truststore`** 中。

一旦连接成功建立，客户端就使用输出流向服务器发送消息。然后，它用输入流读取服务器的响应。

## 6.运行应用程序

要运行服务器，请打开命令窗口并运行:

```java
java -Djavax.net.ssl.keyStore=/path/to/serverkeystore.p12 \ 
  -Djavax.net.ssl.keyStorePassword=password \
  -Djavax.net.ssl.trustStore=/path/to/servertruststore.jks \ 
  -Djavax.net.ssl.trustStorePassword=password \
  com.baeldung.httpsclientauthentication.SSLSocketEchoServer
```

我们为`javax.net.ssl.` `keystore`和`javax.net.ssl.` `trustStore`指定系统属性，指向我们之前用`keytool`创建的`serverkeystore.p12`和`servertruststore.jks`文件。

要运行客户端，我们打开另一个命令窗口并运行:

```java
java -Djavax.net.ssl.keyStore=/path/to/clientkeystore.p12 \ 
  -Djavax.net.ssl.keyStorePassword=password \ 
  -Djavax.net.ssl.trustStore=/path/to/clienttruststore.jks \ 
  -Djavax.net.ssl.trustStorePassword=password \ 
  com.baeldung.httpsclientauthentication.SSLScocketClient 
```

同样，我们将`javax.net.ssl.keyStore`和`javax.net.ssl.` `trustStore`系统属性设置为指向我们之前用`keytool`生成的 `clientkeystore.p12`和`clienttruststore.jks`文件。

## 7.结论

**我们已经编写了一个简单的客户机-服务器 Java 实现，它使用服务器和客户机证书来进行双向 TLS 认证**。

我们使用`keytool`来生成自签名证书。

这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126231223/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)