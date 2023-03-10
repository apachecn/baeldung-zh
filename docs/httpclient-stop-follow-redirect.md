# Apache http client–不要跟随重定向

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-stop-follow-redirect>

## 1。概述

在本文中，我将展示如何**配置 Apache HttpClient 来停止跟踪重定向**。

默认情况下，遵循 HTTP 规范，**HTTP client 将自动遵循重定向**。

对于某些用例来说，这可能非常好，但肯定有不希望这样的用例——我们现在将看看如何改变默认行为和**停止跟随重定向**。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220810164043/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 2。不要跟随重定向

### 2.1。HttpClient 4.3 之前

在 Http 客户端的旧版本(4.3 之前)中，我们可以如下配置客户端对重定向的操作:

```java
@Test
public void givenRedirectsAreDisabled_whenConsumingUrlWhichRedirects_thenNotRedirected() 
  throws ClientProtocolException, IOException {
    DefaultHttpClient instance = new DefaultHttpClient();

    HttpParams params = new BasicHttpParams();
    params.setParameter(ClientPNames.HANDLE_REDIRECTS, false);
    // HttpClientParams.setRedirecting(params, false); // alternative

    HttpGet httpGet = new HttpGet("http://t.co/I5YYd9tddw");
    httpGet.setParams(params);
    CloseableHttpResponse response = instance.execute(httpGet);

    assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
}
```

请注意可用于配置重定向行为**而无需设置实际原始`http.protocol.handle-redirects`参数**的替代 API:

```java
HttpClientParams.setRedirecting(params, false);
```

还要注意的是，禁用 follow redirects 后，我们现在可以检查 Http 响应状态代码是否确实是`301 Moved Permanently`——就像它应该的那样。

### 2.2。HttpClient 4.3 之后

**HttpClient 4.3 引入了更干净、更高级别的 API** 来构建和配置客户端:

```java
@Test
public void givenRedirectsAreDisabled_whenConsumingUrlWhichRedirects_thenNotRedirected() 
  throws ClientProtocolException, IOException {
    HttpClient instance = HttpClientBuilder.create().disableRedirectHandling().build();
    HttpResponse response = instance.execute(new HttpGet("http://t.co/I5YYd9tddw"));

    assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
}
```

注意，新的 API 用这种重定向行为配置了整个客户端——而不仅仅是单个请求。

## 3。结论

这篇快速教程讲述了如何配置 Apache HTTP client——pre`4.3`和 post——以防止它自动跟随 HTTP 重定向。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220810164043/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project exemplifying Live HttpClient 4.3 examples") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。