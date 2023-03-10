# REST API 错误处理的最佳实践

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-error-handling-best-practices>

## 1.概观

REST 是一种无状态架构，在这种架构中，客户端可以访问和操作服务器上的资源。一般来说，REST 服务利用 HTTP 来通告它们管理的一组资源，并提供一个 API 来允许客户机获得或改变这些资源的状态。

在本教程中，我们将了解一些处理 REST API 错误的最佳实践，包括为用户提供相关信息的有用方法、来自大型网站的示例以及使用示例 Spring REST 应用程序的具体实现。

## 延伸阅读:

## [带弹簧的刀架错误处理](/web/20220703113315/https://www.baeldung.com/exception-handling-for-rest-with-spring)

Exception Handling for a REST API - illustrate the new Spring 3.2 recommended approach as well as earlier solutions .[Read more](/web/20220703113315/https://www.baeldung.com/exception-handling-for-rest-with-spring) →

## [Spring RestTemplate 错误处理](/web/20220703113315/https://www.baeldung.com/spring-rest-template-error-handling)

Learn how to handle errors with Spring's RestTemplate[Read more](/web/20220703113315/https://www.baeldung.com/spring-rest-template-error-handling) →

## 【REST API 的自定义错误消息处理

Implement a Global Exception Handler for a REST API with Spring.[Read more](/web/20220703113315/https://www.baeldung.com/global-error-handler-in-a-spring-rest-api) →

## 2.HTTP 状态代码

当一个客户机向一个 HTTP 服务器发出一个请求——并且服务器成功地接收到请求— **服务器必须通知客户机请求是否被成功处理。**

HTTP 通过五类状态代码来实现这一点:

*   100 级(信息)-服务器确认请求
*   200 级(成功)-服务器按预期完成了请求
*   300 级(重定向)–客户端需要执行进一步的操作来完成请求
*   400 级(客户端错误)-客户端发送了无效请求
*   500 级(服务器错误)-由于服务器出错，服务器无法满足有效请求

根据响应代码，客户机可以推测特定请求的结果。

## 3.处理错误

处理错误的第一步是为客户端提供一个正确的状态代码。此外，我们可能需要在回复正文中提供更多信息。

### 3.1.基本反应

我们处理错误的最简单的方法是用一个适当的状态码来响应。

以下是一些常见的响应代码:

*   400 错误请求–客户端发送了一个无效请求，例如缺少必需的请求主体或参数
*   401 未授权–客户端无法通过服务器验证
*   403 禁止–客户端已通过身份验证，但无权访问所请求的资源
*   404 未找到–请求的资源不存在
*   412 前提条件失败–请求头字段中的一个或多个条件被评估为假
*   500 内部服务器错误–服务器上出现一般性错误
*   503 服务不可用–请求的服务不可用

虽然是基本的，这些代码允许客户理解所发生的错误的广泛性质。我们知道，如果我们收到一个 403 错误，例如，我们缺乏访问我们请求的资源的权限。然而，在许多情况下，我们需要在回复中提供补充细节。

500 错误表示在处理请求时服务器上发生了一些问题或异常。一般来说，这个内部错误与我们的客户无关。

因此，为了最大限度地减少对客户端的这类响应，我们应该努力尝试处理或捕捉内部错误，并尽可能用其他适当的状态代码进行响应。

例如，如果因为请求的资源不存在而发生异常，我们应该将它公开为 404 而不是 500 错误。

这并不是说 500 不应该被返回，只是说它应该用于意外的情况——比如服务中断——阻止服务器执行请求。

### 3.2.默认弹簧误差响应

这些原则无处不在，以至于 Spring 将它们编入了默认的[错误处理机制](/web/20220703113315/https://www.baeldung.com/exception-handling-for-rest-with-spring)。

为了演示，假设我们有一个简单的 Spring REST 应用程序,它管理书籍，有一个端点通过 ID 检索书籍:

```java
curl -X GET -H "Accept: application/json" http://localhost:8082/spring-rest/api/book/1
```

如果没有 ID 为 1 的书，我们预计控制器会抛出一个`BookNotFoundException`。

在这个端点上执行 GET 时，我们看到抛出了这个异常，这是响应体:

```java
{
    "timestamp":"2019-09-16T22:14:45.624+0000",
    "status":500,
    "error":"Internal Server Error",
    "message":"No message available",
    "path":"/api/book/1"
}
```

请注意，这个默认的错误处理程序包括错误发生时的时间戳、HTTP 状态代码、标题(`error`字段)、如果在默认错误中启用了 messages(默认情况下为空)的消息，以及发生错误的 URL 路径。

**这些字段为客户或开发人员提供信息，以帮助解决问题**,也构成了构成标准错误处理机制的几个字段。

还要注意，当抛出我们的`BookNotFoundException`时，Spring 会自动返回一个 HTTP 状态码 500。虽然有些 API 会返回 500 状态代码或其他通用代码，正如我们将在脸书和 Twitter APIs 中看到的那样，但为了简单起见，对于所有错误，最好尽可能使用最具体的错误代码。

在我们的例子中，我们可以添加一个`[@ControllerAdvice](/web/20220703113315/https://www.baeldung.com/exception-handling-for-rest-with-spring#controlleradvice)`，这样当一个`BookNotFoundException`被抛出时，我们的 API 返回一个状态 404 来表示`Not Found`，而不是 500 `Internal Server Error`。

### 3.3.更详细的答复

正如在上面的 Spring 示例中所看到的，有时状态代码不足以显示错误的细节。需要时，我们可以使用响应的主体向客户端提供附加信息。

在提供详细的回答时，我们应该包括:

*   错误–错误的唯一标识符
*   消息——简单易懂的消息
*   详细信息——对错误的详细解释

例如，如果一个客户端用不正确的凭证发送一个请求，我们可以用这个主体发送一个 401 响应:

```java
{
    "error": "auth-0001",
    "message": "Incorrect username and password",
    "detail": "Ensure that the username and password included in the request are correct"
}
```

**`error`字段不应与响应代码匹配。**相反，它应该是我们的应用程序特有的错误代码。一般来说，`error`字段没有约定，希望它是唯一的。

通常，该字段只包含字母数字和连接字符，如破折号或下划线。例如，`0001`、`auth-0001`和`incorrect-user-pass`就是错误代码的典型例子。

正文的`message`部分通常被认为是可以在用户界面上显示的。因此，如果我们支持[国际化](/web/20220703113315/https://www.baeldung.com/spring-boot-internationalization)，就应该翻译这个标题。因此，如果客户端发送一个带有对应于法语的`Accept-Language`报头的请求，那么`title`值应该被翻译成法语。

**`detail`部分旨在供客户的开发者使用，而非最终用户**，因此不需要翻译。

此外，我们还可以提供一个 URL——比如`help`字段——客户可以通过它找到更多信息:

```java
{
    "error": "auth-0001",
    "message": "Incorrect username and password",
    "detail": "Ensure that the username and password included in the request are correct",
    "help": "https://example.com/help/error/auth-0001"
}
```

有时，我们可能希望为一个请求报告多个错误。

在这种情况下，我们应该返回一个列表中的错误:

```java
{
    "errors": [
        {
            "error": "auth-0001",
            "message": "Incorrect username and password",
            "detail": "Ensure that the username and password included in the request are correct",
            "help": "https://example.com/help/error/auth-0001"
        },
        ...
    ]
}
```

当一个错误发生时，我们用一个包含一个元素的列表来响应。

注意，对于简单的应用程序来说，响应多个错误可能过于复杂。在许多情况下，响应第一个或最重要的错误就足够了。

### 3.4.标准化响应机构

虽然大多数 REST APIs 遵循相似的约定，但细节通常会有所不同，包括字段名和响应体中包含的信息。这些差异使得库和框架很难统一处理错误。

为了标准化 REST API 错误处理，**IETF 设计了 [RFC 7807](https://web.archive.org/web/20220703113315/https://tools.ietf.org/html/rfc7807) ，它创建了一个通用的错误处理模式**。

该模式由五部分组成:

1.  `type`–对错误进行分类的 URI 标识符
2.  `title`–关于错误的简单易懂的信息
3.  `status`–HTTP 响应代码(可选)
4.  `detail`–人类可读的错误解释
5.  `instance`–识别错误具体发生的 URI

我们可以转换我们的主体，而不是使用我们的自定义错误响应主体:

```java
{
    "type": "/errors/incorrect-user-pass",
    "title": "Incorrect username or password.",
    "status": 401,
    "detail": "Authentication failed due to incorrect username or password.",
    "instance": "/login/log/abc123"
}
```

注意，`type`字段对错误的类型进行分类，而`instance`分别以类似于类和对象的方式标识错误的具体发生。

通过使用 URIs，客户可以沿着这些路径找到更多关于错误的信息，就像使用 [HATEOAS 链接](/web/20220703113315/https://www.baeldung.com/spring-hateoas-tutorial)导航 REST API 一样。

遵守 RFC 7807 是可选的，但如果需要一致性，这是有利的。

## 4.例子

上述实践在一些最流行的 REST APIs 中很常见。虽然字段或格式的具体名称可能因站点而异，但一般模式几乎是通用的。

### 4.1.推特

让我们发送一个 GET 请求，但不提供所需的身份验证数据:

```java
curl -X GET https://api.twitter.com/1.1/statuses/update.json?include_entities=true
```

Twitter API 响应此主体的错误:

```java
{
    "errors": [
        {
            "code":215,
            "message":"Bad Authentication data."
        }
    ]
}
```

此响应包括一个包含单个错误的列表，以及错误代码和消息。在 Twitter 的例子中，不存在详细的消息，而是使用一个一般错误——而不是更具体的 401 错误——来表示身份验证失败。

有时一个更通用的状态代码更容易实现，正如我们将在下面的 Spring 例子中看到的。它允许开发人员捕捉异常组，并且不区分应该返回的状态代码。不过，在可能的情况下，应该使用最具体的状态代码**。**

### 4.2.脸谱网

与 Twitter 类似，[脸书的 Graph REST API](https://web.archive.org/web/20220703113315/https://developers.facebook.com/docs/graph-api) 也在其响应中包含详细信息。

让我们执行一个 POST 请求，用脸书图形 API 进行身份验证:

```java
curl -X GET https://graph.facebook.com/oauth/access_token?client_id=foo&client;_secret=bar&grant;_type=baz
```

我们收到以下错误:

```java
{
    "error": {
        "message": "Missing redirect_uri parameter.",
        "type": "OAuthException",
        "code": 191,
        "fbtrace_id": "AWswcVwbcqfgrSgjG80MtqJ"
    }
}
```

像 Twitter 一样，脸书也使用一般性错误——而不是更具体的 400 级错误——来表示失败。除了消息和数字代码，脸书还包括一个对错误进行分类的`type` 字段和一个充当[内部支持标识符](https://web.archive.org/web/20220703113315/https://developers.facebook.com/docs/graph-api/using-graph-api/error-handling)的跟踪 ID ( `fbtrace_id`)。

## 5.结论

在本文中，我们研究了 REST API 错误处理的一些最佳实践:

*   提供特定的状态代码
*   在回复正文中包含附加信息
*   以统一的方式处理异常

虽然错误处理的细节会因应用程序而异，但这些一般原则适用于几乎所有的 REST APIs，并且应该尽可能地遵守。

这不仅允许客户端以一致的方式处理错误，还简化了我们在实现 REST API 时创建的代码。

本文引用的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220703113315/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-simple)