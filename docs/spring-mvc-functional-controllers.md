# Spring MVC 中的功能控制器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-functional-controllers>

## 1.介绍

Spring 5 引入了 [WebFlux](/web/20221129021438/https://www.baeldung.com/spring-5-functional-web) ，这是一个新的框架，让我们使用[反应式](/web/20221129021438/https://www.baeldung.com/spring-reactor)编程模型来构建 web 应用。

在本教程中，我们将看到如何将这种编程模型应用于 Spring MVC 中的功能控制器。

## 2.Maven 设置

我们将使用 [Spring Boot](/web/20221129021438/https://www.baeldung.com/spring-boot) 来演示新的 API。

这个框架支持熟悉的基于注释的定义控制器的方法。但是它也增加了一种新的特定于领域的语言，提供了一种定义控制器的函数式方法。

从 Spring 5.2 开始，函数式方法也将在 [Spring Web MVC](/web/20221129021438/https://www.baeldung.com/spring-mvc-tutorial) 框架中**可用。**和`WebFlux`模块一样，`RouterFunctions`和`RouterFunction`是这个 API 的主要抽象。

所以让我们从导入[的`spring-boot-starter-web`依赖项](https://web.archive.org/web/20221129021438/https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web)开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3.`RouterFunction` vs @ `Controller`

**在功能领域，web 服务被称为路由**，传统的概念`@Controller`和`@RequestMapping`被一个`RouterFunction`所取代。

为了创建我们的第一个服务，让我们以一个基于注释的服务为例，看看如何将它转换成功能等同的服务。

我们将使用返回产品目录中所有产品的服务示例:

```java
@RestController
public class ProductController {

    @RequestMapping("/product")
    public List<Product> productListing() {
        return ps.findAll();
    }
}
```

现在，让我们来看看它的等效功能:

```java
@Bean
public RouterFunction<ServerResponse> productListing(ProductService ps) {
    return route().GET("/product", req -> ok().body(ps.findAll()))
      .build();
}
```

### 3.1.路线定义

我们应该注意，在函数式方法中，`productListing()`方法返回一个`RouterFunction`而不是响应体。**这是路线的定义，而不是请求的执行。**

`RouterFunction`包括路径、请求头、一个处理函数，它将用于生成响应体和响应头。它可以包含一个或一组 web 服务。

当我们查看嵌套路由时，我们将更详细地讨论 web 服务组。

在这个例子中，**我们使用了`RouterFunctions` 中的静态路由()方法来创建一个`RouterFunction`。**路由的所有请求和响应属性都可以使用这种方法提供。

### 3.2.请求谓词

在我们的示例中，我们使用 route()上的 GET()方法来指定这是一个`GET`请求，并提供一个路径作为`String.`

**当我们想要指定请求的更多细节时，我们也可以使用`RequestPredicate` 。**

例如，上例中的路径也可以使用`RequestPredicate`指定为:

```java
RequestPredicates.path("/product")
```

在这里，**我们已经使用静态实用程序`RequestPredicates`创建了一个对象`RequestPredicate`。**

### 3.3.反应

类似地， **`ServerResponse`包含用于创建响应对象**的静态实用方法。

在我们的例子中，我们使用`ok()`将 HTTP Status 200 添加到响应头中，然后使用`body()` 指定响应体。

此外，`ServerResponse`支持使用`EntityResponse.` 从定制数据类型构建响应，我们也可以通过`RenderingResponse.`使用 Spring MVC 的`ModelAndView`

### 3.4.注册路线

接下来，让我们使用`@Bean`注释注册这个路由，将其添加到应用程序上下文中:

```java
@SpringBootApplication
public class SpringBootMvcFnApplication {

    @Bean
    RouterFunction<ServerResponse> productListing(ProductController pc, ProductService ps) {
        return pc.productListing(ps);
    }
}
```

现在，让我们实现一些我们在使用功能方法开发 web 服务时遇到的常见用例。

## 4.嵌套路由

在一个应用程序中有一堆 web 服务，并根据功能或实体将它们分成逻辑组，这是很常见的。例如，我们可能希望所有与产品相关的服务都以`/product`开始。

让我们在现有路径`/product`上添加另一个路径，按名称查找产品:

```java
public RouterFunction<ServerResponse> productSearch(ProductService ps) {
    return route().nest(RequestPredicates.path("/product"), builder -> {
        builder.GET("/name/{name}", req -> ok().body(ps.findByName(req.pathVariable("name"))));
    }).build();
}
```

在传统的方法中，我们会通过传递一个路径到`@Controller`来实现这一点。然而，**对 web 服务进行分组的等效功能是 route()上的 nest()方法。**

这里，我们首先提供我们想要对新路由进行分组的路径，即`/product`。接下来，我们使用 builder 对象添加路线，与前面的示例类似。

`nest()`方法负责将添加到 builder 对象的路线与主`RouterFunction`合并。

## 5.错误处理

另一个常见的用例是拥有自定义的错误处理机制。我们可以使用`route()`上的 **`onError()`方法来定义一个定制的异常处理程序**。

这相当于在基于注释的方法中使用`@ExceptionHandler`。但是它更加灵活，因为它可以用于为每组路由定义单独的异常处理程序。

让我们将一个异常处理程序添加到我们之前创建的产品搜索路径中，以处理在没有找到产品时抛出的自定义异常:

```java
public RouterFunction<ServerResponse> productSearch(ProductService ps) {
    return route()...
      .onError(ProductService.ItemNotFoundException.class,
         (e, req) -> EntityResponse.fromObject(new Error(e.getMessage()))
           .status(HttpStatus.NOT_FOUND)
           .build())
      .build();
}
```

`onError()`方法接受`Exception`类对象，并期望从函数实现中得到一个`ServerResponse`。

我们使用了 ServerResponse 的一个子类型`EntityResponse`来从自定义数据类型`Error`构建一个响应对象。然后我们添加状态并使用返回一个`ServerResponse`对象的`EntityResponse.build()`。

## 6.过滤

实现身份验证以及管理横切关注点(如日志和审计)的一种常见方式是使用过滤器。过滤器用于决定是继续还是中止请求的处理。

让我们举一个例子，我们想要一个新的路线，将产品添加到目录中:

```java
public RouterFunction<ServerResponse> adminFunctions(ProductService ps) {
    return route().POST("/product", req -> ok().body(ps.save(req.body(Product.class))))
      .onError(IllegalArgumentException.class, 
         (e, req) -> EntityResponse.fromObject(new Error(e.getMessage()))
           .status(HttpStatus.BAD_REQUEST)
           .build())
        .build();
}
```

由于这是一个管理功能，我们还想验证调用服务的用户。

**我们可以通过在 route():** 上添加一个`filter()`方法来实现

```java
public RouterFunction<ServerResponse> adminFunctions(ProductService ps) {
   return route().POST("/product", req -> ok().body(ps.save(req.body(Product.class))))
     .filter((req, next) -> authenticate(req) ? next.handle(req) : 
       status(HttpStatus.UNAUTHORIZED).build())
     ....;
}
```

这里，`filter()`方法提供了请求和下一个处理程序，我们用它来做一个简单的认证，如果成功就保存产品，如果失败就向客户端返回一个`UNAUTHORIZED`错误。

## 7.贯穿各领域的问题

有时，我们可能希望在请求之前、之后或前后执行一些操作。例如，我们可能想要记录传入请求和传出响应的一些属性。

让我们在应用程序每次为传入请求找到匹配时记录一条语句。**我们将在`route()` :** 上使用`before()`方法来实现这一点

```java
@Bean
RouterFunction<ServerResponse> allApplicationRoutes(ProductController pc, ProductService ps) {
    return route()...
      .before(req -> {
          LOG.info("Found a route which matches " + req.uri()
            .getPath());
          return req;
      })
      .build();
}
```

类似地，**在使用`route()`上的`after()`** 方法处理请求后，我们可以添加一个简单的日志语句:

```java
@Bean
RouterFunction<ServerResponse> allApplicationRoutes(ProductController pc, ProductService ps) {
    return route()...
      .after((req, res) -> {
          if (res.statusCode() == HttpStatus.OK) {
              LOG.info("Finished processing request " + req.uri()
                  .getPath());
          } else {
              LOG.info("There was an error while processing request" + req.uri());
          }
          return res;
      })          
      .build();
    }
```

## 8.结论

在本教程中，我们从简单介绍定义控制器的函数方法开始。然后，我们比较了 Spring MVC 注释和它们的功能对等物。

接下来，我们实现了一个简单的 web 服务，它返回带有功能控制器的产品列表。

然后，我们继续实现 web 服务控制器的一些常见用例，包括嵌套路由、错误处理、为访问控制添加过滤器，以及管理横切关注点，如日志记录。

和往常一样，示例代码可以在 GitHub 上找到。