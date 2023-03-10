# 带 SSL 的 Apache HttpClient

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-ssl>

## 1。概述

本文将展示如何用“接受所有”SSL 支持来配置 Apache HttpClient 4。目标很简单——消费没有有效证书的 HTTPS 网址。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接进入 HttpClient 的主要指南。

## 延伸阅读:

## [Apache HttpClient 连接管理](/web/20221126231452/https://www.baeldung.com/httpclient-connection-management)

How to open, manage and close connections with the Apache HttpClient 4.[Read more](/web/20221126231452/https://www.baeldung.com/httpclient-connection-management) →

## [高级 Apache HttpClient 配置](/web/20221126231452/https://www.baeldung.com/httpclient-advanced-config)

HttpClient configurations for advanced use cases.[Read more](/web/20221126231452/https://www.baeldung.com/httpclient-advanced-config) →

## [Apache http client–发送定制 Cookie](/web/20221126231452/https://www.baeldung.com/httpclient-cookies)

How to send Custom Cookies with the Apache HttpClient.[Read more](/web/20221126231452/https://www.baeldung.com/httpclient-cookies) →

## 2。`SSLPeerUnverifiedException`

如果没有使用`HttpClient`配置 SSL，下面的测试——使用 HTTPS URL——将会失败:

```java
public class RestClientLiveManualTest {

    @Test(expected = SSLPeerUnverifiedException.class)
    public void whenHttpsUrlIsConsumed_thenException() 
      throws ClientProtocolException, IOException {

        CloseableHttpClient httpClient = HttpClients.createDefault();
        String urlOverHttps
          = "https://localhost:8082/httpclient-simple";
        HttpGet getMethod = new HttpGet(urlOverHttps);

        HttpResponse response = httpClient.execute(getMethod);
        assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    }
}
```

确切的故障是:

```java
javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
    at sun.security.ssl.SSLSessionImpl.getPeerCertificates(SSLSessionImpl.java:397)
    at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:126)
    ...
```

每当无法为 URL 建立有效的信任链时，就会出现 [`javax.net.ssl.SSLPeerUnverifiedException`异常](https://web.archive.org/web/20221126231452/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/javax/net/ssl/SSLPeerUnverifiedException.html "SSLPeerUnverifiedException javadoc in Java SE 7")。

## 3。配置 SSL–全部接受(HttpClient < 4.3)

现在让我们配置 HTTP 客户端信任所有证书链，而不管它们的有效性如何:

```java
@Test
public final void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenOk() 
  throws GeneralSecurityException {
    HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory();
    CloseableHttpClient httpClient = (CloseableHttpClient) requestFactory.getHttpClient();

    TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
    SSLSocketFactory sf = new SSLSocketFactory(acceptingTrustStrategy, ALLOW_ALL_HOSTNAME_VERIFIER);
    httpClient.getConnectionManager().getSchemeRegistry().register(new Scheme("https", 8443, sf));

    ResponseEntity<String> response = new RestTemplate(requestFactory).
      exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

随着新的`TrustStrategy`现在**覆盖标准证书验证过程**(应该咨询已配置的信任管理器)——测试现在通过了，客户端能够使用 HTTPS URL 。

## 4。 **配置 SSL–全部接受(HttpClient 4.4 及以上)**

有了新的 HTTPClient，我们现在有了一个增强的、重新设计的默认 SSL 主机名验证器。同样有了`SSLConnectionSocketFactory`和`RegistryBuilder`的引入，构建 SSLSocketFactory 就很容易了。所以我们可以这样写上面的测试用例:

```java
@Test
public final void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenOk()
  throws GeneralSecurityException {
    TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
    SSLContext sslContext = SSLContexts.custom().loadTrustMaterial(null, acceptingTrustStrategy).build();
    SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, 
      NoopHostnameVerifier.INSTANCE);

    Registry<ConnectionSocketFactory> socketFactoryRegistry = 
      RegistryBuilder.<ConnectionSocketFactory> create()
      .register("https", sslsf)
      .register("http", new PlainConnectionSocketFactory())
      .build();

    BasicHttpClientConnectionManager connectionManager = 
      new BasicHttpClientConnectionManager(socketFactoryRegistry);
    CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf)
      .setConnectionManager(connectionManager).build();

    HttpComponentsClientHttpRequestFactory requestFactory = 
      new HttpComponentsClientHttpRequestFactory(httpClient);
    ResponseEntity<String> response = new RestTemplate(requestFactory)
      .exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

## 5。带 SSL 的 Spring`RestTemplate`(http client<4.3)

现在我们已经看到了如何配置一个支持 SSL 的 raw `HttpClient`，让我们来看看一个更高级的客户端——Spring`RestTemplate`。

在没有配置 SSL 的情况下，以下测试如预期的那样失败:

```java
@Test(expected = ResourceAccessException.class)
public void whenHttpsUrlIsConsumed_thenException() {
    String urlOverHttps 
      = "https://localhost:8443/httpclient-simple/api/bars/1";
    ResponseEntity<String> response 
      = new RestTemplate().exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

让我们来配置 SSL:

```java
@Test
public void givenAcceptingAllCertificates_whenHttpsUrlIsConsumed_thenException() 
  throws GeneralSecurityException {
    HttpComponentsClientHttpRequestFactory requestFactory 
      = new HttpComponentsClientHttpRequestFactory();
    DefaultHttpClient httpClient
      = (DefaultHttpClient) requestFactory.getHttpClient();
    TrustStrategy acceptingTrustStrategy = (cert, authType) -> true
    SSLSocketFactory sf = new SSLSocketFactory(
      acceptingTrustStrategy, ALLOW_ALL_HOSTNAME_VERIFIER);
    httpClient.getConnectionManager().getSchemeRegistry()
      .register(new Scheme("https", 8443, sf));

    String urlOverHttps
      = "https://localhost:8443/httpclient-simple/api/bars/1";
    ResponseEntity<String> response = new RestTemplate(requestFactory).
      exchange(urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

如您所见，这**与我们为原始 HttpClient** 配置 SSL 的方式非常相似——我们为请求工厂配置 SSL 支持，然后通过这个预配置的工厂实例化模板。

## ⑥。春天`RestTemplate`用 SSL (HttpClient 4.4)

我们可以用同样的方式来配置我们的`RestTemplate`:

```java
@Test
public void givenAcceptingAllCertificatesUsing4_4_whenUsingRestTemplate_thenCorrect() 
throws ClientProtocolException, IOException {
    CloseableHttpClient httpClient
      = HttpClients.custom()
        .setSSLHostnameVerifier(new NoopHostnameVerifier())
        .build();
    HttpComponentsClientHttpRequestFactory requestFactory 
      = new HttpComponentsClientHttpRequestFactory();
    requestFactory.setHttpClient(httpClient);

    ResponseEntity<String> response 
      = new RestTemplate(requestFactory).exchange(
      urlOverHttps, HttpMethod.GET, null, String.class);
    assertThat(response.getStatusCode().value(), equalTo(200));
}
```

## 7。结论

本教程讨论了如何为 Apache HttpClient 配置 SSL，以便它能够使用任何 HTTPS URL，而不管证书如何。也显示了弹簧`RestTemplate`的相同配置。

然而，需要理解的一件重要事情是**这种策略完全忽略了证书检查**——这使得它不安全，并且只在有意义的地方使用。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。