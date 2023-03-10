# 在 OkHTTP 中添加拦截器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-okhttp-interceptors>

## 1。概述

通常是 当在我们的 web 应用程序中管理 HTTP 请求的请求和响应周期时，我们需要一种方法来利用这个链 。通常，这是 到 在我们完成请求之前添加一些自定义行为或者 在我们的 servlet 代码完成 之后简单地 。

OkHttp 是一款适用于 Android 和 Java 应用的高效 HTTP & HTTP/2 客户端。在之前的教程中，我们学习了如何使用 OkHttp 的[基础知识](/web/20220730173805/https://www.baeldung.com/guide-to-okhttp)。

在本教程中，**我们将学习如何拦截 HTTP 请求和响应对象**。

## 2.截击机

顾名思义，拦截器是可插入的 Java 组件，我们可以用它来拦截和处理发送给应用程序代码的请求。

同样，它们为我们在容器将响应发送回客户端之前处理服务器响应提供了一个强大的机制。

当我们想要改变 HTTP 请求中的某些内容时，比如添加一个新的控制头，改变请求的主体，或者简单地生成日志来帮助我们调试，这是非常有用的。

使用拦截器的另一个很好的特性是，它们让我们可以在一个地方封装常见的功能。假设我们想要对所有请求和响应对象全局应用一些逻辑，比如错误处理。

将这种逻辑放入拦截器至少有几个好处:

*   我们只需要在一个地方维护这些代码，而不是所有的端点
*   每个请求都以同样的方式处理错误

最后，我们还可以监控、重写和重试来自拦截器的调用。

## 3.常见用法

一些其他常见的任务，当一个 inceptor 可能是一个明显的选择，包括:

*   记录请求参数和其他有用的信息
*   向我们的请求添加身份验证和授权头
*   格式化我们的请求和响应主体
*   **压缩发送给客户端的响应数据**
*   通过添加一些 cookies 或额外的标题信息来改变我们的响应标题

在接下来的章节中，我们将会看到一些这样的例子。

## 4.属国

当然，我们需要将标准的 [`okhttp`依赖关系](https://web.archive.org/web/20220730173805/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22okhttp%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>
```

我们还需要另一个专门用于测试的依赖项。再来补充一下 OkHttp [`mockwebserver`神器](https://web.archive.org/web/20220730173805/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22mockwebserver%22):

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>4.9.1</version>
    <scope>test</scope>
</dependency>
```

既然我们已经配置了所有必要的依赖项，我们就可以继续编写我们的第一个拦截器了。

## 5.定义一个简单的日志拦截器

让我们从定义我们自己的拦截器开始。为了简单起见，我们的拦截器将记录请求头和请求 URL:

```java
public class SimpleLoggingInterceptor implements Interceptor {

    private static final Logger LOGGER = LoggerFactory.getLogger(SimpleLoggingInterceptor.class);

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();

        LOGGER.info("Intercepted headers: {} from URL: {}", request.headers(), request.url());

        return chain.proceed(request);
    }
}
```

正如我们所见，**要创建我们的拦截器，我们需要做的就是从`Interceptor`接口继承，它有一个强制方法`intercept(Chain chain)`。**然后我们可以继续用我们自己的实现覆盖这个方法。

首先，在打印出标题和请求 URL 之前，我们通过调用`chain.request()`来获取传入的请求。

**需要注意的是，每个拦截器实现的一个关键部分是对`chain.proceed(request)`的调用。**

这个看起来简单的方法是我们发出信号，我们想击中我们的应用程序代码，产生一个响应来满足请求。

### 5.1.将它们连接在一起

要真正利用这个拦截器，我们需要做的就是在构建我们的`OkHttpClient`实例时调用`addInterceptor`方法，它应该可以正常工作:

```java
OkHttpClient client = new OkHttpClient.Builder() 
  .addInterceptor(new SimpleLoggingInterceptor())
  .build();
```

我们可以根据需要继续调用`addInterceptor`方法来获得尽可能多的拦截器。请记住，它们将按照添加的顺序被调用。

### 5.2.测试拦截器

现在，我们已经定义了第一个拦截器；让我们继续编写我们的第一个集成测试:

```java
@Rule
public MockWebServer server = new MockWebServer();

@Test
public void givenSimpleLogginInterceptor_whenRequestSent_thenHeadersLogged() throws IOException {
    server.enqueue(new MockResponse().setBody("Hello Baeldung Readers!"));

    OkHttpClient client = new OkHttpClient.Builder()
      .addInterceptor(new SimpleLoggingInterceptor())
      .build();

    Request request = new Request.Builder()
      .url(server.url("/greeting"))
      .header("User-Agent", "A Baeldung Reader")
      .build();

    try (Response response = client.newCall(request).execute()) {
        assertEquals("Response code should be: ", 200, response.code());
        assertEquals("Body should be: ", "Hello Baeldung Readers!", response.body().string());
    }
}
```

首先，我们使用 OkHttp `MockWebServer` [JUnit 规则](/web/20220730173805/https://www.baeldung.com/junit-4-rules)。

这是一个轻量级的、可脚本化的 web 服务器，用于测试 HTTP 客户端，我们将用它来测试我们的拦截器。通过使用这个规则，我们将为每个集成测试创建一个干净的服务器实例。

记住这一点，现在让我们来看看测试的关键部分:

*   首先，我们设置一个模拟响应，其主体中包含一条简单的消息
*   然后，我们构建我们的`OkHttpClient`并配置我们的`SimpleLoggingInterceptor`
*   接下来，我们用一个`User-Agent`头设置将要发送的请求
*   **最后一步是发送请求，并验证收到的响应代码和正文是否符合预期**

### 5.3.运行测试

最后，当我们运行我们的测试时，我们将看到我们的 HTTP `User-Agent`头被记录:

```java
16:07:02.644 [main] INFO  c.b.o.i.SimpleLoggingInterceptor - Intercepted headers: User-Agent: A Baeldung Reader
 from URL: http://localhost:54769/greeting
```

### 5.4.使用内置的`HttpLoggingInterceptor`

虽然我们的日志拦截器很好地演示了我们如何定义拦截器，但值得一提的是，OkHttp 有一个内置的日志记录器，我们可以利用它。

为了使用这个记录器，我们需要一个额外的 [Maven 依赖项](https://web.archive.org/web/20220730173805/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22logging-interceptor%22):

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>logging-interceptor</artifactId>
    <version>4.9.1</version>
</dependency>
```

然后，我们可以实例化我们的日志记录器，并定义我们感兴趣的日志记录级别:

```java
HttpLoggingInterceptor logger = new HttpLoggingInterceptor();
logger.setLevel(HttpLoggingInterceptor.Level.HEADERS);
```

在本例中，我们只对查看标题感兴趣。

## 6.添加自定义响应标头

既然我们已经理解了创建拦截器背后的基础知识。现在让我们看看另一个典型的用例，我们修改了一个 HTTP 响应头。

如果我们想要添加我们自己的专有应用程序 HTTP 头或重写从我们的服务器返回的一个头，这可能是有用的:

```java
public class CacheControlResponeInterceptor implements Interceptor {

    @Override
    public Response intercept(Chain chain) throws IOException {
        Response response = chain.proceed(chain.request());
        return response.newBuilder()
          .header("Cache-Control", "no-store")
          .build();
    }
}
```

和以前一样，我们调用`chain.proceed`方法，但是这次没有事先使用请求对象。当响应返回时，我们用它来创建一个新的响应，并将`Cache-Control`头设置为`no-store`。

实际上，我们不太可能想让浏览器每次都从服务器拉取数据，但是我们可以使用这种方法在我们的响应中设置任何头。

## 7.使用拦截器的错误处理

如前所述，我们还可以使用拦截器来封装一些逻辑，我们希望将这些逻辑全局应用于所有的请求和响应对象，比如错误处理。

假设当响应不是 HTTP 200 响应时，我们希望返回一个轻量级 JSON 响应，并带有状态和消息。

记住这一点，我们将从定义一个简单的 bean 来保存错误消息和状态代码开始:

```java
public class ErrorMessage {

    private final int status;
    private final String detail;

    public ErrorMessage(int status, String detail) {
        this.status = status;
        this.detail = detail;
    }

    // Getters and setters
}
```

接下来，我们将创建拦截器:

```java
public class ErrorResponseInterceptor implements Interceptor {

    public static final MediaType APPLICATION_JSON = MediaType.get("application/json; charset=utf-8");

    @Override
    public Response intercept(Chain chain) throws IOException {
        Response response = chain.proceed(chain.request());

        if (!response.isSuccessful()) {
            Gson gson = new Gson();
            String body = gson.toJson(
              new ErrorMessage(response.code(), "The response from the server was not OK"));
            ResponseBody responseBody = ResponseBody.create(body, APPLICATION_JSON);

            ResponseBody originalBody = response.body();
            if (originalBody != null) {
                originalBody.close();
            }

            return response.newBuilder().body(responseBody).build();
        }
        return response;
    }
}
```

很简单，我们的拦截器检查响应是否成功，如果不成功，就创建一个包含响应代码和简单消息的 JSON 响应。注意，在这种情况下，我们必须记住关闭原始响应的主体，以释放与之相关的任何资源。

```java
{
    "status": 500,
    "detail": "The response from the server was not OK"
}
```

## 8.网络拦截器

到目前为止，我们已经介绍的拦截器就是 OkHttp 所说的应用程序拦截器。然而，OkHttp 也支持另一种叫做网络拦截器的拦截器。

我们可以用与前面解释的完全相同的方式来定义我们的网络拦截器。**然而，当我们创建 HTTP 客户端实例**时，我们需要调用`addNetworkInterceptor`方法:

```java
OkHttpClient client = new OkHttpClient.Builder()
  .addNetworkInterceptor(new SimpleLoggingInterceptor())
  .build();
```

应用程序和网络入侵者之间的一些重要差异包括:

*   **应用程序拦截器总是被调用一次，即使 HTTP 响应是由缓存提供的**
*   网络拦截器挂接到网络层，是放置重试逻辑的理想位置
*   同样，当我们的逻辑不依赖于响应的实际内容时，我们应该考虑使用网络拦截器
*   使用网络拦截器可以让我们访问承载请求的连接，包括 IP 地址和用于连接 web 服务器的 TLS 配置等信息。
*   应用程序拦截器不需要担心像重定向和重试这样的中间响应
*   相反，如果我们有重定向，网络拦截器可能会被调用不止一次

正如我们所见，这两种选择各有千秋。因此，我们选择哪一个实际上取决于我们自己的特定用例。

然而，通常情况下，应用程序拦截器会很好地完成这项工作。

## 9.结论

在本文中，我们已经了解了如何使用 OkHttp 创建拦截器。首先，我们开始解释什么是拦截器，以及我们如何定义一个简单的日志拦截器来检查我们的 HTTP 请求头。

然后我们看到了如何在我们的响应对象中设置一个 header 和一个不同的 body。最后，我们快速看了一下应用程序和网络拦截器之间的一些差异。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220730173805/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)