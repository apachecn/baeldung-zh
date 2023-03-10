# 爪哇 SASL 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sasl>

 ![](img/b0b6501d3dcfe3cf751d23c2a9f8234c.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220523231806/https://www.baeldung.com/lightrun-n-security)

## 1.概观

在本教程中，我们将介绍简单认证和安全层(SASL)的基础知识。我们将了解 Java 如何支持采用 SASL 来保护通信。

在这个过程中，我们将使用简单的客户端和服务器通信，用 SASL 保护它。

## 2.什么是`SASL`？

SASL 是互联网协议中认证和数据安全的框架。它旨在将互联网协议从特定的认证机制中分离出来。随着我们的深入，我们会更好地理解这个定义的某些部分。

通信中的安全需求是不言而喻的。让我们试着**在客户端和服务器通信**的上下文中理解这一点。通常，客户端和服务器通过网络交换数据。双方能够相互信任并安全地发送数据是必不可少的。

### 2.1.`SASL`在哪里？

在应用程序中，我们可能使用 SMTP 发送电子邮件，使用 LDAP 访问目录服务。但是这些协议中的每一个都可能支持另一种认证机制，比如 Digest-MD5 或 Kerberos。

如果有一种方法可以让协议更明确地交换认证机制，那会怎么样？这正是 SASL 参与进来的地方。支持 SASL 的协议可以无一例外地支持任何 SASL 机制。

因此，**应用程序可以协商合适的机制**，并采用该机制进行认证和安全通信。

### 2.2.`SASL`是如何工作的？

现在，我们已经看到了 SASL 在整个安全方案中的位置，让我们了解它是如何工作的。

SASL **是一个挑战-响应框架**。这里，服务器向客户机发出一个挑战，客户机根据这个挑战发送一个响应。挑战和响应是任意长度的字节数组，因此可以携带任何特定于机制的数据。

[![](img/b42767835d51b249721fea19628b340e.png)](/web/20220523231806/https://www.baeldung.com/wp-content/uploads/2019/09/SASL-Exchange.jpg)

这种**交换可以持续多次迭代**，并最终在服务器不再发出挑战时结束。

此外，客户端和服务器可以在认证后协商安全层。所有后续通信都可以利用这个安全层。但是，请注意，某些机制可能只支持身份验证。

这里需要理解的是，SASL **只为挑战和响应**数据的交换提供了一个框架。它没有提到任何关于数据本身或它们如何交换的内容。这些细节留给采用 SASL 的应用程序。

## 3.Java 中的 SASL 支持

Java 中有一些 APIs】支持用 SASL 开发客户端和服务器端应用。API 不依赖于实际的机制本身。使用 Java SASL API 的应用程序可以根据所需的安全特性选择一种机制。

### 3.1.Java SASL API

作为包“javax.security.sasl”的一部分，需要注意的关键接口是`SaslServer`和`SaslClient`。

`SaslServer`代表 SASL 的服务器端机制。

让我们看看如何实例化一个`SaslServer`:

```java
SaslServer ss = Sasl.createSaslServer(
  mechanism, 
  protocol, 
  serverName, 
  props, 
  callbackHandler);
```

我们使用工厂类`Sasl`来实例化`SaslServer.` 。方法`createSaslServer`接受几个参数:

*   `mechanism`–SASL 支持的机制的 IANA 注册名称
*   `protocol`–正在进行验证的协议的名称
*   `serverName`–服务器的完全限定主机名
*   `props`–用于配置认证交换的一组属性
*   `callbackHandler`–所选机制使用的回调处理程序，用于获取更多信息

在上面的内容中，只有前两个是强制的，其余的可以为空。

`SaslClient`代表 SASL 的客户端机制。让我们看看如何实例化一个`SaslClient`:

```java
SaslClient sc = Sasl.createSaslClient(
  mechanisms, 
  authorizationId, 
  protocol, 
  serverName, 
  props,
  callbackHandler);
```

这里，我们再次使用工厂类`Sasl`来实例化我们的`SaslClient`。`createSaslClient`接受的参数列表和以前几乎一样。

然而，有一些微妙的区别:

*   这里，这是一个可供尝试的机制列表
*   `authorizationId`–这是用于授权的协议相关识别

其余的参数在含义和可选性上是相似的。

### 3.2.Java SASL 安全提供程序

在 Java SASL API 的下面是提供安全特性的实际机制。这些机制的**实现由在 [Java 加密体系结构](https://web.archive.org/web/20220523231806/https://docs.oracle.com/javase/9/security/java-cryptography-architecture-jca-reference-guide.htm) (JCA)注册的安全提供者**提供。

可以有多个向 JCA 注册的安全提供者。这些**中的每一个都可以支持一个或多个 SASL 机制**。

SunSASL 将 Java 作为安全提供者提供，默认情况下注册为 JCA 提供者。但是，这可能会被任何其他可用的提供程序删除或重新排序。

此外，**总是有可能提供一个定制的安全提供者**。这将要求我们实现接口`SaslClient`和`SaslServer`。这样做，我们也可以实现我们的自定义安全机制！

## 4.SASL 举了一个例子

既然我们已经看到了如何创建一个`SaslServer`和一个`SaslClient`，是时候了解如何使用它们了。我们将开发客户端和服务器组件。这些将反复交换挑战和响应以实现认证。在这个简单的例子中，我们将使用 DIGEST-MD5 机制。

### 4.1.客户端和服务器`CallbackHandler`

正如我们前面看到的，我们需要提供`CallbackHandler`到`SaslServer`和`SaslClient`的实现。现在，`CallbackHandler`是一个简单的接口，它定义了一个方法— `handle`。这个方法接受一个数组`Callback`。

这里， **`Callback`给出了安全机制从调用应用**收集认证数据的方法。例如，安全机制可能需要用户名和密码。有很多像`NameCallback`和`PasswordCallback`这样的`Callback`实现可供使用。

首先，让我们看看如何为服务器定义一个`CallbackHandler`:

```java
public class ServerCallbackHandler implements CallbackHandler {
    @Override
    public void handle(Callback[] cbs) throws IOException, UnsupportedCallbackException {
        for (Callback cb : cbs) {
            if (cb instanceof AuthorizeCallback) {
                AuthorizeCallback ac = (AuthorizeCallback) cb;
                //Perform application-specific authorization action
                ac.setAuthorized(true);
            } else if (cb instanceof NameCallback) {
                NameCallback nc = (NameCallback) cb;
                //Collect username in application-specific manner
                nc.setName("username");
            } else if (cb instanceof PasswordCallback) {
                PasswordCallback pc = (PasswordCallback) cb;
                //Collect password in application-specific manner
                pc.setPassword("password".toCharArray());
            } else if (cb instanceof RealmCallback) { 
                RealmCallback rc = (RealmCallback) cb; 
                //Collect realm data in application-specific manner 
                rc.setText("myServer"); 
            }
        }
    }
}
```

现在，让我们看看`Callbackhandler`的客户端:

```java
public class ClientCallbackHandler implements CallbackHandler {
    @Override
    public void handle(Callback[] cbs) throws IOException, UnsupportedCallbackException {
        for (Callback cb : cbs) {
            if (cb instanceof NameCallback) {
                NameCallback nc = (NameCallback) cb;
                //Collect username in application-specific manner
                nc.setName("username");
            } else if (cb instanceof PasswordCallback) {
                PasswordCallback pc = (PasswordCallback) cb;
                //Collect password in application-specific manner
                pc.setPassword("password".toCharArray());
            } else if (cb instanceof RealmCallback) { 
                RealmCallback rc = (RealmCallback) cb; 
                //Collect realm data in application-specific manner 
                rc.setText("myServer"); 
            }
        }
    }
}
```

澄清一下，我们正在**遍历`Callback`数组，并且只处理特定的**。我们必须处理的是特定于正在使用的机制，这里是 DIGEST-MD5。

### 4.2.SASL 认证

因此，我们已经编写了我们的客户端和服务器`CallbackHandler`。我们还为 DIGEST-MD5 机制实例化了`SaslClient`和`SaslServer`。

现在是时候看看他们的行动了:

```java
@Test
public void givenHandlers_whenStarted_thenAutenticationWorks() throws SaslException {
    byte[] challenge;
    byte[] response;

    challenge = saslServer.evaluateResponse(new byte[0]);
    response = saslClient.evaluateChallenge(challenge);

    challenge = saslServer.evaluateResponse(response);
    response = saslClient.evaluateChallenge(challenge);

    assertTrue(saslServer.isComplete());
    assertTrue(saslClient.isComplete());
}
```

让我们试着理解这里发生了什么:

*   首先，我们的客户机从服务器获得默认的挑战
*   然后，客户端评估挑战并准备响应
*   这种挑战-响应的交换继续一个周期
*   在这个过程中，客户机和服务器利用回调处理程序来收集该机制所需的任何附加数据
*   我们的身份验证到此结束，但实际上，它可以迭代多个周期

典型的挑战和响应字节数组的交换发生在网络上。但是，这里为了简单起见，我们假设本地通信。

### 4.3.SASL 保密通信

正如我们前面所讨论的，SASL 是一个能够支持安全通信而不仅仅是身份验证的框架。然而，**这只有在底层机制支持的情况下才有可能**。

首先，让我们先检查我们是否能够协商一个安全的通信:

```java
String qop = (String) saslClient.getNegotiatedProperty(Sasl.QOP);

assertEquals("auth-conf", qop);
```

在这里， **QOP 代表保护质量**。这是客户端和服务器在身份验证期间协商的内容。值“auth-int”表示验证和完整性。而值“auth-conf”表示认证、完整性和机密性。

一旦我们有了安全层，我们就可以利用它来保护我们的通信。

让我们看看如何保护客户端中的传出通信:

```java
byte[] outgoing = "Baeldung".getBytes();
byte[] secureOutgoing = saslClient.wrap(outgoing, 0, outgoing.length);

// Send secureOutgoing to the server over the network
```

类似地，服务器可以处理传入的通信:

```java
// Receive secureIncoming from the client over the network
byte[] incoming = saslServer.unwrap(secureIncoming, 0, netIn.length);

assertEquals("Baeldung", new String(incoming, StandardCharsets.UTF_8));
```

## 5.现实世界中的 SASL

所以，我们现在对什么是 SASL 以及如何在 Java 中使用它有了一个相当好的理解。但是，一般来说，这不是我们最终使用 SASL 的目的，至少在我们的日常生活中是这样。

正如我们之前看到的， **SASL 主要是针对 LDAP 和 SMTP** 这样的协议。尽管，越来越多的应用和 SASL 一起出现，比如卡夫卡。那么，我们如何使用 SASL 来认证这些服务呢？

假设我们已经为 SASL 配置了 Kafka Broker，并选择了 PLAIN 作为机制。简单地说，它使用明文形式的用户名和密码组合进行身份验证。

现在让我们来看看如何配置一个 Java 客户端来使用 SASL/PLAIN 来验证 Kafka 代理。

我们首先提供一个简单的 JAAS 配置“kafka_jaas.conf”:

```java
KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="username"
  password="password";
};
```

我们在启动 JVM 时使用了这个 JAAS 配置:

```java
-Djava.security.auth.login.config=kafka_jaas.conf
```

最后，我们必须添加一些属性来传递给我们的生产者和消费者实例:

```java
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
```

这就是全部了。不过，这只是 Kafka 客户端配置的一小部分。除了 PLAIN，Kafka 还支持 GSSAPI/Kerberos 进行身份验证。

## 6.比较 SASL

尽管 SASL 非常有效地提供了一种机制中立的方式来验证和保护客户端与服务器的通信。然而， **SASL 并不是这方面唯一可用的解决方案**。

Java 本身提供了其他机制来实现这个目标。我们将简要讨论他们，并了解他们如何对付 SASL:

*   [Java 安全套接字扩展](https://web.archive.org/web/20220523231806/https://docs.oracle.com/javase/9/security/java-secure-socket-extension-jsse-reference-guide.htm) (JSSE): **JSSE 是 Java 中实现 Java 安全套接字层(SSL)的一组包**。它提供数据加密、客户端和服务器身份验证以及消息完整性。与 SASL 不同，JSSE 依靠公钥基础设施(PKI)来工作。因此，SASL 比 JSSE 更加灵活和轻便。
*   [Java GSS API](https://web.archive.org/web/20220523231806/https://docs.oracle.com/javase/9/security/java-generic-security-services-java-gss-api1.htm)(JGSS):**JGGS 是通用安全服务应用编程接口(GSS-API)的 Java 语言绑定**。GSS-API 是一个 IETF 标准，用于应用程序访问安全服务。在 Java 中，在 GSS API 下，Kerberos 是唯一受支持的机制。Kerberos 同样需要一个 Kerberos 化的基础设施才能工作。与 SASL 相比，这里的选择是有限和重要的。

总的来说，SASL 是一个非常轻量级的框架，通过可插拔机制提供了各种各样的安全特性。根据需要，采用 SASL 的应用程序在实现正确的安全特性集方面有很多选择。

## 7.结论

总之，在本教程中，我们了解了 SASL 框架的基础，它提供了身份验证和安全通信。我们还讨论了 Java 中用于实现客户端和服务器端 SASL 的 API。

我们看到了如何通过 JCA 提供者使用安全机制。最后，我们还讨论了 SASL 在不同协议和应用程序中的用法。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523231806/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security)