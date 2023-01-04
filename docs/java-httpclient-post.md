# 用 Java HttpClient 发布

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpclient-post>

## 1.概观

Java 11 中引入了 [Java `HttpClient`](https://web.archive.org/web/20221117045616/https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html) API。API **实现了最新 HTTP 标准**的客户端。它支持 HTTP/1.1 和 HTTP/2，包括同步和异步编程模型。

我们可以用它来发送 HTTP 请求并检索它们的响应。在 Java 11 之前，我们不得不依赖基本的`[URLConnection](/web/20221117045616/https://www.baeldung.com/java-http-request)` 实现或第三方库，如 [Apache `HttpClient`](/web/20221117045616/https://www.baeldung.com/httpclient-guide) 。

在本教程中，我们将看看使用 Java `HttpClient`发送 POST 请求。我们将展示如何发送同步和异步 POST 请求，以及并发 POST 请求。此外，我们将检查如何向 POST 请求添加认证参数和 JSON 主体。

最后，我们将了解如何上传文件和提交表单数据。因此，我们将涵盖大多数常见的用例。

## 2.准备发布请求

在发送 HTTP 请求之前，我们首先需要创建一个`HttpClient`的实例。

使用`newBuilder`方法，可以从其构建器中配置和创建 **`HttpClient` 实例。否则，如果不需要配置，我们可以利用`newHttpClient`实用程序方法创建一个默认客户端:**

```java
HttpClient client = HttpClient.newHttpClient();
```

`HttpClient`默认情况下会使用 HTTP/2。如果服务器不支持 HTTP/2，它也会自动降级到 HTTP/1.1。

现在我们准备从它的构建器中创建一个`HttpRequest`的实例。稍后我们将利用客户机实例来发送这个请求。POST 请求的最小参数是服务器 URL、请求方法和主体:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create(serviceUrl))
  .POST(HttpRequest.BodyPublishers.noBody())
  .build();
```

请求体需要通过`BodyPublisher`类提供。它是一个反应式流发布器，按需发布请求体流。在我们的例子中，我们使用了一个不发送请求正文的正文发布者。

## 3.发送发布请求

现在我们已经准备好了 POST 请求，让我们看看发送它的不同选项。

### 3.1.同步地

我们可以使用这个默认的`send`方法发送准备好的请求。这个方法将**阻塞我们的代码，直到收到响应**:

```java
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString())
```

`BodyHandlers`实用程序实现了各种有用的处理程序，比如将响应体作为`String`来处理，或者将响应体流式传输到一个文件。一旦收到响应，`HttpResponse`对象将包含响应状态、标题和主体:

```java
assertThat(response.statusCode())
  .isEqualTo(200);
assertThat(response.body())
  .isEqualTo("{\"message\":\"ok\"}");
```

### 3.2.异步

我们可以使用`sendAsync`方法异步发送前一个例子中的相同请求。这个方法没有阻塞我们的代码，而是将**立即返回一个** [`**CompletableFuture**`](/web/20221117045616/https://www.baeldung.com/java-completablefuture) **实例**:

```java
CompletableFuture<HttpResponse<String>> futureResponse = client.sendAsync(request, HttpResponse.BodyHandlers.ofString());
```

一旦`HttpResponse `可用，则`CompletableFuture`完成:

```java
HttpResponse<String> response = futureResponse.get();
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.body()).isEqualTo("{\"message\":\"ok\"}");
```

### 3.3.同时地

我们可以将[流](/web/20221117045616/https://www.baeldung.com/java-8-streams)与`CompletableFutures`组合起来，以便**同时发出几个请求，并等待它们的响应**:

```java
List<CompletableFuture<HttpResponse<String>>> completableFutures = serviceUrls.stream()
  .map(URI::create)
  .map(HttpRequest::newBuilder)
  .map(builder -> builder.POST(HttpRequest.BodyPublishers.noBody()))
  .map(HttpRequest.Builder::build)
  .map(request -> client.sendAsync(request, HttpResponse.BodyHandlers.ofString()))
  .collect(Collectors.toList());
```

现在，让我们等待所有请求完成，以便我们可以一次处理它们的响应:

```java
CompletableFuture<List<HttpResponse<String>>> combinedFutures = CompletableFuture
  .allOf(completableFutures.toArray(new CompletableFuture[0]))
  .thenApply(future ->
    completableFutures.stream()
      .map(CompletableFuture::join)
      .collect(Collectors.toList()));
