# 使用 Spring ResponseEntity 操作 HTTP 响应

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-response-entity>

## 1.介绍

使用 Spring，我们通常有许多方法来实现相同的目标，包括微调 HTTP 响应。

在这个简短的教程中，我们将看到如何使用`ResponseEntity`设置 HTTP 响应的主体、状态和头。

## 延伸阅读:

## [放心地获取和验证响应数据](/web/20220916105012/https://www.baeldung.com/rest-assured-response)

Have a look at how to use REST-assured to validate and extract the response from a REST endpoint[Read more](/web/20220916105012/https://www.baeldung.com/rest-assured-response) →

## [使用 Spring @ResponseStatus 设置 HTTP 状态码](/web/20220916105012/https://www.baeldung.com/spring-response-status)

Have a look at the @ResponseStatus annotation and how to use it to set the response status code.[Read more](/web/20220916105012/https://www.baeldung.com/spring-response-status) →

## 2.`ResponseEntity`

`ResponseEntity` **代表整个 HTTP 响应:状态码、头和主体**。因此，我们可以使用它来完全配置 HTTP 响应。

如果我们想使用它，我们必须从端点返回它；春天会处理好剩下的事情。

`ResponseEntity`是泛型。因此，我们可以使用任何类型作为响应体:

```
@GetMapping("/hello")
ResponseEntity<String> hello() {
    return new ResponseEntity<>("Hello World!", HttpStatus.OK);
}
```

因为我们以编程方式指定响应状态，所以我们可以为不同的场景返回不同的状态代码:

```
@GetMapping("/age")
ResponseEntity<String> age(
  @RequestParam("yearOfBirth") int yearOfBirth) {

    if (isInFuture(yearOfBirth)) {
        return new ResponseEntity<>(
          "Year of birth cannot be in the future", 
          HttpStatus.BAD_REQUEST);
    }

    return new ResponseEntity<>(
      "Your age is " + calculateAge(yearOfBirth), 
      HttpStatus.OK);
}
```

此外，我们可以设置 HTTP 头:

```
@GetMapping("/customHeader")
ResponseEntity<String> customHeader() {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Custom-Header", "foo");

    return new ResponseEntity<>(
      "Custom header set", headers, HttpStatus.OK);
}
```

此外，`ResponseEntity` **提供了两个嵌套的构建器接口** : `HeadersBuilder`及其子接口`BodyBuilder`。因此，我们可以通过`ResponseEntity`的静态方法访问它们的能力。

最简单的情况是带有正文和 HTTP 200 响应代码的响应:

```
@GetMapping("/hello")
ResponseEntity<String> hello() {
    return ResponseEntity.ok("Hello World!");
}
```

对于最流行的 [HTTP 状态码](/web/20220916105012/https://www.baeldung.com/cs/http-status-codes)，我们得到静态方法:

```
BodyBuilder accepted();
BodyBuilder badRequest();
BodyBuilder created(java.net.URI location);
HeadersBuilder<?> noContent();
HeadersBuilder<?> notFound();
BodyBuilder ok();
```

此外，我们可以使用`BodyBuilder status(HttpStatus status)`和`BodyBuilder status(int status)`方法来设置任何 HTTP 状态。

最后，用`ResponseEntity<T> BodyBuilder.body(T body)`我们可以设置 HTTP 响应体:

```
@GetMapping("/age")
ResponseEntity<String> age(@RequestParam("yearOfBirth") int yearOfBirth) {
    if (isInFuture(yearOfBirth)) {
        return ResponseEntity.badRequest()
            .body("Year of birth cannot be in the future");
    }

    return ResponseEntity.status(HttpStatus.OK)
        .body("Your age is " + calculateAge(yearOfBirth));
}
```

我们还可以设置自定义标题:

```
@GetMapping("/customHeader")
ResponseEntity<String> customHeader() {
    return ResponseEntity.ok()
        .header("Custom-Header", "foo")
        .body("Custom header set");
}
```

因为 `BodyBuilder.body()`返回一个`ResponseEntity`而不是`BodyBuilder,`，所以它应该是最后一个调用。

注意，使用`HeaderBuilder`我们不能设置响应体的任何属性。

当从控制器返回`ResponseEntity<T>` 对象时，我们可能会在处理请求时得到一个异常或错误，并希望**将错误相关信息返回给其他类型的用户，比如说 E** 。

Spring 3.2 带来了对全局 **`@ExceptionHandler `的支持，新的`@ControllerAdvice `注释**可以处理这些类型的场景。关于深入的细节，请参考我们现有的文章[这里](/web/20220916105012/https://www.baeldung.com/exception-handling-for-rest-with-spring)。

虽然`ResponseEntity`非常强大，但我们不应该过度使用它。在简单的情况下，有其他选项可以满足我们的需求，它们会产生更干净的代码。

## 3.可供选择的事物

### 3.1.`@ResponseBody`

在经典的 Spring MVC 应用程序中，端点通常返回呈现的 HTML 页面。有时候我们只需要返回实际的数据；例如，当我们在 AJAX 中使用端点时。

在这种情况下，我们可以用`@ResponseBody`标记请求处理程序方法，而 **Spring 将该方法的结果值视为 HTTP 响应体**本身。

要了解更多信息，[这篇文章是从](/web/20220916105012/https://www.baeldung.com/spring-request-response-body)开始的好地方。

### 3.2.`@ResponseStatus`

当一个端点成功返回时，Spring 提供一个 HTTP 200 (OK)响应。如果端点抛出异常，Spring 会寻找一个异常处理程序，告诉它使用哪种 HTTP 状态。

我们可以用@ResponseStatus 标记这些方法，因此，Spring **返回一个定制的 HTTP 状态**。

更多例子，请访问我们关于[自定义状态码](/web/20220916105012/https://www.baeldung.com/spring-response-status)的文章。

### 3.3.直接操纵响应

Spring 还让我们直接访问`javax.servlet.http.HttpServletResponse`对象；我们只需要将它声明为方法参数:

```
@GetMapping("/manual")
void manual(HttpServletResponse response) throws IOException {
    response.setHeader("Custom-Header", "foo");
    response.setStatus(200);
    response.getWriter().println("Hello World!");
}
```

由于 Spring 在底层实现之上提供了抽象和额外的功能，**我们不应该以这种方式操纵响应**。

## 4.结论

在本文中，我们讨论了在 Spring 中操纵 HTTP 响应的多种方法，并分析了它们的优缺点。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220916105012/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc)