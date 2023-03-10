# 在 Java 中将 HTTP 响应体作为字符串读取

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-http-response-body-as-string>

## 1.介绍

在本教程中，我们将探索几个在 Java 中将 HTTP 响应体作为字符串读取的库。从第一个版本开始，Java 就提供了 [`HttpURLConnection`](/web/20220926195223/https://www.baeldung.com/java-http-request) API。这仅包括基本的功能，并且众所周知对用户不太友好。

在 JDK 11 中，Java 引入了新的改进的`[HttpClient](/web/20220926195223/https://www.baeldung.com/java-9-http-client)` API 来处理 HTTP 通信。我们将讨论这些库，并检查替代库，如 [Apache HttpClient](/web/20220926195223/https://www.baeldung.com/httpclient-guide) 和 [Spring Rest 模板](/web/20220926195223/https://www.baeldung.com/rest-template)。

## 2.`HttpClient`

如前所述， [`HttpClient`](/web/20220926195223/https://www.baeldung.com/java-9-http-client) 被添加到 Java 11 中。它允许我们通过网络访问资源，但与`HttpURLConnection`、**、`HttpClient`不同的是，它支持 HTTP/1.1 和 HTTP/2** 。此外，它**提供同步和异步请求类型**。

提供了一个现代化的 API，具有很大的灵活性和强大的功能。这个 API 由三个核心类组成:`HttpClient`、`HttpRequest`和`HttpResponse`。

`HttpResponse`描述了一个`HttpRequest`调用的结果。 **`HttpResponse`不是直接创造出来的，是身体被完全接收后才可用的。**

要将响应体作为`String,`读取，我们首先需要创建简单的客户端和请求对象:

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(DUMMY_URL))
    .build();
```

然后我们将使用`BodyHandlers`并调用方法`ofString()`来返回响应:

```java
HttpResponse response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

## 3.`HttpURLConnection`

[`HttpURLConnection`](/web/20220926195223/https://www.baeldung.com/java-http-request) 是一个轻量级 HTTP 客户端，用于通过 HTTP 或 HTTPS 协议访问资源，它允许我们创建一个`InputStream`。一旦我们获得了`InputStream,`，我们就可以像读取一个普通的本地文件一样读取它。

在 Java 中，我们可以用来访问互联网的主要类是`java.net.URL`类和`java.net.HttpURLConnection`类。首先，我们将使用`URL`类来指向一个 web 资源。然后我们可以通过使用`HttpURLConnection`类来访问它。

为了将来自`URL`的响应体作为`String`，我们应该首先**使用我们的`URL`创建一个`HttpURLConnection`** :

```java
HttpURLConnection connection = (HttpURLConnection) new URL(DUMMY_URL).openConnection();
```

`new URL(DUMMY_URL).openConnection()`返回一个`HttpURLConnection`。这个对象允许我们添加标题或检查响应代码。

接下来，我们将**从`connection`** 对象中获取`InputStream`:

```java
InputStream inputStream = connection.getInputStream();
```

最后，我们需要用 **[将`InputStream`转换成`String`](/web/20220926195223/https://www.baeldung.com/convert-input-stream-to-string)** 。

## 4.阿帕奇`HttpClient`

在这一节中，我们将学习如何使用 [Apache `HttpClient`](/web/20220926195223/https://www.baeldung.com/httpclient-guide) 将 HTTP 响应体作为字符串读取。

要使用这个库，我们需要将它的依赖项添加到我们的 Maven 项目中:

```java
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.12</version>
</dependency>
```

我们可以通过`CloseableHttpClient`类来**检索和发送数据。要用默认配置创建它的实例，我们可以使用`HttpClients.createDefault()`。**

`CloseableHttpClient`提供了一个`execute`方法来发送和接收数据。这个方法使用了一个`HttpUriRequest`类型的参数，它有很多子类，包括`HttpGet`和`HttpPost`。

首先，我们将**创建一个`HttpGet`对象**:

```java
HttpGet request = new HttpGet(DUMMY_URL);
```

其次，我们将**创建客户端**:

```java
CloseableHttpClient client = HttpClients.createDefault();
```

然后我们将**从`execute`方法的结果中检索响应对象**:

```java
CloseableHttpResponse response = client.execute(request);
```

最后，我们将通过**将响应实体转换为`String`** 来返回响应体:

```java
HttpEntity entity = response.getEntity();
String result = EntityUtils.toString(entity);
```

## 5.弹簧`RestTemplate`

在这一节中，我们将演示如何使用 [Spring `RestTemplate`](/web/20220926195223/https://www.baeldung.com/rest-template) 以字符串形式读取 HTTP 响应主体。**我们必须注意到 RestTemplate 现在已经过时了。**因此，我们应该考虑使用弹簧`WebClient`，如下一节所述。

`RestTemplate`类是 Spring 提供的一个基本工具，它为**提供了一个简单的模板，用于在底层 HTTP 客户端库(如 JDK `HttpURLConnection`、阿帕奇`HttpClient`等)上进行客户端 HTTP 操作**。

`RestTemplate`为[提供了一些创建 HTTP 请求和处理响应的有用方法](https://web.archive.org/web/20220926195223/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)。

我们可以通过首先向我们的 Maven 项目添加一些依赖项来使用这个库:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>${spring-boot.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>${spring-boot.version}</version>
    <scope>test</scope>
</dependency>
```

为了发出 web 请求并以字符串形式返回响应体，我们将**创建一个`RestTemplate`** 的实例:

```java
RestTemplate restTemplate = new RestTemplate();
```

然后，我们将通过调用方法`getForObject()`并传入 URL 和所需的响应类型来**获得响应对象。**我们将在示例中使用`String.class`:

```java
String response = restTemplate.getForObject(DUMMY_URL, String.class);
```

## 6.弹簧`WebClient`

最后，我们来看看如何用 [**弹簧`WebClient,`**](/web/20220926195223/https://www.baeldung.com/spring-5-webclient) **无功、无阻塞解决方案替代弹簧** `**RestTemplate**.`

我们可以通过将 [`spring-boot-starter-webflux`](https://web.archive.org/web/20220926195223/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-webflux%22%20AND%20g%3A%22org.springframework.boot%22) 依赖项添加到我们的 Maven 项目来使用这个库:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

执行 HTTP Get 请求的最简单方法是使用`create`方法:

```java
WebClient webClient = WebClient.create(DUMMY_URL);
```

执行 HTTP Get 请求最简单的方法是调用`get`和`retrieve`方法。**然后我们将使用`bodyToMono`方法和`String.class`类型**来提取主体作为单个字符串实例:

```java
Mono<String> body = webClient.get().retrieve().bodyToMono(String.class);
```

最后，我们将**调用`block`方法告诉 web flux 等待，直到整个主体流被读取**并被复制到字符串结果中:

```java
String s = body.block();
```

## 7.结论

在本文中，我们学习了如何使用几个库来读取 HTTP 响应体作为`String`。

像往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220926195223/https://github.com/eugenp/tutorials/tree/master/apache-httpclient-2)