```

当我们使用`allOf`和`join`方法组合所有响应时，我们得到一个新的`CompletableFuture` 来保存我们的响应:

```java
List<HttpResponse<String>> responses = combinedFutures.get();
responses.forEach((response) -> {
  assertThat(response.statusCode()).isEqualTo(200);
  assertThat(response.body()).isEqualTo("{\"message\":\"ok\"}");
});
```

## 4.添加身份验证参数

我们可以在客户端级别设置一个**验证器，用于所有请求的 HTTP 验证**:

```java
HttpClient client = HttpClient.newBuilder()
  .authenticator(new Authenticator() {
    @Override
    protected PasswordAuthentication getPasswordAuthentication() {
      return new PasswordAuthentication(
        "baeldung",
        "123456".toCharArray());
      }
  })
  .build();
```

然而，`HttpClient`不会发送基本凭证，直到服务器使用`WWW-Authenticate`头对它们进行质询。

要绕过这一点，我们总是可以手动创建和发送基本授权头:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create(serviceUrl))
  .POST(HttpRequest.BodyPublishers.noBody())
  .header("Authorization", "Basic " + 
    Base64.getEncoder().encodeToString(("baeldung:123456").getBytes()))
  .build();
```

## 5.添加几何体

在到目前为止的例子中，我们还没有向我们的 POST 请求添加任何主体。然而，POST 方法通常用于**通过请求体**向服务器发送数据。

### 5.1.JSON 体

`BodyPublishers` 实用程序实现了各种有用的发布器，比如从`String`或文件发布请求体。我们可以**将 JSON 数据发布为`String`** ，使用 UTF-8 字符集进行转换:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create(serviceUrl))
  .POST(HttpRequest.BodyPublishers.ofString("{\"action\":\"hello\"}"))
  .build();
```

### 5.2.上传文件

让我们创建一个[临时文件](/web/20221117045616/https://www.baeldung.com/junit-5-temporary-directory)，我们可以使用它通过`HttpClient`上传:

```java
Path file = tempDir.resolve("temp.txt");
List<String> lines = Arrays.asList("1", "2", "3");
Files.write(file, lines);
```

**`HttpClient`提供了一个单独的方法`BodyPublishers.ofFile,` ，用于将文件添加到文章正文**中。我们可以简单地添加我们的临时文件作为方法参数，API 会处理剩下的事情:

```java
HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create(serviceUrl))
  .POST(HttpRequest.BodyPublishers.ofFile(file))
  .build();
```

### 5.3.提交表单

与文件相反，`HttpClient`没有提供一个单独的方法来发布表单数据。因此，我们将再次需要**使用`BodyPublishers.ofString` 方法**:

```java
Map<String, String> formData = new HashMap<>();
formData.put("username", "baeldung");
formData.put("message", "hello");

HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create(serviceUrl))
  .POST(HttpRequest.BodyPublishers.ofString(getFormDataAsString(formData)))
  .build();
```

然而，我们需要使用一个定制的实现将表单数据从一个`Map`转换成一个`String`:

```java
private static String getFormDataAsString(Map<String, String> formData) {
    StringBuilder formBodyBuilder = new StringBuilder();
    for (Map.Entry<String, String> singleEntry : formData.entrySet()) {
        if (formBodyBuilder.length() > 0) {
            formBodyBuilder.append("&");
        }
        formBodyBuilder.append(URLEncoder.encode(singleEntry.getKey(), StandardCharsets.UTF_8));
        formBodyBuilder.append("=");
        formBodyBuilder.append(URLEncoder.encode(singleEntry.getValue(), StandardCharsets.UTF_8));
    }
    return formBodyBuilder.toString();
}
```

## 6.结论

在本文中，**我们** **探索了使用 Java 11** 中引入的 Java HttpClient API 发送 POST 请求。

我们学习了如何创建一个`HttpClient`实例并准备一个 POST 请求。我们看到了如何同步、异步和并发地发送准备好的请求。接下来，我们还了解了如何添加基本的身份验证参数。

最后，我们看了向 POST 请求添加正文。我们讨论了 JSON 有效负载、上传文件和提交表单数据。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221117045616/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-httpclient)