# 信任 OkHttp 中的自签名证书

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/okhttp-self-signed-cert>

## 1.概观

在本文中，我们将看到如何初始化和**配置一个`OkHttpClient`来信任自签名证书**。为此，我们将设置一个最小的支持 HTTPS 的 Spring Boot 应用程序，由自签名证书保护。

参考我们在 OkHttp 上的[文章集，了解更多关于这个库的细节。](/web/20221028002251/https://www.baeldung.com/tag/okhttp/)

## 2.基本面

在我们深入研究负责实现这一点的代码之前，让我们先来看看底线。**SSL 的[本质是它在任意两方](/web/20221028002251/https://www.baeldung.com/java-ssl)**之间建立安全连接，通常是客户端和服务器。此外，它还有助于**保护通过网络传输的数据的隐私和完整性**。

[JSSE API](https://web.archive.org/web/20221028002251/https://docs.oracle.com/en/java/javase/11/security/java-secure-socket-extension-jsse-reference-guide.html)，Java SE 的安全组件，为 SSL/TLS 协议提供完整的 API 支持。

SSL 证书，也称为数字证书，在建立 TLS 握手、促进通信方之间的加密和信任方面起着至关重要的作用。自签名 SSL 证书不是由众所周知的可信证书颁发机构(CA)颁发的。开发者可以很容易地[生成并](/web/20221028002251/https://www.baeldung.com/openssl-self-signed-cert)签署它们，从而为他们的软件启用 HTTPS。

由于自签名证书不可信，无论是浏览器还是标准的 HTTPS 客户端如 [OkHttp](/web/20221028002251/https://www.baeldung.com/guide-to-okhttp) 和 [Apache HTTP 客户端](/web/20221028002251/https://www.baeldung.com/httpclient-ssl)都默认不信任它们。

最后，我们可以使用 web 浏览器或 OpenSSL 命令行实用程序方便地获取[服务器证书。](/web/20221028002251/https://www.baeldung.com/linux/ssl-certificates)

## 3.设置测试环境

为了演示一个应用程序使用 OkHttp 接受并信任自签名证书，让我们快速地[配置并启动一个简单的 Spring Boot 应用程序](/web/20221028002251/https://www.baeldung.com/spring-boot-https-self-signed-certificate)，启用 HTTPS(由自签名证书保护)。

默认配置将启动一个监听端口 8443 的 Tomcat 服务器，并公开一个可在`“https://localhost:8443/welcome”`访问的安全 REST API。

现在，让我们使用 OkHttp 客户端向该服务器发出一个 HTTPS 请求，并使用`“/welcome”` API。

## 4.`OkHttpClient`和 SSL

这个部分将初始化一个`OkHttpClient`并使用它连接到我们刚刚设置的测试环境。此外，我们将检查路径中遇到的错误，并逐步达到我们的最终目标，即使用 OkHttp 信任自签名证书。

首先，让我们为`OkHttpClient`创建一个构建器:

```java
OkHttpClient.Builder builder = new OkHttpClient.Builder();
```

此外，让我们声明我们将在整个教程中使用的 HTTPS URL:

```java
int SSL_APPLICATION_PORT = 8443;
String HTTPS_WELCOME_URL = "https://localhost:" + SSL_APPLICATION_PORT + "/welcome";
```

### 4.1.`SSLHandshakeException`

在没有为 SSL 配置`OkHttpClient`的情况下，如果我们试图使用 HTTPS URL，我们会得到一个安全异常:

```java
@Test(expected = SSLHandshakeException.class)
public void whenHTTPSSelfSignedCertGET_thenException() {
    builder.build()
    .newCall(new Request.Builder()
    .url(HTTPS_WELCOME_URL).build())
    .execute();
}
```

堆栈跟踪是:

```java
javax.net.ssl.SSLHandshakeException: PKIX path building failed: 
    sun.security.provider.certpath.SunCertPathBuilderException:
    unable to find valid certification path to requested target
    ...
```

上述错误恰恰意味着服务器使用了未经证书颁发机构(CA)签名的自签名证书。

因此，客户端无法验证[信任链](https://web.archive.org/web/20221028002251/https://docs.oracle.com/cd/E19146-01/821-1828/ginal/index.html)直到根证书，所以它抛出了一个 [`SSLHandshakeException`](/web/20221028002251/https://www.baeldung.com/java-ssl-handshake-failures) 。

### 4.2.`SSLPeerUnverifiedException`

现在，让我们配置`OkHttpClient`来信任一个证书，不管它的性质是 CA 签名还是自签名。

首先，我们需要创建我们自己的 [`TrustManager`](https://web.archive.org/web/20221028002251/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/X509TrustManager.html) ,它会使默认的证书验证无效，并用我们的自定义实现覆盖那些验证:

```java
TrustManager TRUST_ALL_CERTS = new X509TrustManager() {
    @Override
    public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) {
    }

    @Override 
    public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) {
    }

    @Override
    public java.security.cert.X509Certificate[] getAcceptedIssuers() {
        return new java.security.cert.X509Certificate[] {};
    }
};
```

接下来，我们将使用上面的`TrustManager`来初始化一个`[SSLContext](https://web.archive.org/web/20221028002251/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/SSLContext.html)`，同时设置`OkHttpClient`生成器的 [`SSLSocketFactory`](https://web.archive.org/web/20221028002251/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/SSLSocketFactory.html) :

```java
SSLContext sslContext = SSLContext.getInstance("SSL");
sslContext.init(null, new TrustManager[] { TRUST_ALL_CERTS }, new java.security.SecureRandom());
builder.sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager) TRUST_ALL_CERTS);
```

同样，让我们运行测试。不难相信，即使在进行了上述调整之后，使用 HTTPS 网址也会抛出一个错误:

```java
@Test(expected = SSLPeerUnverifiedException.class)
public void givenTrustAllCerts_whenHTTPSSelfSignedCertGET_thenException() {
    // initializing the SSLContext and set the sslSocketFactory
    builder.build()
        .newCall(new Request.Builder()
        .url(HTTPS_WELCOME_URL).build())
        .execute();
}
```

确切的误差是:

```java
javax.net.ssl.SSLPeerUnverifiedException: Hostname localhost not verified:
    certificate: sha256/bzdWeeiDwIVjErFX98l+ogWy9OFfBJsTRWZLB/bBxbw=
    DN: CN=localhost, OU=localhost, O=localhost, L=localhost, ST=localhost, C=IN
    subjectAltNames: []
```

这是由于一个众所周知的问题，即[主机名验证失败](https://web.archive.org/web/20221028002251/https://tersesystems.com/blog/2014/03/23/fixing-hostname-verification/)。大多数 **HTTP 库根据证书的 SubjectAlternativeName 的 DNS 名称字段**执行主机名验证，该字段在服务器的自签名证书中不可用，如上面详细的堆栈跟踪所示。

### 4.3.超越`HostnameVerifier`

正确配置`OkHttpClient`的最后一步是禁用默认的`HostnameVerifier`,用另一个绕过主机名验证的来替换它。

让我们进行最后一项定制:

```java
builder.hostnameVerifier(new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
}); 
```

现在，让我们最后一次运行测试:

```java
@Test
public void givenTrustAllCertsSkipHostnameVerification_whenHTTPSSelfSignedCertGET_then200OK() {
    // initializing the SSLContext and set the sslSocketFactory
    // set the custom hostnameVerifier
    Response response = builder.build()
        .newCall(new Request.Builder()
        .url(HTTPS_WELCOME_URL).build())
        .execute();
    assertEquals(200, response.code());
    assertNotNull(response.body());
    assertEquals("<h1>Welcome to Secured Site</h1>", response.body()
        .string());
}
```

**最后，`OkHttpClient`成功地消费了由自签名证书**保护的 HTTPS URL。

## 5.结论

在本教程中，我们学习了如何为一个`OkHttpClient`配置 SSL，以便它能够信任一个自签名证书并使用任何 HTTPS URL。

然而，需要考虑的重要一点是，尽管这种设计完全省略了证书验证和主机名验证，但是客户端和服务器之间的所有通信仍然是加密的。双方之间的信任丢失了，但是 SSL 握手和加密没有被破坏。

和往常一样，我们可以在 GitHub 上找到完整的源代码[。](https://web.archive.org/web/20221028002251/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)