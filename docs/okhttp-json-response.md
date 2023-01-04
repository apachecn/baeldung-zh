# 解码 OkHttp JSON 响应

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/okhttp-json-response>

## 1.介绍

在本教程中，我们将探索几种使用 [OkHttp](/web/20220628125656/https://www.baeldung.com/guide-to-okhttp) 解码 JSON 响应的技术。

## 2.OkHttp `Response`

OkHttp 是一个用于 Java 和 Android 的 Http 客户端，具有透明处理 GZIP、响应缓存和网络故障恢复等特性。

尽管有这些很棒的特性，OkHttp 没有内置的用于 JSON、XML 和其他内容类型的编码器/解码器。然而，我们可以在 XML/JSON 绑定库的帮助下实现这些，或者我们可以使用高级库，如 [Feign](/web/20220628125656/https://www.baeldung.com/intro-to-feign) 或[改型](/web/20220628125656/https://www.baeldung.com/retrofit)。

为了实现我们的 JSON 解码器，我们需要从服务调用的结果中提取 JSON。为此，我们可以通过`the Response`对象的`body()`方法来访问主体。`ResponseBody`类有几个提取数据的选项:

*   `**byteStream()**`:将正文的原始字节暴露为一个`InputStream`；我们可以将它用于所有格式，但通常它用于二进制文件和文件
*   `**charStream()**`:当我们有一个文本响应时，`charStream()`将它的`InputStream` 包装在一个`Reader`中，并根据响应的内容类型或“UTF-8”来处理编码，如果在响应头中没有设置字符集的话；然而，当使用`charStream()`时，我们不能改变`Reader`的编码
*   `**string()**`:返回整个响应体为`String`；管理编码与`charStream()`相同，但是如果我们需要不同的编码，我们可以用 **`source().readString(charset)`** 代替

在本文中，我们将使用`string()` ，因为我们的响应很小，并且我们没有内存或性能问题。当性能和内存很重要时，`byteStream()`和`charStream()`方法是生产系统中更好的选择。

首先，让我们将 [okhttp](https://web.archive.org/web/20220628125656/https://search.maven.org/search?q=g:com.squareup.okhttp3%20a:okhttp) 添加到 pom.xml 文件中:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId> 
    <version>3.14.2</version> 
</dependency>
```

然后，我们模拟`SimpleEntity`来测试我们的解码器:

```java
public class SimpleEntity {
    protected String name;

    public SimpleEntity(String name) {
        this.name = name;
    }

    // no-arg constructor, getters, and setters
} 
```

现在，我们将开始我们的测试:

```java
SimpleEntity sampleResponse = new SimpleEntity("Baeldung");

OkHttpClient client = // build an instance;
MockWebServer server = // build an instance;
Request request = new Request.Builder().url(server.url("...")).build();
```

## 3.与杰克逊一起解码`ResponseBody`

Jackson 是最流行的 JSON-Object 绑定库之一。

让我们将 [jackson-databind](https://web.archive.org/web/20220628125656/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20a:jackson-databind) 添加到 pom.xml 中:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

Jackson 的`ObjectMapper` 让我们将 JSON 转换成一个对象。因此，我们可以使用`ObjectMapper.readValue()`对响应进行解码:

```java
ObjectMapper objectMapper = new ObjectMapper(); 
ResponseBody responseBody = client.newCall(request).execute().body(); 
SimpleEntity entity = objectMapper.readValue(responseBody.string(), SimpleEntity.class);

Assert.assertNotNull(entity);
Assert.assertEquals(sampleResponse.getName(), entity.getName());
```

## 4.用 Gson 解码`ResponseBody`

Gson 是另一个有用的库，用于将 JSON 映射到对象，反之亦然。

让我们将 [gson](https://web.archive.org/web/20220628125656/https://search.maven.org/search?q=g:com.google.code.gson%20a:gson) 添加到 pom.xml 文件中:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

让我们看看如何使用`Gson.fromJson()` 来解码响应体:

```java
Gson gson = new Gson(); 
ResponseBody responseBody = client.newCall(request).execute().body();
SimpleEntity entity = gson.fromJson(responseBody.string(), SimpleEntity.class);

Assert.assertNotNull(entity);
Assert.assertEquals(sampleResponse.getName(), entity.getName()); 
```

## 5.结论

在本文中，我们与 Jackson 和 Gson 一起探索了几种解码 OkHttp 的 JSON 响应的方法。

完整的样片可在 GitHub 上的[处获得。](https://web.archive.org/web/20220628125656/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)