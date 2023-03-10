# 用 curl 测试 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/curl-rest>

## 1.概观

本教程给出了使用`curl.`测试 REST API 的简要概述

**`curl`是一个用于传输数据的命令行工具，它支持大约 22 种协议，包括 HTTP。**这种组合使它成为测试 REST 服务的一个非常好的专用工具。

## 延伸阅读:

## [用邮递员集合测试 Web APIs】](/web/20220626112837/https://www.baeldung.com/postman-testing-collections)

Learn how to create a Postman Collection that can test a REST API[Read more](/web/20220626112837/https://www.baeldung.com/postman-testing-collections) →

## [放心指南](/web/20220626112837/https://www.baeldung.com/rest-assured-tutorial)

Explore the basics of REST-assured - a library that simplifies the testing and validation of REST APIs.[Read more](/web/20220626112837/https://www.baeldung.com/rest-assured-tutorial) →

## 2.命令行选项

**`curl`支持超过 200 个命令行选项**。我们可以在命令中的 URL 后面加上零个或多个。

在我们将它用于我们的目的之前，让我们先来看看两个能让我们的生活更轻松的方法。

### 2.1.冗长的

当我们测试时，最好将详细模式设置为:

```java
curl -v http://www.example.com/
```

因此，这些命令提供了有用的信息，如解析的 IP 地址、我们试图连接的端口和报头。

### 2.2.输出

默认情况下，`curl`将响应体输出到标准输出。此外，我们可以提供输出选项来保存到文件:

```java
curl -o out.json http://www.example.com/index.html
```

当响应大小很大时，这尤其有用。

## 3.带`curl`的 HTTP 方法

每个 HTTP 请求都包含一个方法。最常用的方法是 GET、POST、PUT 和 DELETE。

### 3.1.得到

这是使用 `curl`进行 HTTP 调用时的默认方法。事实上，前面展示的例子都是简单的 GET 调用。

当在端口 8082 运行一个服务的本地实例时，我们将使用类似下面的命令进行 GET 调用:

```java
curl -v http://localhost:8082/spring-rest/foos/9
```

因为我们启用了详细模式，所以除了响应体之外，我们还获得了更多的信息:

```java
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8082 (#0)
> GET /spring-rest/foos/9 HTTP/1.1
> Host: localhost:8082
> User-Agent: curl/7.60.0
> Accept: */*
>
< HTTP/1.1 200
< X-Application-Context: application:8082
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Sun, 15 Jul 2018 11:55:26 GMT
<
{
  "id" : 9,
  "name" : "TuwJ"
}* Connection #0 to host localhost left intact
```

### 3.2.邮政

我们使用这个方法向接收服务发送数据，这意味着我们使用 data 选项。

最简单的方法是在命令中嵌入数据:

```java
curl -d 'id=9&name;=baeldung' http://localhost:8082/spring-rest/foos/new
```

或者，我们可以将包含请求体的文件传递给数据选项，如下所示:

```java
curl -d @request.json -H "Content-Type: application/json" 
  http://localhost:8082/spring-rest/foos/new
```

通过使用上面的命令，我们可能会遇到如下所示的错误消息:

```java
{
  "timestamp" : "15-07-2018 05:57",
  "status" : 415,
  "error" : "Unsupported Media Type",
  "exception" : "org.springframework.web.HttpMediaTypeNotSupportedException",
  "message" : "Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported",
  "path" : "/spring-rest/foos/new"
}
```

这是因为`curl`向所有 POST 请求添加了以下默认标题:

```java
Content-Type: application/x-www-form-urlencoded
```

这也是浏览器在普通帖子中使用的内容。在我们的使用中，我们通常希望根据我们的需要定制标题。

例如，如果我们的服务需要 JSON 内容类型，那么我们可以使用-H 选项来修改我们最初的 POST 请求:

```java
curl -d '{"id":9,"name":"baeldung"}' -H 'Content-Type: application/json' 
  http://localhost:8082/spring-rest/foos/new
```

Windows 命令提示符不支持单引号，就像类 Unix shell 一样。

因此，我们需要用双引号替换单引号，尽管我们在必要时会尽量避免使用它们:

```java
curl -d "{\"id\":9,\"name\":\"baeldung\"}" -H "Content-Type: application/json" 
  http://localhost:8082/spring-rest/foos/new
```

此外，当我们想要发送更多的数据时，使用数据文件通常是个好主意。

### 3.3.放

这种方法非常类似于 POST，但是当我们想要发送现有资源的新版本时，我们使用它。为了做到这一点，我们使用-X 选项。

在没有提到请求方法类型的情况下，`curl`默认使用 GET 因此，我们在 PUT 的情况下明确提到了方法类型:

```java
curl -d @request.json -H 'Content-Type: application/json' 
  -X PUT http://localhost:8082/spring-rest/foos/9
```

### 3.4.删除

同样，我们通过使用-X 选项来指定我们想要使用 DELETE:

```java
curl -X DELETE http://localhost:8082/spring-rest/foos/9
```

## 4.自定义标题

我们可以替换默认标题或添加我们自己的标题。

例如，要更改主机头，我们这样做:

```java
curl -H "Host: com.baeldung" http://example.com/
```

为了关闭用户代理头，我们输入一个空值:

```java
curl -H "User-Agent:" http://example.com/
```

测试时最常见的场景是更改内容类型和接受头。我们只需在每个头前加上-H 选项:

```java
curl -d @request.json -H "Content-Type: application/json" 
  -H "Accept: application/json" http://localhost:8082/spring-rest/foos/new
```

## 5.证明

需要认证的[服务](/web/20220626112837/https://www.baeldung.com/spring-security-basic-authentication)会发回一个 401-未授权的 HTTP 响应代码，以及一个相关的 WWW-Authenticate 头。

对于基本认证，我们可以使用用户选项简单地将用户名和密码组合嵌入到我们的请求中:

```java
curl --user baeldung:secretPassword http://example.com/
```

然而，如果我们想要[使用 OAuth2 进行认证](/web/20220626112837/https://www.baeldung.com/rest-api-spring-oauth2-angularjs)，我们首先需要从我们的授权服务中获取`access_token`。

服务响应将包含`access_token:`

```java
{
  "access_token": "b1094abc0-54a4-3eab-7213-877142c33fh3",
  "token_type": "bearer",
  "refresh_token": "253begef-868c-5d48-92e8-448c2ec4bd91",
  "expires_in": 31234
}
```

现在我们可以在授权头中使用令牌了:

```java
curl -H "Authorization: Bearer b1094abc0-54a4-3eab-7213-877142c33fh3" http://example.com/
```

## 6.结论

在本文中，我们演示了如何使用 `curl`的最小功能来测试我们的 REST 服务。虽然它能做的比这里讨论的更多，但是对于我们的目的来说，这已经足够了。

随意在命令行上键入`curl` -h 来检查所有可用的选项。用于演示的 REST 服务可以在 GitHub 的[这里获得。](https://web.archive.org/web/20220626112837/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-simple)