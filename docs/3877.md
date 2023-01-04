# 圆形视图路径错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-circular-view-path-error>

## 1.介绍

在本教程中，我们将看看如何在 Spring MVC 应用程序中获得并解决循环视图路径错误。

## 2.属国

为了演示这一点，让我们创建一个简单的 Spring Boot web 项目。首先，我们需要在 Maven 项目文件中添加[Spring Boot web starter 依赖项](https://web.archive.org/web/20221129004617/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web/2.3.0.RELEASE/jar):

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3.重现问题

然后，让我们创建一个简单的 Spring Boot 应用程序，其中一个`Controller`解析为一个路径:

```
@Controller
public class CircularViewPathController {

    @GetMapping("/path")
    public String path() {
        return "path";
    }
}
```

返回值是将产生响应数据的视图名称。在我们的例子中，返回值是`path`，它与`path.html`模板相关联:

```
<html>
<head>
    <title>path.html</title>
</head>
<body>
    <p>path.html</p>
</body>
</html>
```

启动服务器后，我们可以通过向`[http://localhost:8080/path](https://web.archive.org/web/20221129004617/http://localhost:8080/path)`发出 GET 请求来重现错误。结果将是循环视图路径错误:

```
{"timestamp":"2020-05-22T11:47:42.173+0000","status":500,"error":"Internal Server Error",
"message":"Circular view path [path]: would dispatch back to the current handler URL [/path] 
again. Check your ViewResolver setup! (Hint: This may be the result of an unspecified view, 
due to default view name generation.)","path":"/path"} 
```

## 4.解决方法

默认情况下，Spring MVC 框架应用 [`InternalResourceView`](https://web.archive.org/web/20221129004617/https://docs.spring.io/spring/docs/3.0.x/javadoc-api/org/springframework/web/servlet/view/InternalResourceView.html) 类作为视图解析器。因此，**如果`@GetMapping`的值与视图**的值相同，请求将失败，并出现圆形视图路径错误。

一个可能的解决方案是重命名视图，并更改控制器方法中的返回值。

```
@Controller
public class CircularViewPathController {
  @GetMapping("/path")
  public String path() {
    return "path2";
  }
}
```

如果我们不想重命名视图并改变控制器方法中的返回值，那么另一个解决方案是为项目选择另一个视图处理器。

对于最常见的情况，我们可以选择[百里香 Java 模板引擎](https://web.archive.org/web/20221129004617/https://www.thymeleaf.org/)。让我们将[`spring-boot-starter-thymeleaf` 依赖项](https://web.archive.org/web/20221129004617/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-thymeleaf/2.3.0.RELEASE/jar)添加到项目中:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

重建项目后，我们可以再次运行它，并且请求是成功的。在这种情况下，百里香叶替换了`InternalResourceView`类。

## 5.结论

在本教程中，我们研究了循环视图路径错误，它发生的原因，以及如何解决这个问题。和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129004617/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3)