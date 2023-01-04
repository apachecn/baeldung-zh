# 带 SSL 的 Java HttpClient

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpclient-ssl>

## 1.概观

在本教程中，我们将探索如何使用 Java HttpClient 连接到 HTTPS URL。我们还将学习如何使用没有有效 SSL 证书的 URL 客户端。

在旧版本的 Java 中，我们更喜欢使用 Apache HTTPClient 和 OkHttp 这样的库来连接服务器。在 Java 11 中，JDK 中增加了一个[改进的 HttpClient 库](/web/20221117045616/https://www.baeldung.com/java-9-http-client)。让我们探索如何使用它通过 SSL 调用服务。

## 2.使用 Java HttpClient 调用 HTTPS URL

我们将使用测试用例来运行客户端代码。出于测试目的，我们将使用在 HTTPS 上运行的现有 URL。

让我们编写代码来设置客户端并调用服务:

```java
HttpClient httpClient = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create("https://www.google.com/"))
  .build();

HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString()); 
```

这里，我们首先使用`HttpClient.newHttpClient() `方法创建了一个客户端。然后，我们创建了一个请求，并设置了我们将要访问的服务的 URL。最后，我们使用`HttpClient.send()`方法发送请求并收集响应——一个包含响应体的`HttpResponse `对象作为`String.`

当我们将上面的代码放入一个测试用例中并执行下面的断言时，我们会观察到它通过了:

```java
assertEquals(200, response.statusCode());
```

## 3.调用无效的 HTTPS URL

现在，让我们将 URL 更改为另一个没有有效 SSL 证书的 URL。我们可以通过更改请求对象来做到这一点:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(new URI("https://www.testingmcafeesites.com/"))
  .build();
```

当我们再次运行测试时，我们得到以下错误:

```java
Caused by: java.security.cert.CertificateException: No subject alternative DNS name matching www.testingmcafeesites.com found.
  at java.base/sun.security.util.HostnameChecker.matchDNS(HostnameChecker.java:212)
  at java.base/sun.security.util.HostnameChecker.match(HostnameChecker.java:103) 
```

这是因为该 URL 没有有效的 SSL 证书。

## 4.绕过 SSL 证书验证

为了解决上面的错误，让我们来看一个绕过 SSL 证书验证的解决方案。

在 Apache HttpClient 中，我们可以将客户端修改为[绕过证书验证](/web/20221117045616/https://www.baeldung.com/httpclient-ssl)。然而，我们不能用 Java HttpClient 做到这一点。**我们将不得不依靠修改 JVM 来禁用主机名验证。**

一种方法是[将网站的证书导入 Java 密钥库](/web/20221117045616/https://www.baeldung.com/java-import-cer-certificate-into-keystore)。这是一种常见的做法，如果只有少量可信的内部网站，这是一个很好的选择。

然而，如果有大量的网站或太多的环境需要管理，这可能会变得令人厌倦。在这种情况下，我们可以**使用属性`jdk.internal.httpclient.disableHostnameVerification`来禁用主机名验证。**

我们可以在运行应用程序时**将该属性设置为命令行参数**:

```java
java -Djdk.internal.httpclient.disableHostnameVerification=true -jar target/java-httpclient-ssl-0.0.1-SNAPSHOT.jar 
```

**或者，我们可以在创建客户端**之前通过 **以编程方式设置该属性:**

```java
Properties props = System.getProperties();
props.setProperty("jdk.internal.httpclient.disableHostnameVerification", Boolean.TRUE.toString());

HttpClient httpClient = HttpClient.newHttpClient(); 
```

当我们现在运行测试时，我们会看到它通过。

**我们应该注意，更改属性将意味着对所有请求禁用证书验证。** **这可能不可取，尤其是在生产方面。**然而，在非生产环境中引入该属性是很常见的。

## 5.我们能用 Spring 使用 Java HttpClient 吗？

Spring 提供了两个流行的接口来发出 HTTP 请求:

*   `RestTemplate`对于同步请求
*   `WebClient`对于同步和异步请求

两者都可以和流行的 HTTP 客户端一起使用，比如 Apache HttpClient、OkHttp 和旧的`HttpURLConnection`。但是，**我们不能把 Java HttpClient 插到这两个接口**。**这更像是他们的替代品。**

我们可以使用 Java HttpClient 进行同步和异步请求，转换请求和响应，添加超时等。因此，可以直接使用它，而不需要 Spring 的接口。

## 6.结论

在本文中，我们探讨了如何使用 Java HTTP 客户端连接到需要 SSL 的服务器。我们还研究了对没有有效证书的 URL 使用客户端的方法。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221117045616/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)