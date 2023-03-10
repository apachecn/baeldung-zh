# Java HttpClient 基本认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpclient-basic-auth>

## 1。概述

在这个简短的教程中，我们将看看基本认证。我们将看到它是如何工作的，并配置 [Java `HttpClient`](/web/20220617075807/https://www.baeldung.com/java-9-http-client) 来使用这种认证。

## 2。基本认证

**基本认证是一种简单的认证方法。客户端可以通过用户名和密码进行身份验证。**这些凭证以特定的格式在`Authorization` HTTP 报头中发送。它以关键字`Basic`开头，后跟一个 base64 编码的值`username:password`。冒号在这里很重要。标题应该严格遵循这种格式。

例如，要使用`baeldung`用户名和`HttpClient`密码进行验证，我们必须发送这个头:

```java
Basic YmFlbGR1bmc6SHR0cENsaWVudA==
```

我们可以通过使用 base64 解码器并检查解码结果来验证它。

## 3。Java HttpClient

**[Java 9 引入了一个新的`HttpClient`](/web/20220617075807/https://www.baeldung.com/java-9-http-client) 作为孵化模块，在 Java 11 中被标准化。**我们将使用 Java 11，所以我们可以简单地从`java.net.http`包中导入它，无需任何额外的配置或依赖。

让我们从执行一个简单的 GET 请求开始，现在不需要任何认证:

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
  .GET()
  .uri(new URI("https://postman-echo.com/get"))
  .build();

HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

logger.info("Status {}", response.statusCode());
```

首先，我们创建一个`HttpClient`，它可以用来执行 HTTP 请求。其次，我们使用[生成器设计模式](/web/20220617075807/https://www.baeldung.com/creational-design-patterns#builder)创建一个`HttpRequest`。 [`GET`方法](/web/20220617075807/https://www.baeldung.com/java-9-http-client#2-specifying-the-http-method)设置请求的 HTTP 方法。 [`uri`方法](/web/20220617075807/https://www.baeldung.com/java-9-http-client#1-setting-uri)设置我们想要发送请求的 URL。

之后，我们使用客户端发送请求。`send`方法的第二个参数是一个[响应体处理程序](/web/20220617075807/https://www.baeldung.com/java-9-http-client#1-handling-response-body)。这告诉客户端我们希望将响应主体视为一个`String`。

让我们运行应用程序并检查日志。输出应该如下所示:

```java
INFO com.baeldung.httpclient.basicauthentication.HttpClientBasicAuthentication - Status 200
```

我们看到 HTTP 状态是 200，这意味着我们的请求成功了。在此之后，让我们看看如何处理认证。

## 4。使用 HttpClient 认证器

在我们配置身份验证之前，我们需要一个 URL 来测试它。让我们使用一个需要认证的 [Postman Echo](https://web.archive.org/web/20220617075807/https://learning.postman.com/docs/developer/echo-api/) 端点。首先，将之前的 URL 改为这个，并再次运行应用程序:

```java
HttpRequest request = HttpRequest.newBuilder()
  .GET()
  .uri(new URI("https://postman-echo.com/basic-auth"))
  .build();
```

让我们检查日志并寻找状态代码。这次我们收到了 HTTP 状态 401“未授权”。此响应代码意味着端点需要身份验证，但客户端没有发送任何凭据。

让我们更改我们的客户机，以便它发送所需的身份验证数据。**我们可以通过配置`HttpClient Builder`来做到这一点，我们的客户端将使用我们设置的凭证。**该端点接受用户名`“postman”`和密码`“password”`。让我们为我们的客户端添加一个`authenticator`:

```java
HttpClient client = HttpClient.newBuilder()
  .authenticator(new Authenticator() {
      @Override
      protected PasswordAuthentication getPasswordAuthentication() {
          return new PasswordAuthentication("postman", "password".toCharArray());
      }
  })
  .build(); 
```

让我们再次运行应用程序。现在请求成功了，我们收到了 HTTP 状态 200。

## 5.使用 HTTP 头进行身份验证

我们可以使用另一种方法来访问需要身份验证的端点。**我们从前面的章节中了解了`Authorization`头是如何构造的，所以我们可以手动设置它的值**。尽管这必须针对每个请求来完成，而不是通过认证器来设置一次。

让我们移除验证器，看看如何设置请求头。我们需要使用 [base64 编码](/web/20220617075807/https://www.baeldung.com/java-base64-encode-and-decode#1-java-8-basic-base64)来构造头值:

```java
private static final String getBasicAuthenticationHeader(String username, String password) {
    String valueToEncode = username + ":" + password;
    return "Basic " + Base64.getEncoder().encodeToString(valueToEncode.getBytes());
} 
```

让我们为`Authorization`头设置这个值并运行应用程序:

```java
HttpRequest request = HttpRequest.newBuilder()
  .GET()
  .uri(new URI("https://postman-echo.com/basic-auth"))
  .header("Authorization", getBasicAuthenticationHeader("postman", "password"))
  .build();
```

我们的请求是成功的，这意味着我们正确地构造和设置了头值。

## 6。结论

在这个简短的教程中，我们看到了什么是基本身份验证以及它是如何工作的。我们通过为 Java `HttpClient`设置一个`authenticator`来使用 Java`HttpClient`进行基本认证。我们使用不同的方法通过手动设置 HTTP 头来进行身份验证。

和往常一样，这些例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220617075807/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)