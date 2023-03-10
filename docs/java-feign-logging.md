# 假装日志配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-feign-logging>

## 1.概观

在本教程中，我们将描述如何让 Feign client 登录到我们的 Spring Boot 应用程序。此外，我们还将了解它的不同配置类型。为了复习一下 Feign client，[请查看我们的综合指南](/web/20221208143830/https://www.baeldung.com/spring-cloud-openfeign)。

## 2.假装客户

****是一个声明性的 web 服务**客户端**，它通过将注释处理成模板化的请求来工作。******使用一个虚拟客户端，我们可以摆脱样板代码来发出 HTTP API 请求。我们只需要放入一个带注释的接口。因此，实际的实现将在运行时创建。**

 **## 3.日志记录配置

假装客户端日志记录有助于我们更好地了解已经发出的请求。**要启用日志记录，我们需要将包含我们在`application.` `properties`文件`. `** 中的假想客户端的 `class` 或 `package` 的 Spring Boot 日志记录级别设置为`DEBUG`

让我们为一个类设置日志级别属性:

```java
logging.level.<packageName>.<className> = DEBUG
```

或者，如果我们有一个包，其中放有我们所有的虚拟客户端，我们可以为整个包添加它:

```java
logging.level.<packageName> = DEBUG
```

接下来，我们需要为 feign 客户端设置日志级别。请注意，上一步只是为了启用日志记录。

有四种日志记录级别可供选择:

*   `NONE:`无日志记录(默认)
*   `BASIC:`记录请求方法和 URL 以及响应状态代码和执行时间
*   `HEADERS:`记录基本信息以及请求和响应头
*   `FULL`:记录请求和响应的头、主体和元数据

我们可以通过 java 配置或在我们的属性文件中配置它们。

### 3.1.Java 配置

我们需要声明一个配置类，姑且称之为`FeignConfig`:

```java
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

之后，我们将配置类绑定到我们的假客户端类`FooClient`:

```java
@FeignClient(name = "foo-client", configuration = FeignConfig.class)
public interface FooClient {

    // methods for different requests
}
```

### 3.2.使用属性

第二种方法是将它设置在我们的 `application.properties.`中，让我们在这里引用我们的虚拟客户端的名称，在我们的例子中是`foo-client`:

```java
feign.client.config.foo-client.loggerLevel = full
```

或者，我们可以覆盖所有假扮客户端的默认配置级别:

```java
feign.client.config.default.loggerLevel = full
```

## 4.例子

对于这个例子，我们已经配置了一个客户端来读取[JSONPlaceHolder API](https://web.archive.org/web/20221208143830/https://jsonplaceholder.typicode.com/)。我们将在 feign 客户端的帮助下检索所有用户。

下面，我们将声明`UserClient`类:

```java
@FeignClient(name = "user-client", url="https://jsonplaceholder.typicode.com", configuration = FeignConfig.class)
public interface UserClient {

    @RequestMapping(value = "/users", method = RequestMethod.GET)
    String getUsers();
}
```

我们将使用我们在配置部分创建的相同的`FeignConfig`。请注意，日志记录级别仍然是`Logger.Level.FULL`。

让我们看看调用`/users`时日志是什么样子的:

```java
2021-05-31 17:21:54 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] ---> GET https://jsonplaceholder.typicode.com/users HTTP/1.1
2021-05-31 17:21:54 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] ---> END HTTP (0-byte body)
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] <--- HTTP/1.1 200 OK (902ms)
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] access-control-allow-credentials: true
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] cache-control: max-age=43200
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] content-type: application/json; charset=utf-8
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] date: Mon, 31 May 2021 14:21:54 GMT
                                                                                            // more headers
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getUsers] [
  {
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)",

    // more user details
  },

  // more users objects
]
2021-05-31 17:21:55 DEBUG 2992 - [thread-1] com.baeldung.UserClient : [UserClient#getPosts] <--- END HTTP (5645-byte body)
```

在日志的第一部分，我们可以看到`request` 被记录；带有 HTTP GET 方法的 URL 端点。在这种情况下，由于它是一个 GET `request`，我们没有请求体。

第二部分包含了`response`。它显示了 `response.`的`headers`和`body`

## 5.结论

在这个简短的教程中，我们已经了解了 Feign client 日志机制以及如何启用它。我们发现有多种方式可以根据我们的需求进行配置。

和往常一样，本教程中的例子可以在 GitHub 上找到。**