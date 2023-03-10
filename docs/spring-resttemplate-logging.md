# Spring RestTemplate 请求/响应日志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-logging>

## 1.概观

在本教程中，我们将学习如何实现高效的`RestTemplate`请求/响应日志记录。这对于调试两台服务器之间的交换尤其有用。

不幸的是，Spring Boot 没有提供检查或记录简单 JSON 响应体的简单方法。

我们将探索几种方法来记录 HTTP 头或者最有趣的部分，HTTP 主体。

**注**:弹簧`RestTemplate`将不再使用，取而代之的是`WebClient`。你可以在这里找到使用`WebClient`的类似文章:[Logging Spring WebClient Calls](/web/20220804184856/https://www.baeldung.com/spring-log-webclient-calls)。

## 2.使用 RestTemplate 的基本日志记录

让我们开始**配置`application.properties`** 文件中的`RestTemplate`记录器:

```java
logging.level.org.springframework.web.client.RestTemplate=DEBUG
```

因此，**只能看到基本信息**，比如请求 URL、方法、主体和响应状态:

```java
o.s.w.c.RestTemplate - HTTP POST http://localhost:8082/spring-rest/persons
o.s.w.c.RestTemplate - Accept=[text/plain, application/json, application/*+json, */*]
o.s.w.c.RestTemplate - Writing [my request body] with org.springframework.http.converter.StringHttpMessageConverter
o.s.w.c.RestTemplate - Response 200 OK
```

然而，**响应主体没有记录在这里**，这很不幸，因为这是最有趣的部分。

为了解决这个问题，我们将选择 Apache HttpClient 或 Spring 拦截器。

## 3.使用 Apache HttpClient 记录标题/正文

首先，我们必须让`RestTemplate`使用 [Apache HttpClient](https://web.archive.org/web/20220804184856/https://hc.apache.org/httpcomponents-client-4.5.x/index.html) 实现。

我们将需要[Maven 依赖](https://web.archive.org/web/20220804184856/https://search.maven.org/artifact/org.apache.httpcomponents/httpclient):

```java
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.12</version>
</dependency>
```

当创建`RestTemplate`实例时，**我们应该告诉它我们正在使用 Apache HttpClient** :

```java
RestTemplate restTemplate = new RestTemplate();
restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
```

然后，让我们在`application.properties`文件中配置客户端记录器:

```java
logging.level.org.apache.http=DEBUG
logging.level.httpclient.wire=DEBUG
```

现在我们可以看到请求/响应头和主体:

```java
 o.a.http.headers - http-outgoing-0 >> POST /spring-rest/persons HTTP/1.1
    o.a.http.headers - http-outgoing-0 >> Accept: text/plain, application/json, application/*+json, */*
// ... more request headers
    o.a.http.headers - http-outgoing-0 >> User-Agent: Apache-HttpClient/4.5.9 (Java/1.8.0_171)
    o.a.http.headers - http-outgoing-0 >> Accept-Encoding: gzip,deflate
org.apache.http.wire - http-outgoing-0 >> "POST /spring-rest/persons HTTP/1.1[\r][\n]"
org.apache.http.wire - http-outgoing-0 >> "Accept: text/plain, application/json, application/*+json, */*[\r][\n]"
org.apache.http.wire - http-outgoing-0 >> "Content-Type: text/plain;charset=ISO-8859-1[\r][\n]"
// ... more request headers
org.apache.http.wire - http-outgoing-0 >> "[\r][\n]"
org.apache.http.wire - http-outgoing-0 >> "my request body"
org.apache.http.wire - http-outgoing-0 << "HTTP/1.1 200 [\r][\n]"
org.apache.http.wire - http-outgoing-0 << "Content-Type: application/json[\r][\n]"
// ... more response headers
org.apache.http.wire - http-outgoing-0 << "Connection: keep-alive[\r][\n]"
org.apache.http.wire - http-outgoing-0 << "[\r][\n]"
org.apache.http.wire - http-outgoing-0 << "21[\r][\n]"
org.apache.http.wire - http-outgoing-0 << "["Lucie","Jackie","Danesh","Tao"][\r][\n]" 
```

然而，**这些日志很冗长，不便于调试**。

我们将在下一章看到如何解决这个问题。

## 4.用 RestTemplate 拦截器记录主体

作为另一种解决方案，我们可以为`RestTemplate` 配置[拦截器。](/web/20220804184856/https://www.baeldung.com/spring-rest-template-interceptor)

### 4.1.日志拦截器实现

首先，让我们**创建一个新的`LoggingInterceptor`来定制我们的日志**。这个拦截器将请求体记录为一个简单的字节数组。但是，对于响应，我们必须阅读整个正文流:

```java
public class LoggingInterceptor implements ClientHttpRequestInterceptor {

    static Logger LOGGER = LoggerFactory.getLogger(LoggingInterceptor.class);

    @Override
    public ClientHttpResponse intercept(
      HttpRequest req, byte[] reqBody, ClientHttpRequestExecution ex) throws IOException {
        LOGGER.debug("Request body: {}", new String(reqBody, StandardCharsets.UTF_8));
        ClientHttpResponse response = ex.execute(req, reqBody);
        InputStreamReader isr = new InputStreamReader(
          response.getBody(), StandardCharsets.UTF_8);
        String body = new BufferedReader(isr).lines()
            .collect(Collectors.joining("\n"));
        LOGGER.debug("Response body: {}", body);
        return response;
    }
}
```

注意，这个拦截器对响应内容本身有影响，我们将在下一章中发现。

### 4.2.通过 RestTemplate 使用拦截器

现在，我们必须处理一个流问题:**当拦截器使用响应流时，我们的客户端应用程序将看到一个空的响应体**。

为了避免这种情况，我们应该使用`BufferingClientHttpRequestFactory`:它将流内容缓冲到内存中。这样，它可以被读取两次:一次由我们的拦截器读取，另一次由我们的客户端应用程序读取:

```java
ClientHttpRequestFactory factory = 
        new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory());
        RestTemplate restTemplate = new RestTemplate(factory);
```

然而，使用这个工厂涉及到一个性能缺陷，我们将在下一小节中描述。

然后我们可以将日志拦截器添加到`RestTemplate`实例中——我们将把它附加在现有拦截器之后，如果有的话:

```java
List<ClientHttpRequestInterceptor> interceptors = restTemplate.getInterceptors();
if (CollectionUtils.isEmpty(interceptors)) {
    interceptors = new ArrayList<>();
}
interceptors.add(new LoggingInterceptor());
restTemplate.setInterceptors(interceptors); 
```

因此，日志中只显示必要的信息:

```java
c.b.r.l.LoggingInterceptor - Request body: my request body
c.b.r.l.LoggingInterceptor - Response body: ["Lucie","Jackie","Danesh","Tao"]
```

### 4.3.RestTemplate 拦截器缺陷

前面提到过，使用`BufferingClientHttpRequestFactory`有一个严重的缺点:它取消了流式传输的好处。因此，**将整个身体数据加载到内存中会使我们的应用程序面临性能问题。更糟糕的是，会导致`OutOfMemoryError`** 。

为了防止这种情况，一个可能的选择是假设当数据量增加时，这些详细日志将被关闭，这通常发生在生产中。例如，只有在我们的记录器上启用了`DEBUG`级别时，我们才能**使用缓冲的`RestTemplate`实例:**

```java
RestTemplate restTemplate = null;
if (LOGGER.isDebugEnabled()) {
    ClientHttpRequestFactory factory 
    = new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory());
    restTemplate = new RestTemplate(factory);
} else {
    restTemplate = new RestTemplate();
}
```

类似地，我们将**确保我们的拦截器只在`DEBUG`日志记录被启用时读取响应**:

```java
if (LOGGER.isDebugEnabled()) {
    InputStreamReader isr = new InputStreamReader(response.getBody(), StandardCharsets.UTF_8);
    String body = new BufferedReader(isr)
        .lines()
        .collect(Collectors.joining("\n"));
    LOGGER.debug("Response body: {}", body);
}
```

## 5.结论

请求/响应日志记录并不是一件简单的事情，因为 Spring Boot 并没有开箱即用。

幸运的是，我们已经看到**我们可以使用 Apache HttpClient logger 来获得交换数据的详细跟踪**。

或者，我们可以实现一个**定制拦截器来获得更多人类可读的日志**。但是，对于大数据量，这可能会导致性能下降。

和往常一样，这篇文章的源代码可以在 GitHub 的`test`文件夹中的[找到。该示例在同一个项目中定义的 REST 端点的现场测试中使用了`RestTemplate`。](https://web.archive.org/web/20220804184856/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)