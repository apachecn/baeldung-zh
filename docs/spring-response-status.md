# 使用 Spring @ResponseStatus 设置 HTTP 状态代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-response-status>

## 1.介绍

在 Spring MVC 中，我们有很多方法来**设置 HTTP 响应的状态码**。

在这个简短的教程中，我们将看到最直接的方法:使用`@ResponseStatus`注释。

## 2.论控制器方法

当一个端点成功返回时，Spring 提供一个 HTTP 200 (OK)响应。

如果我们想要指定一个控制器方法的**响应状态，我们可以用`@ResponseStatus.` 标记该方法，它有两个可互换的参数用于期望的响应状态:`code,`和`value.`例如，我们可以[指示服务器拒绝煮咖啡，因为它是一个茶壶](https://web.archive.org/web/20220625233643/https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/418):**

```java
@ResponseStatus(HttpStatus.I_AM_A_TEAPOT)
void teaPot() {}
```

当我们想要发出一个错误信号时，我们可以通过`reason`参数提供一个错误消息:

```java
@ResponseStatus(HttpStatus.BAD_REQUEST, reason = "Some parameters are invalid")
void onIllegalArgumentException(IllegalArgumentException exception) {}
```

注意，当我们设置`reason`时，Spring 调用`HttpServletResponse.sendError()`。因此，它会向客户端发送一个 **HTML 错误页面，这使得它不适合 REST 端点**。

还要注意，Spring 只使用`@ResponseStatus`，当**标记的方法成功完成**(没有抛出`Exception`)。

## 3.使用错误处理程序

我们有三种方法使用`@ResponseStatus`将`Exception`转换成 HTTP 响应状态:

*   使用`@ExceptionHandler`
*   使用`@ControllerAdvice`
*   标记`Exception`类

为了使用前两种解决方案，我们必须定义一个错误处理程序方法。你可以在[这篇文章](/web/20220625233643/https://www.baeldung.com/exception-handling-for-rest-with-spring)中读到更多关于这个话题的内容。

我们可以使用这些错误处理方法**来使用`@ResponseStatus`，就像我们在上一节中使用常规 MVC 方法**一样。

当我们不需要动态错误响应时，最直接的解决方案是第三种:用`@ResponseStatus:`标记异常类

```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST)
class CustomException extends RuntimeException {}
```

当 Spring 捕捉到这个`Exception`时，它使用我们在`@ResponseStatus`中提供的设置。

注意，当我们用`@ResponseStatus`标记一个`Exception`类时，不管我们是否设置了`reason`，Spring 总是调用`HttpServletResponse.sendError()` `,`。

还要注意，Spring 对子类使用相同的配置，除非我们也用`@ResponseStatus`标记它们。

## 4.结论

在本文中，我们看到了如何使用`@ResponseStatus`在不同的场景中设置 HTTP 响应代码，包括错误处理。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625233643/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5)