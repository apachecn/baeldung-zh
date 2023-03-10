# Spring MVC 中的 HttpMediaTypeNotAcceptableException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-httpmediatypenotacceptable>

## 1。概述

在这篇简短的文章中，我们将看看*HttpMediaTypeNotAcceptableException*异常，并了解我们可能遇到它的情况。

## 2。问题

当用 Spring 实现 API 端点时，我们通常需要指定消费/生产的媒体类型(通过`consumes`和*生产*参数)。这缩小了 API 将为该特定操作返回给客户机的可能格式的范围。

HTTP 还有专用的`“Accept”`头——用于指定客户端识别和可以接受的媒体类型。简单地说，服务器将使用客户机请求的媒体类型之一发回一个资源表示。

然而，如果没有双方都可以使用的通用类型，Spring 将抛出*HttpMediaTypeNotAcceptableException*异常。

## 3。实际例子

让我们创建一个简单的例子来演示这个场景。

我们将使用一个 POST 端点——它只能与`“application/` json `“`一起工作，并返回 json 数据:

```java
@PostMapping(
  value = "/test", 
  consumes = MediaType.APPLICATION_JSON_VALUE, 
  produces = MediaType.APPLICATION_JSON_VALUE)
public Map<String, String> example() {
    return Collections.singletonMap("key", "value");
}
```

然后，让我们使用 CURL 发送一个不可识别内容类型的请求:

```java
curl -X POST --header "Accept: application/pdf" http://localhost:8080/test -v

> POST /test HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.51.0
> Accept: application/pdf
```

我们得到的回应是:

```java
< HTTP/1.1 406 
< Content-Length: 0
```

## 4。解决方案

只有一种方法可以解决这个问题——发送/接收一种受支持的类型。

我们所能做的就是提供一个更具描述性的消息(默认情况下 Spring 返回一个空的消息体),用一个定制的 *ExceptionHandler* 通知客户端所有可接受的媒体类型。

在我们的例子中，只有*“应用程序/JSON”*:

```java
@ResponseBody
@ExceptionHandler(HttpMediaTypeNotAcceptableException.class)
public String handleHttpMediaTypeNotAcceptableException() {
    return "acceptable MIME type:" + MediaType.APPLICATION_JSON_VALUE;
}
```

## 5。结论

在本教程中，我们已经考虑了当客户端请求的内容和服务器实际能够产生的内容不匹配时，Spring MVC 抛出的*HttpMediaTypeNotAcceptableException*异常。

和往常一样，文章中提到的代码片段可以在我们的 GitHub 库中找到。