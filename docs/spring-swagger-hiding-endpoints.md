# 对 Spring Boot 的 Swagger 文档隐藏端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-swagger-hiding-endpoints>

## 1.概观

在创建 Swagger 文档时，我们经常需要隐藏端点，以免暴露给最终用户。最常见的情况是端点尚未准备好。此外，我们可能有一些不想公开的私有端点。

在这篇短文中，我们将看看如何从 [Swagger API 文档](/web/20221129002555/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)中隐藏端点。为了实现这一点，我们将在控制器类中使用注释。

## 2.用`@ApiIgnore`隐藏端点

**`@ApiIgnore`注释允许我们隐藏一个端点**。让我们为控制器中的一个端点添加这个注释:

```java
@ApiIgnore
@ApiOperation(value = "This method is used to get the author name.")
@GetMapping("/getAuthor")
public String getAuthor() {
    return "Umang Budhwar";
}
```

## 3.用`@ApiOperation`隐藏端点

或者，我们可以使用 **`@ApiOperation`来隐藏单个端点**:

```java
@ApiOperation(value = "This method is used to get the current date.", hidden = true)
@GetMapping("/getDate")
public LocalDate getDate() {
    return LocalDate.now();
}
```

注意，**我们需要将`hidden`属性设置为`true`** 来让 Swagger 忽略这个端点。

## 4.用`@ApiIgnore`隐藏所有端点

尽管如此，有时我们需要**隐藏控制器类**的所有端点。我们可以通过用`@ApiIgnore`注释控制器类来实现这一点:

```java
@ApiIgnore
@RestController
public class RegularRestController {
    // regular code
}
```

需要注意的是**这将从文档中隐藏控制器本身。**

## 6.用`@Hidden`隐藏端点

如果我们使用的是 [OpenAPI v3](/web/20221129002555/https://www.baeldung.com/spring-rest-openapi-documentation) ，我们可以使用`@Hidden`注释隐藏一个端点:

```java
@Hidden
@GetMapping("/getAuthor")
public String getAuthor() {
    return "Umang Budhwar";
}
```

## 7.用`@Hidden`隐藏所有端点

类似地，我们可以用`@Hidden`注释控制器来隐藏所有的端点:

```java
@Hidden
@RestController
public class RegularRestController {
    // regular code
}
```

这也将从文档中隐藏控制器。

**注意:我们只能在使用 OpenAPI 时使用`@Hidden`。Swagger v3 对此注释的支持仍在进行中。**

## 8.结论

在本教程中，我们已经看到了如何从 Swagger 文档中隐藏端点。我们讨论了如何隐藏一个控制器类的单个端点和所有端点。

和往常一样，Swagger 示例的完整代码可以在 GitHub 上的[处获得，OpenAPI v3 示例也可以在 GitHub](https://web.archive.org/web/20221129002555/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-swagger) 上的[处获得。](https://web.archive.org/web/20221129002555/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-springdoc)