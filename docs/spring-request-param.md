# Spring @RequestParam 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-request-param>

## 1。概述

在这个快速教程中，我们将探索 Spring 的`@RequestParam`注释及其属性。

**简单来说，我们可以使用`@RequestParam`从请求中提取查询参数、表单参数，甚至文件。**

## 延伸阅读:

## [Spring @RequestMapping 新快捷方式注释](/web/20220706134547/https://www.baeldung.com/spring-new-requestmapping-shortcuts)

In this article, we introduce different types of @RequestMapping shortcuts for quick web development using traditional Spring MVC framework.[Read more](/web/20220706134547/https://www.baeldung.com/spring-new-requestmapping-shortcuts) →

## [Spring @ Controller 和@RestController 注释](/web/20220706134547/https://www.baeldung.com/spring-controller-vs-restcontroller)

Learn about the differences between @Controller and @RestController annotations in Spring MVC.[Read more](/web/20220706134547/https://www.baeldung.com/spring-controller-vs-restcontroller) →

## 2。一个简单的映射

假设我们有一个端点`/api/foos`，它接受一个名为`id`的查询参数:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam String id) {
    return "ID: " + id;
}
```

在这个例子中，我们使用了`@RequestParam`来提取`id`查询参数。

一个简单的 GET 请求将调用`getFoos`:

```java
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

接下来，**我们来看看注释的属性:`name`、`value`、 `required`和`defaultValue`。**

## 3。指定请求参数名

在前面的示例中，变量名和参数名是相同的。

然而，有时我们希望这些是不同的。或者，如果我们不使用 Spring Boot，我们可能需要做特殊的编译时配置，否则参数名实际上不会出现在字节码中。

幸运的是，**我们可以使用`name`属性**来配置`@RequestParam`名称:

```java
@PostMapping("/api/foos")
@ResponseBody
public String addFoo(@RequestParam(name = "id") String fooId, @RequestParam String name) { 
    return "ID: " + fooId + " Name: " + name;
}
```

我们也可以做`@RequestParam(value = “id”)`或者只做`@RequestParam(“id”).`

## 4。可选请求参数

默认情况下，用`@RequestParam`标注的方法参数是必需的。

这意味着如果参数不在请求中，我们将得到一个错误:

```java
GET /api/foos HTTP/1.1
-----
400 Bad Request
Required String parameter 'id' is not present
```

**我们可以用`required `属性:**将`@RequestParam`配置为可选的

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam(required = false) String id) { 
    return "ID: " + id;
}
```

在这种情况下，两者都:

```java
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

和

```java
http://localhost:8080/spring-mvc-basics/api/foos
----
ID: null
```

将正确调用该方法。

**未指定参数时，方法参数绑定到`null`。**

### 4.1.使用 Java 8 `Optional`

或者，我们可以将参数包装在`[Optional](/web/20220706134547/https://www.baeldung.com/java-optional)`中:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam Optional<String> id){
    return "ID: " + id.orElseGet(() -> "not provided");
}
```

在这种情况下，**我们不需要指定`required`属性。**

如果没有提供请求参数，将使用默认值:

```java
http://localhost:8080/spring-mvc-basics/api/foos 
---- 
ID: not provided
```

## 5。请求参数的默认值

我们还可以通过使用`defaultValue`属性为`@RequestParam`设置一个默认值:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam(defaultValue = "test") String id) {
    return "ID: " + id;
}
```

**这与`required=false, `相似，用户不再需要提供参数**:

```java
http://localhost:8080/spring-mvc-basics/api/foos
----
ID: test
```

尽管如此，我们仍然可以提供:

```java
http://localhost:8080/spring-mvc-basics/api/foos?id=abc
----
ID: abc
```

注意，当我们设置`defaultValue `属性时，`required`确实被设置为`false`。

## 6。映射所有参数

**我们也可以有多个参数，而不用定义它们的名字**，或者只使用一个`Map`来计数:

```java
@PostMapping("/api/foos")
@ResponseBody
public String updateFoos(@RequestParam Map<String,String> allParams) {
    return "Parameters are " + allParams.entrySet();
}
```

这将反映出发送的任何参数:

```java
curl -X POST -F 'name=abc' -F 'id=123' http://localhost:8080/spring-mvc-basics/api/foos
-----
Parameters are {[name=abc], [id=123]}
```

## 7。映射多值参数

一个`@RequestParam`可以有多个值:

```java
@GetMapping("/api/foos")
@ResponseBody
public String getFoos(@RequestParam List<String> id) {
    return "IDs are " + id;
}
```

**和 Spring MVC 会映射一个逗号分隔的`id `参数**:

```java
http://localhost:8080/spring-mvc-basics/api/foos?id=1,2,3
----
IDs are [1,2,3]
```

**或单独的`id` 参数列表**:

```java
http://localhost:8080/spring-mvc-basics/api/foos?id=1&id;=2
----
IDs are [1,2]
```

## 8。结论

在这篇文章中，我们学习了如何使用`@RequestParam.`

示例的完整源代码可以在 GitHub 项目中找到。