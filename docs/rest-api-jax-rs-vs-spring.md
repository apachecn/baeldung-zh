# 休息 API: JAX-RS vs Spring

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-jax-rs-vs-spring>

## 1.概观

在本教程中，我们将看到 [JAX-RS](/web/20221208143832/https://www.baeldung.com/jax-rs-spec-and-implementations) 和 [Spring MVC](/web/20221208143832/https://www.baeldung.com/rest-with-spring-series) 在 REST API 开发上的区别。

## 2.雅加达 RESTful Web 服务

要成为 [JAVA EE](/web/20221208143832/https://www.baeldung.com/java-enterprise-evolution) 世界的一部分，一个特性必须有一个规范、一个兼容的实现和一个 [TCK](https://web.archive.org/web/20221208143832/https://en.wikipedia.org/wiki/Technology_Compatibility_Kit) 。相应的， **JAX-RS** **是一套建筑休息服务**的规范。**它最著名的参考实现是 [RESTEasy](/web/20221208143832/https://www.baeldung.com/resteasy-tutorial) 和 [Jersey](/web/20221208143832/https://www.baeldung.com/jersey-rest-api-with-spring)** 。

现在，让我们通过实现一个简单的控制器来熟悉一下 Jersey:

```java
@Path("/hello")
public class HelloController {

    @GET
    @Path("/{name}")
    @Produces(MediaType.TEXT_PLAIN)
    public Response hello(@PathParam("name") String name) {
        return Response.ok("Hello, " + name).build();
    }

} 
```

如上所述，端点返回一个简单的“文本/普通”响应，如注释`@Produces`所述。特别是，我们公开了一个`hello` HTTP 资源，它接受一个名为`name`的参数和两个`@Path`注释。我们还需要使用注释`@GET`指定这是一个`GET`请求。

## 3.用 Spring MVC 休息

Spring MVC 是 Spring 框架的一个模块，用于创建 web 应用程序。它为 Spring 框架增加了 REST 功能。

让我们使用 Spring MVC 来制作与上面相同的`GET`请求示例:

```java
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping(value = "/{name}", produces = MediaType.TEXT_PLAIN_VALUE)
    public ResponseEntity<?> hello(@PathVariable String name) {
        return new ResponseEntity<>("Hello, " + name, HttpStatus.OK);
    }

}
```

查看代码，`@RequestMapping`声明我们正在处理一个`hello` HTTP 资源。特别是，通过`@GetMapping `注释，我们指定它是一个`GET`请求。它接受一个名为`name`的参数，并返回一个“文本/普通”响应。

## 4.差异

JAX-RS 致力于提供一组 Java 注释，并将它们应用于普通的 Java 对象。事实上，这些注释帮助我们抽象出客户端-服务器通信的底层细节。为了简化实现，它提供了注释来处理 HTTP 请求和响应，并将它们绑定在代码中。 **JAX-RS 只是一个规范，它需要一个兼容的实现来使用**。

另一方面， **Spring MVC** **是一个完整的框架，具有 REST 能力**。像 JAX-RS 一样，它也为我们提供了有用的注释来从底层细节中进行抽象。它的主要优势是作为 [Spring 框架](/web/20221208143832/https://www.baeldung.com/spring-intro)生态系统的一部分。因此，它允许我们像任何其他 Spring 模块一样使用依赖注入。此外，它很容易与其他组件集成，如 [Spring AOP](/web/20221208143832/https://www.baeldung.com/spring-aop) 、 [Spring Data REST](/web/20221208143832/https://www.baeldung.com/spring-data-rest-intro) 和 [Spring Security](/web/20221208143832/https://www.baeldung.com/security-spring) 。

## 5.结论

在这篇简短的文章中，我们看到了 JAX-RS 和 Spring MVC 之间的主要区别。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-jersey)