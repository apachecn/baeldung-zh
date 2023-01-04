# Swagger @Api 描述已被否决

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-swagger-api-description-deprecated>

## 1.概观

描述 RESTful API 在文档中起着重要的作用。一个用于记录 REST APIs 的常用工具是 [Swagger 2](/web/20220626211616/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api) 。然而，一个用于添加描述的有用属性已经被弃用。在本教程中，我们将使用 Swagger 2 和 [OpenAPI 3](/web/20220626211616/https://www.baeldung.com/spring-rest-openapi-documentation) 找到不赞成使用的`description`属性的解决方案，并且我们将展示如何使用它们来描述一个 [Spring Boot](/web/20220626211616/https://www.baeldung.com/spring-boot-start) REST API 应用程序。

## 2.API 描述

默认情况下，Swagger 为 REST API 类名生成一个空描述。因此，我们需要指定一个合适的注释来描述 REST API。我们既可以使用带有`@Api`注释的 Swagger 2，也可以在 OpenAPI 3 中使用`@Tag`注释。

## 3.霸气 2

要将 Swagger 2 用于 Spring Boot REST API，我们可以使用 [Springfox](https://web.archive.org/web/20220626211616/https://github.com/springfox/springfox) 库。我们需要在`pom.xml`文件中添加 [`springfox-boot-starter`](https://web.archive.org/web/20220626211616/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.springfox%22%20a%3A%22springfox-boot-starter%22) 依赖关系:

```java
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

Springfox 库提供了`@Api`注释来将类配置为 Swagger 资源。以前，`@Api`注释提供了一个`description`属性来定制 API 文档:

```java
@Api(value = "", description = "")
```

然而，如前所述，**的`description`属性已被弃用**。幸运的是，还有一个选择。我们可以**使用`tags`属性**:

```java
@Api(value = "", tags = {"tag_name"})
```

在 Swagger 1.5 中，我们将使用`@SwaggerDefinition`注释来定义`tag`。但是，在 Swagger 2 中已经不支持了。因此，在 Swagger 2 中，我们将`Docket` bean 中的`tags`和`descriptions`定义为:

```java
@Configuration
public class SwaggerConfiguration {

    public static final String BOOK_TAG = "book service";

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
          .select()
          .apis(RequestHandlerSelectors.any())
          .paths(PathSelectors.any())
          .build()
          .tags(new Tag(BOOK_TAG, "the book API with description api tag"));
    }

} 
```

这里，我们使用`Docket` bean 中的`Tag`类来创建我们的`tag`。这样，我们可以引用控制器中的`tag`:

```java
@RestController
@RequestMapping("/api/book")
@Api(tags = {SwaggerConfiguration.BOOK_TAG})
public class BookController {

    @GetMapping("/")
    public List<String> getBooks() {
        return Arrays.asList("book1", "book2");
    }
}
```

## 4.OpenAPI 3

OpenAPI 3 是 OpenAPI `Specification.`的最新版本，它是 OpenAPI 2 (Swagger 2)的继任者。为了使用 OpenAPI 3 描述 API，我们可以使用`@Tag`注释。此外，`@Tag`注释提供了一个`description`和外部链接。让我们定义一下`BookController`类:

```java
@RestController
@RequestMapping("/api/book")
@Tag(name = "book service", description = "the book API with description tag annotation")
public class BookController {

    @GetMapping("/")
    public List<String> getBooks() {
        return Arrays.asList("book1", "book2");
    }
}
```

## 5.结论

在这篇简短的文章中，我们描述了如何在 Spring Boot 应用程序的 REST API 中添加描述。我们研究了如何使用 Swagger 2 和 OpenAPI 3 实现这一点。对于 Swagger 部分，代码可以从 GitHub 上的[处获得。要查看 OpenAPI 3 样本代码，请查看 GitHub](https://web.archive.org/web/20220626211616/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-swagger) 上的模块[。](https://web.archive.org/web/20220626211616/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-springdoc)