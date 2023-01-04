# 谷歌 Http 客户端指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/google-http-client>

## 1。概述

在本文中，我们将看一看用于 Java 的 Google HTTP 客户端库，这是一个快速、良好抽象的库，用于通过 HTTP 连接协议访问任何资源。

客户端的主要功能有:

*   一个 HTTP 抽象层，可以让你解耦任何底层的库
*   快速、高效和灵活的 HTTP 响应和请求内容的 JSON 和 XML 解析模型
*   易于使用的 HTTP 资源映射注释和抽象

该库也可以在 Java 5 和更高版本中使用，这使得它成为遗留(SE 和 EE)项目的一个重要选择。

在本文中，我们将开发一个简单的应用程序，**将连接到 GitHub API 并检索用户**，同时涵盖库的一些最有趣的特性。

## 2。Maven 依赖关系

要使用这个库，我们需要`google-http-client`依赖项:

```
<dependency>
    <groupId>com.google.http-client</groupId>
    <artifactId>google-http-client</artifactId>
    <version>1.23.0</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220625231214/https://search.maven.org/classic/#search%7Cga%7C1%7Cgoogle-http-client) 找到。

## 3.提出一个简单的请求

让我们从向 GitHub 页面发出一个简单的 GET 请求开始，展示 Google Http 客户端是如何开箱即用的:

```
HttpRequestFactory requestFactory
  = new NetHttpTransport().createRequestFactory();
HttpRequest request = requestFactory.buildGetRequest(
  new GenericUrl("https://github.com"));
String rawResponse = request.execute().parseAsString()
```

要提出最简单的要求，我们至少需要:

*   这用于构建我们的请求
*   低层 HTTP 传输层的抽象
*   包装 Url 的类
*   `HttpRequest` 处理请求的实际执行

在接下来的小节中，我们将通过一个返回 JSON 格式的实际 API 来研究所有这些和一个更复杂的例子。

## 4。可插拔 HTTP 传输

这个库有一个很好抽象的`HttpTransport`类，允许我们在它的基础上构建和**改变底层的底层 HTTP 传输库选择**:

```
public class GitHubExample {
    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
}
```

在这个例子中，我们使用的是`NetHttpTransport`，它基于所有 Java SDKs 中都有的`HttpURLConnection`。这是一个很好的开始选择，因为它是众所周知的和可靠的。

当然，可能会有这样的情况，我们需要一些高级定制，因此需要一个更复杂的低级库。

对于这种情况，有`ApacheHttpTransport:`

```
public class GitHubExample {
    static HttpTransport HTTP_TRANSPORT = new ApacheHttpTransport();
}
```

`ApacheHttpTransport`是基于流行的 [Apache HttpClient](https://web.archive.org/web/20220625231214/https://hc.apache.org/httpcomponents-client-ga/index.html) 的，它包含了配置连接的多种选择。

此外，该库还提供了构建底层实现的选项，因此非常灵活。

## 5。JSON 解析

Google Http 客户端包括另一个用于 JSON 解析的抽象。这样做的一个主要优点是**低级解析库的选择是可互换的**。

有三个内置的选择，它们都扩展了`JsonFactory,`，并且还包括实现我们自己的选择的可能性。

### 5.1。可互换解析库

在我们的例子中，我们将使用 Jackson2 实现，这需要 [`google-http-client-jackson2`](https://web.archive.org/web/20220625231214/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22google-http-client-jackson2%22) 的依赖关系:

```
<dependency>
    <groupId>com.google.http-client</groupId>
    <artifactId>google-http-client-jackson2</artifactId>
    <version>1.23.0</version>
</dependency>
```

接下来，我们现在可以包括`JsonFactory:`

```
public class GitHubExample {

    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
    staticJsonFactory JSON_FACTORY = new JacksonFactory();
}
```

**`JacksonFactory`是用于解析/序列化操作的最快和最流行的库。**

这是以库的大小为代价的(在某些情况下这可能是一个问题)。为此，Google 还提供了`GsonFactory`，它是 Google GSON 库的一个实现，是一个轻量级的 JSON 解析库。

也有可能编写我们的低级解析器实现。

### 5.2。@ `Key`注解

我们可以使用`@Key`注释来指示需要从 JSON 解析或序列化到 JSON 的字段:

```
public class User {

    @Key
    private String login;
    @Key
    private long id;
    @Key("email")
    private String email;

    // standard getters and setters
}
```

这里我们正在做一个`User`抽象，它是我们从 GitHub API 批量接收的(我们将在本文后面进行实际的解析)`.`

请注意，没有 `@Key`注释的**字段被认为是内部的，不会从 JSON** 解析或序列化到 JSON【】。此外，字段的可见性无关紧要，getter 或 setter 方法的存在也无关紧要。

我们可以指定`@Key`注释的值，将其映射到正确的 JSON 键。

### 5.3。`GenericJson`

只有我们声明并标记为`@Key`的字段才会被解析。

为了保留其他内容，我们可以声明我们的类来扩展`GenericJson:`

```
public class User extends GenericJson {
    //...
}
```

**`GenericJson`实现了`Map`接口，这意味着我们可以使用 get 和 put 方法来设置/获取请求/响应中的 JSON 内容。**

## 6。打电话

为了用 Google Http 客户端连接到一个端点，我们需要一个`HttpRequestFactory`，它将用我们之前的抽象`HttpTransport`和`JsonFactory:`来配置

```
public class GitHubExample {

    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
    static JsonFactory JSON_FACTORY = new JacksonFactory();

