# Apache HttpClient 中的自定义用户代理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-user-agent-header>

## 1。概述

这个快速教程将向**展示如何使用 Apache HttpClient** 发送一个定制的`User-Agent`头。

## 2。在`HttpClient` 上设置`User-Agent`

### 2.1。HttpClient 4.3 之前

当使用较旧版本的 Http 客户端(4.3 之前)时，设置`User-Agent` 的值是通过低级 API 完成的**:**

```java
client.getParams().setParameter(CoreProtocolPNames.USER_AGENT, "Mozilla/5.0 Firefox/26.0");
```

同样的事情也可以通过**一个更高级别的 API 以及**来完成——无需处理原始的`http.useragent`属性:

```java
HttpProtocolParams.setUserAgent(client.getParams(), "Mozilla/5.0 Firefox/26.0");
```

完整的示例如下所示:

```java
@Test
public void whenClientUsesCustomUserAgent_thenCorrect() 
  throws ClientProtocolException, IOException {
    DefaultHttpClient client = new DefaultHttpClient();
    HttpProtocolParams.setUserAgent(client.getParams(), "Mozilla/5.0 Firefox/26.0");

    HttpGet request = new HttpGet("http://www.github.com");
    client.execute(request);
}
```

### 2.2。HttpClient 4.3 之后

在 Apache 客户端的最新版本(4.3 版之后)中，通过新的 fluent APIs 以更简洁的方式实现了同样的功能:

```java
@Test
public void whenRequestHasCustomUserAgent_thenCorrect() 
  throws ClientProtocolException, IOException {
    HttpClient instance = HttpClients.custom().setUserAgent("Mozilla/5.0 Firefox/26.0").build();
    instance.execute(new HttpGet("http://www.github.com"));
}
```

## 3。根据个人请求设置`User-Agent`

定制的`User-Agent` 报头也可以针对单个请求设置，而不是针对整个`HttpClient`:

```java
@Test
public void givenDeprecatedApi_whenRequestHasCustomUserAgent_thenCorrect() 
  throws ClientProtocolException, IOException {
    HttpClient instance = HttpClients.custom().build();
    HttpGet request = new HttpGet(SAMPLE_URL);
    request.setHeader(HttpHeaders.USER_AGENT, "Mozilla/5.0 Firefox/26.0");
    instance.execute(request);
}
```

## 4。结论

本文展示了如何使用 HttpClient 发送带有自定义头的请求——例如模拟特定浏览器的行为。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20221117160710/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project exemplifying Live HttpClient 4.3 examples") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。