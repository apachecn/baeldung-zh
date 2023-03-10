# Apache HttpAsyncClient 教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpasyncclient-tutorial>

## 1。概述

在本教程中，我们将举例说明 Apache `HttpAsyncClient`最常见的用例——从基本用法，到如何**设置代理**，如何使用 **SSL 证书**，最后——如何**通过异步客户端认证**。

## 2。简单的例子

首先——让我们看看如何在一个简单的例子中使用`HttpAsyncClient`——发送一个 GET 请求:

```java
@Test
public void whenUseHttpAsyncClient_thenCorrect() throws Exception {
    CloseableHttpAsyncClient client = HttpAsyncClients.createDefault();
    client.start();
    HttpGet request = new HttpGet("http://www.google.com");

    Future<HttpResponse> future = client.execute(request, null);
    HttpResponse response = future.get();
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

注意**我们需要在使用异步客户端之前`start` 它**；否则，我们会得到以下异常:

```java
java.lang.IllegalStateException: Request cannot be executed; I/O reactor status: INACTIVE
    at o.a.h.u.Asserts.check(Asserts.java:46)
    at o.a.h.i.n.c.CloseableHttpAsyncClientBase.
      ensureRunning(CloseableHttpAsyncClientBase.java:90)
```

## 3。多线程与`HttpAsyncClient`

现在——让我们看看如何使用`HttpAsyncClient`同时执行多个请求。

在下面的例子中，我们使用`HttpAsyncClient`和`PoolingNHttpClientConnectionManager`向三个不同的主机发送三个 GET 请求:

```java
@Test
public void whenUseMultipleHttpAsyncClient_thenCorrect() throws Exception {
    ConnectingIOReactor ioReactor = new DefaultConnectingIOReactor();
    PoolingNHttpClientConnectionManager cm = 
      new PoolingNHttpClientConnectionManager(ioReactor);
    CloseableHttpAsyncClient client = 
      HttpAsyncClients.custom().setConnectionManager(cm).build();
    client.start();

    String[] toGet = { 
        "http://www.google.com/", 
        "http://www.apache.org/", 
        "http://www.bing.com/" 
    };

    GetThread[] threads = new GetThread[toGet.length];
    for (int i = 0; i < threads.length; i++) {
        HttpGet request = new HttpGet(toGet[i]);
        threads[i] = new GetThread(client, request);
    }

    for (GetThread thread : threads) {
        thread.start();
    }
    for (GetThread thread : threads) {
        thread.join();
    }
}
```

下面是我们处理响应的`GetThread`实现:

```java
static class GetThread extends Thread {
    private CloseableHttpAsyncClient client;
    private HttpContext context;
    private HttpGet request;

    public GetThread(CloseableHttpAsyncClient client,HttpGet req){
        this.client = client;
        context = HttpClientContext.create();
        this.request = req;
    }

    @Override
    public void run() {
        try {
            Future<HttpResponse> future = client.execute(request, context, null);
            HttpResponse response = future.get();
            assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
        } catch (Exception ex) {
            System.out.println(ex.getLocalizedMessage());
        }
    }
}
```

## 4。代理用`HttpAsyncClient`

接下来——让我们看看如何设置和使用带有`HttpAsyncClient`的**代理**。

在下面的例子中，我们通过代理发送一个 HTTP **GET 请求:**

```java
@Test
public void whenUseProxyWithHttpClient_thenCorrect() throws Exception {
    CloseableHttpAsyncClient client = HttpAsyncClients.createDefault();
    client.start();

    HttpHost proxy = new HttpHost("74.50.126.248", 3127);
    RequestConfig config = RequestConfig.custom().setProxy(proxy).build();
    HttpGet request = new HttpGet("https://issues.apache.org/");
    request.setConfig(config);

    Future<HttpResponse> future = client.execute(request, null);
    HttpResponse response = future.get();

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 5。带`HttpAsyncClient`的 SSL 证书

现在——让我们看看如何将一个 **SSL 证书**与`HttpAsyncClient`一起使用。

在下面的例子中，我们将`HttpAsyncClient`配置为**接受所有证书**:

```java
@Test
public void whenUseSSLWithHttpAsyncClient_thenCorrect() throws Exception {
    TrustStrategy acceptingTrustStrategy = new TrustStrategy() {
        public boolean isTrusted(X509Certificate[] certificate,  String authType) {
            return true;
        }
    };
    SSLContext sslContext = SSLContexts.custom()
      .loadTrustMaterial(null, acceptingTrustStrategy).build();

    CloseableHttpAsyncClient client = HttpAsyncClients.custom()
      .setSSLHostnameVerifier(SSLConnectionSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER)
      .setSSLContext(sslContext).build();
    client.start();

    HttpGet request = new HttpGet("https://mms.nw.ru/");
    Future<HttpResponse> future = client.execute(request, null);
    HttpResponse response = future.get();

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 6。带`HttpAsyncClient`的饼干

接下来——让我们看看如何通过`HttpAsyncClient`使用 cookies。

在下面的例子中，我们在发送请求之前设置了一个 cookie 值:

```java
@Test
public void whenUseCookiesWithHttpAsyncClient_thenCorrect() throws Exception {
    BasicCookieStore cookieStore = new BasicCookieStore();
    BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
    cookie.setDomain(".github.com");
    cookie.setPath("/");
    cookieStore.addCookie(cookie);

    CloseableHttpAsyncClient client = HttpAsyncClients.custom().build();
    client.start();

    HttpGet request = new HttpGet("http://www.github.com");
    HttpContext localContext = new BasicHttpContext();
    localContext.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);
    Future<HttpResponse> future = client.execute(request, localContext, null);
    HttpResponse response = future.get();

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 7 .**。使用 HttpAsyncClient 认证**

接下来，让我们看看如何使用`HttpAsyncClient`进行认证。

在以下示例中，我们使用`CredentialsProvider`通过基本身份验证访问主机:

```java
@Test
public void whenUseAuthenticationWithHttpAsyncClient_thenCorrect() throws Exception {
    CredentialsProvider provider = new BasicCredentialsProvider();
    UsernamePasswordCredentials creds = new UsernamePasswordCredentials("user", "pass");
    provider.setCredentials(AuthScope.ANY, creds);

    CloseableHttpAsyncClient client = 
      HttpAsyncClients.custom().setDefaultCredentialsProvider(provider).build();
    client.start();

    HttpGet request = new HttpGet("http://localhost:8080");
    Future<HttpResponse> future = client.execute(request, null);
    HttpResponse response = future.get();

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 8。结论

在本文中，我们展示了异步 Apache Http 客户端的各种用例。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220625232035/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "The code samples for the async http client on github") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。