    private static void run() throws Exception {
        HttpRequestFactory requestFactory 
          = HTTP_TRANSPORT.createRequestFactory(
            (HttpRequest request) -> {
              request.setParser(new JsonObjectParser(JSON_FACTORY));
          });
    }
}
```

接下来我们需要一个 URL 来连接。该库将此作为一个扩展`GenericUrl` 的类来处理，在该类上声明的任何字段都被视为查询参数:

```
public class GitHubUrl extends GenericUrl {

    public GitHubUrl(String encodedUrl) {
        super(encodedUrl);
    }

    @Key
    public int per_page;

}
```

这里，在我们的`GitHubUrl,` 中，我们声明了`per_page`属性，以指示在对 GitHub API 的一次调用中我们想要多少用户。

让我们继续使用`GitHubUrl:`构建我们的通话

```
private static void run() throws Exception {
    HttpRequestFactory requestFactory
      = HTTP_TRANSPORT.createRequestFactory(
        (HttpRequest request) -> {
          request.setParser(new JsonObjectParser(JSON_FACTORY));
        });
    GitHubUrl url = new GitHubUrl("https://api.github.com/users");
    url.per_page = 10;
    HttpRequest request = requestFactory.buildGetRequest(url);
    Type type = new TypeToken<List<User>>() {}.getType();
    List<User> users = (List<User>)request
      .execute()
      .parseAs(type);
}
```

注意我们如何指定 API 调用需要多少用户，然后我们用`HttpRequestFactory`构建请求。

接下来，由于 GitHub API 的响应包含用户列表，我们需要提供一个复杂的`Type`，也就是一个`List<User>`。

然后，在最后一行，我们进行调用并解析对我们的*用户*类列表的响应。

## 7.自定义标题

当发出一个 API 请求时，我们通常会做的一件事是包含某种自定义头，甚至是修改过的头:

```
HttpHeaders headers = request.getHeaders();
headers.setUserAgent("Baeldung Client");
headers.set("Time-Zone", "Europe/Amsterdam");
```

我们通过在创建请求之后、执行请求和添加必要的值之前获取`HttpHeaders` 来实现这一点。

请注意，Google Http 客户端**包含了一些作为特殊方法的头。**以`User-Agent` 头为例，如果我们试图用 set 方法包含它，它将抛出一个错误。

## 8。指数补偿

Google Http 客户端的另一个重要特性是基于某些状态代码和阈值重试请求的可能性。

我们可以在创建请求对象后立即包含指数补偿设置:

```
ExponentialBackOff backoff = new ExponentialBackOff.Builder()
  .setInitialIntervalMillis(500)
  .setMaxElapsedTimeMillis(900000)
  .setMaxIntervalMillis(6000)
  .setMultiplier(1.5)
  .setRandomizationFactor(0.5)
  .build();
request.setUnsuccessfulResponseHandler(
  new HttpBackOffUnsuccessfulResponseHandler(backoff));
```

**在`HttpRequest`** 中默认关闭指数补偿，所以我们必须在`HttpRequest`中包含一个`HttpUnsuccessfulResponseHandler`的实例来激活它。

## 9。记录日志

Google Http 客户端使用`java.util.logging.Logger`来记录 Http 请求和响应细节，包括 URL、标题和内容。

通常，使用`logging.properties`文件来管理日志记录:

```
handlers = java.util.logging.ConsoleHandler
java.util.logging.ConsoleHandler.level = ALL
com.google.api.client.http.level = ALL
```

在我们的例子中，我们使用`ConsoleHandler`，但是也可以选择`FileHandler`。

属性文件配置 JDK 日志记录工具的操作。可以将此配置文件指定为系统属性:

```
-Djava.util.logging.config.file=logging.properties
```

因此，在设置了文件和系统属性之后，库将生成如下所示的日志:

```
-------------- REQUEST  --------------
GET https://api.github.com/users?page=1&per;_page=10
Accept-Encoding: gzip
User-Agent: Google-HTTP-Java-Client/1.23.0 (gzip)

Nov 12, 2017 6:43:15 PM com.google.api.client.http.HttpRequest execute
curl -v --compressed -H 'Accept-Encoding: gzip' -H 'User-Agent: Google-HTTP-Java-Client/1.23.0 (gzip)' -- 'https://api.github.com/users?page=1&per;_page=10'
Nov 12, 2017 6:43:16 PM com.google.api.client.http.HttpResponse 
-------------- RESPONSE --------------
HTTP/1.1 200 OK
Status: 200 OK
Transfer-Encoding: chunked
Server: GitHub.com
Access-Control-Allow-Origin: *
...
Link: <https://api.github.com/users?page=1&per;_page=10&since;=19>; rel="next", <https://api.github.com/users{?since}>; rel="first"
X-GitHub-Request-Id: 8D6A:1B54F:3377D97:3E37B36:5A08DC93
Content-Type: application/json; charset=utf-8
...
```

## 10。结论

在本教程中，我们展示了 Google HTTP Client Library for Java 及其更有用的特性。他们的 [Github](https://web.archive.org/web/20220625231214/https://github.com/google/google-http-java-client) 包含更多关于它的信息以及库的源代码。

和往常一样，本教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625231214/https://github.com/eugenp/tutorials/tree/master/libraries-http)