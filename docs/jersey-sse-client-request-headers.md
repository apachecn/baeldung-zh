# 向 Jersey SSE 客户端请求添加标头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-sse-client-request-headers>

## 1.概观

在本教程中，我们将看到一种使用 Jersey 客户端 API 在[服务器发送的事件](/web/20220625164018/https://www.baeldung.com/spring-server-sent-events) (SSE)客户端请求中发送报头的简单方法。

我们还将介绍使用默认的 Jersey 传输连接器发送基本的键/值头、身份验证头和受限头的正确方法。

## 2.直截了当

可能我们在尝试使用 SSEs 发送报头时都遇到过这种情况:

我们使用一个`SseEventSource`来接收 SSEs，但是为了构建`SseEventSource`，我们需要一个`WebTarget`实例，它没有为我们提供添加头部的方法。`Client`实例也没有帮助。听起来熟悉吗？

**记住，头与 SSE 无关，而是与客户端请求本身有关，**所以我们真的应该在那里寻找。

让我们看看我们能用`ClientRequestFilter`做些什么。

## 3.属国

为了开始我们的旅程，我们需要在我们的 Maven `pom.xml`文件中有[的`jersey-client` 依赖关系](https://web.archive.org/web/20220625164018/https://search.maven.org/search?q=g:org.glassfish.jersey.core%20AND%20a:jersey-client&core=gav)以及[新泽西的 SSE 依赖关系](https://web.archive.org/web/20220625164018/https://search.maven.org/search?q=g:org.glassfish.jersey.media%20AND%20a:jersey-media-sse&core=gav):

```java
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>2.29</version>
</dependency>
```

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-sse</artifactId>
    <version>2.29</version>
</dependency>
```

请注意， **Jersey 从 2.29 开始支持 JAX-RS 2.1，**所以看起来我们将能够使用其中的功能。

## 4.`ClientRequestFilter`

首先，我们将实现向每个客户端请求添加报头的过滤器:

```java
public class AddHeaderOnRequestFilter implements ClientRequestFilter {

    public static final String FILTER_HEADER_VALUE = "filter-header-value";
    public static final String FILTER_HEADER_KEY = "x-filter-header";

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        requestContext.getHeaders().add(FILTER_HEADER_KEY, FILTER_HEADER_VALUE);
    }
}
```

之后，我们会注册并消费它。

对于我们的例子，我们将使用`https://sse.example.org`作为一个假想的端点，我们希望我们的客户端从这里消费事件。实际上，我们会将其更改为我们希望客户使用的真正的 [SSE 事件服务器端点](/web/20220625164018/https://www.baeldung.com/java-ee-jax-rs-sse)。

```java
Client client = ClientBuilder.newBuilder()
  .register(AddHeaderOnRequestFilter.class)
  .build();

WebTarget webTarget = client.target("https://sse.example.org/");

SseEventSource sseEventSource = SseEventSource.target(webTarget).build();
sseEventSource.register((event) -> { /* Consume event here */ });
sseEventSource.open();
// do something here until ready to close
sseEventSource.close();
```

现在，如果我们需要向我们的 SSE 端点发送更复杂的消息头，比如认证消息头，该怎么办？

让我们一起进入下一节，了解更多关于 Jersey 客户端 API 中的头的信息。

## 5.Jersey 客户端 API 中的标题

**重要的是要知道默认的球衣传输连接器实现使用了 JDK** 的`HttpURLConnection`类。这个类限制了某些头的使用。为了避免这种限制，我们可以设置系统属性:

```java
System.setProperty("sun.net.http.allowRestrictedHeaders", "true");
```

我们可以在球衣文件中找到一个限制头球的列表。

### 5.1.简单通用标题

定义头部最直接的方法是调用`WebTarget#request` 来获得提供`header`方法的`Invocation.Builder`。

```java
public Response simpleHeader(String headerKey, String headerValue) {
    Client client = ClientBuilder.newClient();
    WebTarget webTarget = client.target("https://sse.example.org/");
    Invocation.Builder invocationBuilder = webTarget.request();
    invocationBuilder.header(headerKey, headerValue);
    return invocationBuilder.get();
}
```

实际上，我们可以很好地压缩它以增加可读性:

```java
public Response simpleHeaderFluently(String headerKey, String headerValue) {
    Client client = ClientBuilder.newClient();

    return client.target("https://sse.example.org/")
      .request()
      .header(headerKey, headerValue)
      .get();
}
```

从这里开始，我们将只对示例使用流畅的形式，因为这样更容易理解。

### 5.2.基本认证

实际上，Jersey 客户端 API **提供了`HttpAuthenticationFeature`类，允许我们轻松地发送认证头**:

```java
public Response basicAuthenticationAtClientLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.basic(username, password);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
      .request()
      .get();
}
```

因为我们在构建客户端时注册了`feature`,所以它将应用于每个请求。API 处理基本规范所需的用户名和密码的编码。

顺便说一下，请注意我们暗示 HTTPS 是我们连接的模式。虽然这始终是一项有价值的安全措施，但在使用基本身份验证时，这是最基本的，否则，密码会暴露为明文。球衣还支持[更复杂的安全配置](https://web.archive.org/web/20220625164018/https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/user-guide.html#d0e5313)。

现在，我们也可以在请求时指定 creds:

```java
public Response basicAuthenticationAtRequestLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.basicBuilder().build();
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
      .request()
      .property(HTTP_AUTHENTICATION_BASIC_USERNAME, username)
      .property(HTTP_AUTHENTICATION_BASIC_PASSWORD, password)
      .get();
}
```

### 5.3.摘要认证

Jersey 的`HttpAuthenticationFeature` 也支持摘要认证:

```java
public Response digestAuthenticationAtClientLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.digest(username, password);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
      .request()
      .get();
}
```

同样，我们可以在请求时覆盖:

```java
public Response digestAuthenticationAtRequestLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.digest();
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("http://sse.example.org/")
      .request()
      .property(HTTP_AUTHENTICATION_DIGEST_USERNAME, username)
      .property(HTTP_AUTHENTICATION_DIGEST_PASSWORD, password)
      .get();
}
```

### 5.4.使用 OAuth 2.0 进行承载令牌认证

OAuth 2.0 支持承载令牌的概念作为另一种认证机制。

我们需要[新泽西的`oauth2-client` 属地](https://web.archive.org/web/20220625164018/https://search.maven.org/search?q=g:org.glassfish.jersey.security%20AND%20a:oauth2-client&core=gav)给我们`OAuth2ClientSupportFeature` ，类似于`HttpAuthenticationFeature`:

```java
<dependency>
    <groupId>org.glassfish.jersey.security</groupId>
    <artifactId>oauth2-client</artifactId>
    <version>2.29</version>
</dependency>
```

要添加不记名令牌，我们将遵循与前面类似的模式:

```java
public Response bearerAuthenticationWithOAuth2AtClientLevel(String token) {
    Feature feature = OAuth2ClientSupport.feature(token);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.examples.org/")
      .request()
      .get();
}
```

或者，我们可以在请求级别覆盖，这在令牌由于旋转而改变时特别方便:

```java
public Response bearerAuthenticationWithOAuth2AtRequestLevel(String token, String otherToken) {
    Feature feature = OAuth2ClientSupport.feature(token);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
      .request()
      .property(OAuth2ClientSupport.OAUTH2_PROPERTY_ACCESS_TOKEN, otherToken)
      .get();
}
```

### 5.5.使用 OAuth 1.0 进行承载令牌认证

第四，如果我们需要集成使用 OAuth 1.0 的遗留代码，我们将需要 [Jersey 的`oauth1-client` dependency](https://web.archive.org/web/20220625164018/https://search.maven.org/search?q=g:org.glassfish.jersey.security%20AND%20a:oauth1-client&core=gav) :

```java
<dependency>
    <groupId>org.glassfish.jersey.security</groupId>
    <artifactId>oauth1-client</artifactId>
    <version>2.29</version>
</dependency>
```

与 OAuth 2.0 类似，我们可以使用`OAuth1ClientSupport`:

```java
public Response bearerAuthenticationWithOAuth1AtClientLevel(String token, String consumerKey) {
    ConsumerCredentials consumerCredential = 
      new ConsumerCredentials(consumerKey, "my-consumer-secret");
    AccessToken accessToken = new AccessToken(token, "my-access-token-secret");

    Feature feature = OAuth1ClientSupport
      .builder(consumerCredential)
      .feature()
      .accessToken(accessToken)
      .build();

    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
      .request()
      .get();
}
```

请求级别再次由`OAuth1ClientSupport.OAUTH_PROPERTY_ACCESS_TOKEN` 属性启用。

## 6.结论

总之，在本文中，我们介绍了如何使用过滤器在 Jersey 中向 SSE 客户端请求添加头。我们还专门讨论了如何使用认证头。

这个例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625164018/https://github.com/eugenp/tutorials/tree/master/jersey)