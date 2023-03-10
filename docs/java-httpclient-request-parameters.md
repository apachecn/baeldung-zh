# 向 Java HttpClient 请求添加参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpclient-request-parameters>

## 1.介绍

在本教程中，我们将讨论向 Java `HttpClient`请求添加参数。

Java `HTTPClient` 是 Java 11 的内置功能。因此，我们可以不用像 [Apache HttpClient](/web/20221117045616/https://www.baeldung.com/httpclient-guide) 和 [OkHttp](/web/20221117045616/https://www.baeldung.com/guide-to-okhttp) 这样的第三方库来发送 HTTP 请求。

## 2.添加参数

`HttpRequest.Builder`帮助我们使用构建器模式轻松创建 HTTP 请求和添加参数。

**Java `HttpClient` API 没有提供任何添加查询参数**的方法。虽然我们可以利用第三方库，比如来自 Apache HttpClient 的 [`URIBuilder`](https://web.archive.org/web/20221117045616/https://www.drafts.baeldung.com/apache-httpclient-parameters#add-parameters-to-httpclient-requests-using-uribuilder) 来构建请求 URI 字符串。让我们看看只使用 Java 11 中添加的功能会是什么样子:

```java
HttpRequest request = HttpRequest.newBuilder()
  .version(HttpClient.Version.HTTP_2)
  .uri(URI.create("https://postman-echo.com/get?param1=value1&param2;=value2"))
  .GET()
  .build();
```

注意，我们已经将`version()`方法设置为使用 HTTP 版本 2。Java `HTTPClient`默认使用 HTTP 2。但是，如果服务器不支持 HTTP 2 的请求，该版本将自动降级为 HTTP 1.1。

此外，我们使用了默认的 HTTP 请求方法`GET()`。如果我们不指定 HTTP 请求方法，将使用默认的 GET 方法。

最后，我们还可以用默认配置以简洁的形式编写相同的请求:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create("https://postman-echo.com/get?param1=value1&param2;=value2"))
  .build();
```

## 3.结论

在这个例子中，我们介绍了如何向 Java `HTTPClient`请求添加参数。此外，所有这些例子和代码片段的实现都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221117045616/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-3)

在示例中，我们使用了由`https://postman-echo.com.`提供的样本 REST 端点