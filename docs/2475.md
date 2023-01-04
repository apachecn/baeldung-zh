# 带有 Apache HttpClient 的自定义 HTTP 头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-custom-http-header>

## 1。概述

在本教程中，我们将了解如何使用 HttpClient 设置自定义头。

如果你想更深入地了解和学习其他很酷的东西，你可以使用 http client——直接去[http client 的主要教程](/web/20220628150454/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")。

## 延伸阅读:

## [用 Java 做一个简单的 HTTP 请求](/web/20220628150454/https://www.baeldung.com/java-http-request)

A quick and practical guide to performing basic HTTP requests using Java's built-in HttpUrlConnection.[Read more](/web/20220628150454/https://www.baeldung.com/java-http-request) →

## [高级 Apache HttpClient 配置](/web/20220628150454/https://www.baeldung.com/httpclient-advanced-config)

HttpClient configurations for advanced use cases.[Read more](/web/20220628150454/https://www.baeldung.com/httpclient-advanced-config) →

## [探索 Java 中的新 HTTP 客户端](/web/20220628150454/https://www.baeldung.com/java-9-http-client)

Explore the new Java HttpClient API which provides a lot of flexibility and powerful features.[Read more](/web/20220628150454/https://www.baeldung.com/java-9-http-client) →

## 2。根据要求设置标题–4.3 及以上

HttpClient 4.3 引入了一种新的构建请求的方式——即`RequestBuilder`。为了设置一个标题，**我们将使用`setHeader`方法——在构建器上:**

```
HttpClient client = HttpClients.custom().build();
HttpUriRequest request = RequestBuilder.get()
  .setUri(SAMPLE_URL)
  .setHeader(HttpHeaders.CONTENT_TYPE, "application/json")
  .build();
client.execute(request);
```

## 3。根据请求设置标题–在 4.3 之前

在 http client 4.3 之前的版本中，**我们可以通过对请求进行简单的`setHeader`调用来为请求设置任何自定义头:**

```
HttpClient client = new DefaultHttpClient();
HttpGet request = new HttpGet(SAMPLE_URL);
request.setHeader(HttpHeaders.CONTENT_TYPE, "application/json");
client.execute(request);
```

正如我们所看到的，我们将请求上的`Content-Type`直接设置为一个定制值——JSON。

## 4。在客户端设置默认标题

我们也可以**在客户端**上将其配置为默认报头，而不是在每个请求上设置报头:

```
Header header = new BasicHeader(HttpHeaders.CONTENT_TYPE, "application/json");
List<Header> headers = Lists.newArrayList(header);
HttpClient client = HttpClients.custom().setDefaultHeaders(headers).build();
HttpUriRequest request = RequestBuilder.get().setUri(SAMPLE_URL).build();
client.execute(request);
```

当所有请求都需要相同的标题时，这非常有用——例如自定义应用程序标题。

## 5。结论

本文演示了如何向通过 Apache HttpClient 发送的一个或所有请求添加 HTTP 头。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。