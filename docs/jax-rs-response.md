# 在 JAX-RS 设置响应主体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jax-rs-response>

****

## 1。概述

为了简化 REST web 服务及其客户端的 Java 开发，设计了一个标准的可移植的 API 实现，称为 Jersey。

Jersey 是一个用于开发 REST web 服务的开源框架，提供对`JAX-RS`API 的支持，并作为`JAX-RS`参考实现。

在本教程中，我们将看看如何用不同的媒体类型建立一个`Jersey`响应体。

## 2。Maven 依赖关系

首先，我们需要在`pom.xml`文件中包含以下依赖项:

```java
<dependency>
    <groupId>org.glassfish.jersey.bundles</groupId>
    <artifactId>jaxrs-ri</artifactId>
    <version>2.26</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-server</artifactId>
    <version>2.26</version>
</dependency>
```

最新版本的`JAX-RS`可以在 [jaxrs-ri](https://web.archive.org/web/20220629001801/https://search.maven.org/search?q=g:org.glassfish.jersey.bundles%20AND%20a:jaxrs-ri&core=gav) 找到，`Jersey`服务器可以在 [jersey-server](https://web.archive.org/web/20220629001801/https://search.maven.org/search?q=g:org.glassfish.jersey.core%20AND%20a:jersey-server&core=gav) 找到

## 3。于泽的回应

自然，使用`Jersey`构建响应有不同的方式，我们将在下面研究如何构建它们。

这里所有的例子都是 HTTP GET 请求，我们将使用`curl` 命令来测试资源。

### 3.1。Ok 文本响应

此处显示的端点是如何将纯文本作为 Jersey 响应返回的简单示例:

```java
@GET
@Path("/ok")
public Response getOkResponse() {

    String message = "This is a text response";

    return Response
      .status(Response.Status.OK)
      .entity(message)
      .build();
}
```

我们可以使用`curl`进行 HTTP GET 来验证响应:

```java
curl -XGET http://localhost:8080/jersey/response/ok
```

该端点将发送回如下响应:

```java
This is a text response
```

当未指定媒体类型时，**球衣将默认为文本/普通。**

### 3.2。错误响应

**错误也可以作为 Jersey 响应发送回**:

```java
@GET
@Path("/not_ok")
public Response getNOkTextResponse() {

    String message = "There was an internal server error";

    return Response
      .status(Response.Status.INTERNAL_SERVER_ERROR)
      .entity(message)
      .build();
}
```

为了验证响应，我们可以使用`curl`进行 HTTP GET 请求:

```java
curl -XGET http://localhost:8080/jersey/response/not_ok
```

错误消息将在响应中发回:

```java
There was an internal server error
```

### 3.3。纯文本响应

我们还可以返回**简单的纯文本响应**:

```java
@GET
@Path("/text_plain")
public Response getTextResponseTypeDefined() {

    String message = "This is a plain text response";

    return Response
      .status(Response.Status.OK)
      .entity(message)
      .type(MediaType.TEXT_PLAIN)
      .build();
}
```

同样，我们可以使用`curl`进行 HTTP GET 来验证响应:

```java
curl -XGET http://localhost:8080/jersey/response/text_plain
```

回应如下:

```java
This is a plain text response
```

同样的结果也可以通过`Produces `注释来实现，而不是在`Response`中使用`type()`方法:

```java
@GET
@Path("/text_plain_annotation")
@Produces({ MediaType.TEXT_PLAIN })
public Response getTextResponseTypeAnnotated() {

    String message = "This is a plain text response via annotation";

    return Response
      .status(Response.Status.OK)
      .entity(message)
      .build();
}
```

我们可以使用`curl`进行响应验证:

```java
curl -XGET http://localhost:8080/jersey/response/text_plain_annotation
```

以下是回应:

```java
This is a plain text response via annotation
```

### 3.4。使用 POJO 的 JSON 响应

一个简单的**普通旧 Java 对象(POJO)也可以用来构建一个 Jersey 响应**。

我们有一个非常简单的 POJO，如下所示，我们将使用它来构建一个响应:

```java
public class Person {
    String name;
    String address;

    // standard constructor
    // standard getters and setters
}
```

现在可以使用`Person` POJO 作为响应体返回 JSON:

```java
@GET
@Path("/pojo")
public Response getPojoResponse() {

    Person person = new Person("Abhinayak", "Nepal");

    return Response
      .status(Response.Status.OK)
      .entity(person)
      .build();
}
```

可以通过下面的`curl`命令来验证这个 GET 端点的工作情况:

```java
curl -XGET http://localhost:8080/jersey/response/pojo
```

POJO 这个人将被转换成一个 JSON，并作为响应发送回去:

```java
{"address":"Nepal","name":"Abhinayak"}
```

### 3.5。使用简单字符串的 JSON 响应

我们可以使用**预先格式化的字符串来创建一个响应**，这很简单。

以下端点是如何在 Jersey 响应中将表示为`String`的 JSON 作为 JSON 发回的示例:

```java
@GET
@Path("/json")
public Response getJsonResponse() {

    String message = "{\"hello\": \"This is a JSON response\"}";

    return Response
      .status(Response.Status.OK)
      .entity(message)
      .type(MediaType.APPLICATION_JSON)
      .build();
}
```

这可以通过使用`curl`执行 HTTP GET 来验证响应:

```java
curl -XGET http://localhost:8080/jersey/response/json
```

调用该资源将返回一个 JSON:

```java
{"hello":"This is a JSON response"}
```

同样的模式也适用于其他常见的媒体类型，如 XML 或 HTML 。我们只需要使用`MediaType.TEXT_XML`或`MediaType.TEXT_HTML`通知 Jersey 这是一个 XML 或 HTML，Jersey 会处理剩下的事情。

## 4。结论

在这篇简短的文章中，我们构建了新泽西(JAX-RS)对各种媒体类型的回应。

文章中提到的所有代码片段都可以在 GitHub 的[中找到。](https://web.archive.org/web/20220629001801/https://github.com/eugenp/tutorials/tree/master/jersey)

****