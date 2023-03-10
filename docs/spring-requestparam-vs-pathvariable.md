# spring @ request param vs @ path variable 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-requestparam-vs-pathvariable>

## 1。概述

在这个快速教程中，我们将探索 Spring 的 [`@RequestParam`](/web/20220628053909/https://www.baeldung.com/spring-request-param) 和`@PathVariable`注释之间的区别。

`@RequestParam`和`@PathVariable`都可以用来从请求 URI 中提取值，但是它们有一点不同。

## 延伸阅读:

## [在 Spring 中验证请求参数和路径变量](/web/20220628053909/https://www.baeldung.com/spring-validate-requestparam-pathvariable)

Learn how to validate request parameters and path variables with Spring MVC[Read more](/web/20220628053909/https://www.baeldung.com/spring-validate-requestparam-pathvariable) →

## [Spring 的 RequestBody 和 ResponseBody 注释](/web/20220628053909/https://www.baeldung.com/spring-request-response-body)

Learn about the Spring @RequestBody and @ResponseBody annotations.[Read more](/web/20220628053909/https://www.baeldung.com/spring-request-response-body) →

## [使用 Spring @ResponseStatus 设置 HTTP 状态码](/web/20220628053909/https://www.baeldung.com/spring-response-status)

Have a look at the @ResponseStatus annotation and how to use it to set the response status code.[Read more](/web/20220628053909/https://www.baeldung.com/spring-response-status) →

## 2。查询参数 vs URI 路径

**s 从查询字符串中提取值，`@PathVariable` s 从 URI 路径中提取值**:

```java
@GetMapping("/foos/{id}")
@ResponseBody
public String getFooById(@PathVariable String id) {
    return "ID: " + id;
}
```

然后我们可以根据路径进行映射:

```java
http://localhost:8080/spring-mvc-basics/foos/abc
----
ID: abc
```

对于`@RequestParam`，它将是:

```java
@GetMapping("/foos")
@ResponseBody
public String getFooByIdUsingQueryParam(@RequestParam String id) {
    return "ID: " + id;
}
```

这将给我们同样的回答，只是一个不同的 URI:

```java
http://localhost:8080/spring-mvc-basics/foos?id=abc
----
ID: abc
```

## 3。编码值与精确值

因为`@PathVariable`是从 URI 路径中提取值，所以它没有被编码。另一方面，`@RequestParam`被编码。

使用前面的例子，`ab+c`将原样返回:

```java
http://localhost:8080/spring-mvc-basics/foos/ab+c
----
ID: ab+c
```

**但是对于`a @RequestParam`请求，参数是 URL 解码的**:

```java
http://localhost:8080/spring-mvc-basics/foos?id=ab+c
----
ID: ab c
```

## 4。可选值

**`@RequestParam`和`@PathVariable`都可以选择。**

从 Spring 4.3.3 开始，我们可以通过使用`required`属性使`@PathVariable`可选:

```java
@GetMapping({"/myfoos/optional", "/myfoos/optional/{id}"})
@ResponseBody
public String getFooByOptionalId(@PathVariable(required = false) String id){
    return "ID: " + id;
}
```

那么我们可以选择:

```java
http://localhost:8080/spring-mvc-basics/myfoos/optional/abc
----
ID: abc 
```

或者:

```java
http://localhost:8080/spring-mvc-basics/myfoos/optional
----
ID: null
```

对于`@RequestParam`，我们也可以使用 [`required`](/web/20220628053909/https://www.baeldung.com/spring-request-param#making-an-optional-request-parameter) 属性。

注意，**在使`@PathVariable`可选时我们应该小心，以避免路径冲突。**

## 5。结论

在本文中，我们了解了`@RequestParam`和`@PathVariable`之间的区别。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628053909/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5)