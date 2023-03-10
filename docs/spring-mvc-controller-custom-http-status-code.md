# 从 Spring 控制器返回自定义状态代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-controller-custom-http-status-code>

## 1。概述

这篇短文将展示从 Spring MVC 控制器返回自定义 [HTTP 状态码](/web/20221208143917/https://www.baeldung.com/cs/http-status-codes)的几种方法。

为了更清楚地向客户机表达请求的结果，并使用 HTTP 协议的丰富语义，这通常是很重要的。例如，如果请求出错，为每种类型的可能问题发送特定的错误代码将允许客户端向用户显示适当的错误消息。

基本 Spring MVC 项目的设置超出了本文的范围，但是您可以在这里找到更多信息。

## 2。返回自定义状态代码

Spring 提供了一些从其`Controller`类返回自定义状态代码的主要方法:

*   使用`ResponseEntity`
*   在异常类上使用`@ResponseStatus`注释，以及
*   使用`@ControllerAdvice`和`@ExceptionHandler`注释。

这些选项并不相互排斥；远非如此，它们实际上可以相互补充。

本文将涵盖前两种方式(`ResponseEntity` 和`@ResponseStatus`)。如果你想了解更多关于使用`@ControllerAdvice`和`@ExceptionHandler`的知识，你可以在这里阅读[。](/web/20221208143917/https://www.baeldung.com/exception-handling-for-rest-with-spring#controlleradvice)

### 2.1。通过`ResponseEntity` 返回状态代码

在标准的 Spring MVC 控制器中，我们将定义一个简单的映射:

```java
@RequestMapping(value = "/controller", method = RequestMethod.GET)
@ResponseBody
public ResponseEntity sendViaResponseEntity() {
    return new ResponseEntity(HttpStatus.NOT_ACCEPTABLE);
}
```

在接收到对“`/controller`”的 GET 请求时，Spring 将返回一个带有 406 代码(不可接受)的响应。对于这个例子，我们任意选择了特定的响应代码。您可以返回任何 HTTP 状态代码(完整的列表可以在[这里](https://web.archive.org/web/20221208143917/https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)找到)。

### 2.2。通过异常返回状态代码

我们将向控制器添加第二个方法来演示如何使用`Exception`返回状态代码:

```java
@RequestMapping(value = "/exception", method = RequestMethod.GET)
@ResponseBody
public ResponseEntity sendViaException() {
    throw new ForbiddenException();
}
```

在收到对`/exception`的 GET 请求时，Spring 会抛出一个`ForbiddenException`。这是一个自定义异常，我们将在单独的类中定义它:

```java
@ResponseStatus(HttpStatus.FORBIDDEN)
public class ForbiddenException extends RuntimeException {}
```

在这个异常中不需要代码。所有的工作都由`@ResponseStatus`注释完成。

在这种情况下，当抛出异常时，抛出异常的控制器返回一个响应，响应代码为 403(禁止)。如有必要，您还可以在注释中添加一条消息，该消息将随响应一起返回。

在这种情况下，该类如下所示:

```java
@ResponseStatus(value = HttpStatus.FORBIDDEN, reason="To show an example of a custom message")
public class ForbiddenException extends RuntimeException {}
```

需要注意的是，虽然从技术上来说，让异常返回任何状态代码都是可能的，但是在大多数情况下，对错误代码(4XX 和 5XX)使用异常才有逻辑意义。

## 3。结论

本教程展示了如何从 Spring MVC 控制器返回自定义状态代码。

实现可以在[示例 GitHub 项目](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http)中找到。