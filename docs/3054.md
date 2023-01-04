# Unirest 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/unirest>

## 1。概述

Unirest 是 Mashape 的一个轻量级 HTTP 客户端库。除了 Java，Node.js 也可以使用它。Net，Python，Ruby 等。

在我们开始之前，请注意我们将使用 [mocky.io](https://web.archive.org/web/20220626205115/https://www.mocky.io/) 来处理所有的 HTTP 请求。

## 2。Maven 设置

首先，让我们添加必要的依赖项:

```
<dependency>
    <groupId>com.mashape.unirest</groupId>
    <artifactId>unirest-java</artifactId>
    <version>1.4.9</version>
</dependency>
```

点击查看最新版本[。](https://web.archive.org/web/20220626205115/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.mashape.unirest%22%2C%20a%3A%22unirest-java%22)

## 3。简单请求

让我们发送一个简单的 HTTP 请求，以理解框架的语义:

```
@Test
public void shouldReturnStatusOkay() {
    HttpResponse<JsonNode> jsonResponse 
      = Unirest.get("http://www.mocky.io/v2/5a9ce37b3100004f00ab5154")
      .header("accept", "application/json").queryString("apiKey", "123")
      .asJson();

    assertNotNull(jsonResponse.getBody());
    assertEquals(200, jsonResponse.getStatus());
}
```

请注意，该 API 流畅、高效且非常易于阅读。

我们用`header()` 和`fields()` API 传递头和参数。

并且请求在`asJson()`方法调用中被调用；我们这里也有其他选项，比如`asBinary(), asString()`和`asObject().`

为了传递多个标题或字段，我们可以创建一个映射，并将它们分别传递给`.headers(Map<String, Object> headers)`和`.fields(Map<String, String> fields)`:

```
@Test
public void shouldReturnStatusAccepted() {
    Map<String, String> headers = new HashMap<>();
    headers.put("accept", "application/json");
    headers.put("Authorization", "Bearer 5a9ce37b3100004f00ab5154");

    Map<String, Object> fields = new HashMap<>();
    fields.put("name", "Sam Baeldung");
    fields.put("id", "PSP123");

    HttpResponse<JsonNode> jsonResponse 
      = Unirest.put("http://www.mocky.io/v2/5a9ce7853100002a00ab515e")
      .headers(headers).fields(fields)
      .asJson();

    assertNotNull(jsonResponse.getBody());
    assertEquals(202, jsonResponse.getStatus());
}
```

### 3.1。传递查询参数

为了将数据作为查询`String,`传递，我们将使用`queryString()` 方法:

```
HttpResponse<JsonNode> jsonResponse 
  = Unirest.get("http://www.mocky.io/v2/5a9ce37b3100004f00ab5154")
  .queryString("apiKey", "123")
```

### 3.2。使用路径参数

对于传递任何 URL 参数，我们可以使用`routeParam()`方法:

```
HttpResponse<JsonNode> jsonResponse 
  = Unirest.get("http://www.mocky.io/v2/5a9ce37b3100004f00ab5154/{userId}")
  .routeParam("userId", "123")
```

参数占位符名称必须与方法的第一个参数相同。

### 3.3。正文为的请求

如果我们的请求需要一个 string/JSON 主体，我们使用`body()`方法传递它:

```
@Test
public void givenRequestBodyWhenCreatedThenCorrect() {

    HttpResponse<JsonNode> jsonResponse 
      = Unirest.post("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
      .body("{\"name\":\"Sam Baeldung\", \"city\":\"viena\"}")
      .asJson();

    assertEquals(201, jsonResponse.getStatus());
}
```

### 3.4。对象映射器

为了在请求中使用`asObject()`或`body()`，我们需要定义我们的对象映射器。为了简单起见，我们将使用 Jackson 对象映射器。

让我们首先将以下依赖项添加到`pom.xml`:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.4</version>
</dependency>
```

在 Maven Central 上始终使用最新版本的[。](https://web.archive.org/web/20220626205115/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)

现在让我们配置我们的映射器:

```
Unirest.setObjectMapper(new ObjectMapper() {
    com.fasterxml.jackson.databind.ObjectMapper mapper 
      = new com.fasterxml.jackson.databind.ObjectMapper();

    public String writeValue(Object value) {
        return mapper.writeValueAsString(value);
    }

    public <T> T readValue(String value, Class<T> valueType) {
        return mapper.readValue(value, valueType);
    }
});
```

注意`setObjectMapper()`应该只被调用一次，用于设置映射器；一旦设置了映射器实例，它将用于所有请求和响应。

现在让我们使用一个定制的`Article`对象来测试新功能:

```
@Test
public void givenArticleWhenCreatedThenCorrect() {
    Article article 
      = new Article("ID1213", "Guide to Rest", "baeldung");
    HttpResponse<JsonNode> jsonResponse 
      = Unirest.post("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
      .body(article)
      .asJson();

    assertEquals(201, jsonResponse.getStatus());
}
```

## 4。请求方法

与任何 HTTP 客户端类似，该框架为每个 HTTP 动词提供了单独的方法:

帖子:

```
Unirest.post("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

放:

```
Unirest.put("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

获取:

```
Unirest.get("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

删除:

```
Unirest.delete("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

补丁:

```
Unirest.patch("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

选项:

```
Unirest.options("http://www.mocky.io/v2/5a9ce7663100006800ab515d")
```

## 5。响应方法

一旦我们得到响应，让我们检查状态代码和状态消息:

```
//...
jsonResponse.getStatus()

//...
```

提取邮件头:

```
//...
jsonResponse.getHeaders();
//...
```

获取响应正文:

```
//...
jsonResponse.getBody();
jsonResponse.getRawBody();
//...
```

注意，`getRawBody(),` 返回未解析的响应体的流，而`getBody()`使用前面部分定义的对象映射器返回已解析的体。

## 6。处理异步请求

Unirest 还能够处理异步请求——使用`java.util.concurrent.Future` 和回调方法:

```
@Test
public void whenAysncRequestShouldReturnOk() {
    Future<HttpResponse<JsonNode>> future = Unirest.post(
      "http://www.mocky.io/v2/5a9ce37b3100004f00ab5154?mocky-delay=10000ms")
      .header("accept", "application/json")
      .asJsonAsync(new Callback<JsonNode>() {

        public void failed(UnirestException e) {
            // Do something if the request failed
        }

        public void completed(HttpResponse<JsonNode> response) {
            // Do something if the request is successful
        }

        public void cancelled() {
            // Do something if the request is cancelled
        }
        });

    assertEquals(200, future.get().getStatus());
}
```

`com.mashape.unirest.http.async.Callback<T>`接口提供了三种方法，`failed()`、 `cancelled()`和 `completed().`

根据响应重写方法以执行必要的操作。

## 7 .**。文件上传**

要上传或发送文件作为请求的一部分，将一个`java.io.File` 对象作为一个名为 file:

```
@Test
public void givenFileWhenUploadedThenCorrect() {

    HttpResponse<JsonNode> jsonResponse = Unirest.post(
      "http://www.mocky.io/v2/5a9ce7663100006800ab515d")
      .field("file", new File("/path/to/file"))
      .asJson();

    assertEquals(201, jsonResponse.getStatus());
}
```

我们也可以使用`ByteStream:`

```
@Test
public void givenByteStreamWhenUploadedThenCorrect() {
    try (InputStream inputStream = new FileInputStream(
      new File("/path/to/file/artcile.txt"))) {
        byte[] bytes = new byte[inputStream.available()];
        inputStream.read(bytes);
        HttpResponse<JsonNode> jsonResponse = Unirest.post(
          "http://www.mocky.io/v2/5a9ce7663100006800ab515d")
          .field("file", bytes, "article.txt")
          .asJson();

        assertEquals(201, jsonResponse.getStatus());
    }
}
```

或者直接使用输入流，在`fields()` 方法中添加`ContentType.APPLICATION_OCTET_STREAM` 作为第二个参数:

```
@Test
public void givenInputStreamWhenUploadedThenCorrect() {
    try (InputStream inputStream = new FileInputStream(
      new File("/path/to/file/artcile.txt"))) {

        HttpResponse<JsonNode> jsonResponse = Unirest.post(
          "http://www.mocky.io/v2/5a9ce7663100006800ab515d")
          .field("file", inputStream, ContentType.APPLICATION_OCTET_STREAM, "article.txt").asJson();

        assertEquals(201, jsonResponse.getStatus());
    }
}
```

## 8。Unirest 配置

该框架还支持 HTTP 客户端的典型配置，如连接池、超时、全局头等。

让我们设置每条路由的连接数和最大连接数:

```
Unirest.setConcurrency(20, 5);
```

配置连接和套接字超时:

```
Unirest.setTimeouts(20000, 15000);
```

请注意，时间值以毫秒为单位。

现在让我们为所有请求设置 HTTP 头:

```
Unirest.setDefaultHeader("X-app-name", "baeldung-unirest");
Unirest.setDefaultHeader("X-request-id", "100004f00ab5");
```

我们可以随时清除全局标题:

```
Unirest.clearDefaultHeaders();
```

在某些时候，我们可能需要通过代理服务器发出请求:

```
Unirest.setProxy(new HttpHost("localhost", 8080));
```

需要注意的一个重要方面是优雅地关闭或退出应用程序。Unirest 产生一个后台事件循环来处理操作，我们需要在退出应用程序之前关闭该循环:

```
Unirest.shutdown();
```

## 9。结论

在本教程中，我们主要关注轻量级 HTTP 客户端框架——Unirest。我们使用了一些简单的例子，既有同步模式，也有异步模式。

最后，我们还使用了一些高级配置，如连接池、代理设置等。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626205115/https://github.com/eugenp/tutorials/tree/master/libraries-http)