# 用 Java HttpClient 定制 HTTP 头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-http-client-custom-header>

## 1.概观

Java 11 正式引入了 [Java HttpClient](/web/20221120204744/https://www.baeldung.com/java-9-http-client) 。在此之前，当我们需要使用 HTTP 客户端时，我们经常使用第三方库，如 [Apache HttpClient](/web/20221120204744/https://www.baeldung.com/httpclient-guide) 。

在这个简短的教程中，我们将看到如何用 Java HttpClient 添加定制的 HTTP 头。

## 2.自定义 HTTP 头

我们可以使用来自`HttpRequest.Builder`对象的三种方法之一轻松地添加定制头:`header`、`headers`或`setHeader`。让我们看看他们的行动。

### 2.1.使用`header()`方法

`header()` 方法允许我们一次添加一个标题。

我们可以根据需要多次添加相同的头名称，如下例所示，它们都将被发送:

```java
HttpClient httpClient = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
  .header("X-Our-Header-1", "value1")
  .header("X-Our-Header-1", "value2")
  .header("X-Our-Header-2", "value2")
  .uri(new URI(url)).build();

return httpClient.send(request, HttpResponse.BodyHandlers.ofString());
```

### 2.2.使用`headers()`方法

如果我们想同时添加多个头，我们可以使用`headers()`方法:

```java
HttpRequest request = HttpRequest.newBuilder()
  .headers("X-Our-Header-1", "value1", "X-Our-Header-2", "value2")
  .uri(new URI(url)).build();
```

该方法还允许我们向一个标题名添加多个值:

```java
HttpRequest request = HttpRequest.newBuilder()
  .headers("X-Our-Header-1", "value1", "X-Our-Header-1", "value2")
  .uri(new URI(url)).build();
```

### 2.3.使用`setHeader()`方法

最后，我们可以使用`setHeader()`方法添加一个头。但是，与`header()`方法不同的是，**如果我们不止一次地使用同一个头名称，它将覆盖我们之前用那个名称**设置的任何头:

```java
HttpRequest request = HttpRequest.newBuilder()
  .setHeader("X-Our-Header-1", "value1")
  .setHeader("X-Our-Header-1", "value2")
  .uri(new URI(url)).build();
```

在上面的例子中，我们的头的值是“value2”。

## 3.结论

总之，我们已经学习了用 Java HttpClient 添加定制 HTTP 头的不同方法。

和往常一样，GitHub 上的[提供了本文的示例代码。](https://web.archive.org/web/20221120204744/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-httpclient)