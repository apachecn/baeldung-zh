# 带有 Zuul 代理的弹簧座

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-with-zuul-proxy>

## 1。概述

在本文中，我们将探索前端应用程序和 REST API 之间的**通信，这两个 API 是单独部署的**。

目标是解决浏览器的 CORS 和同源策略限制，并允许 UI 调用 API，即使它们不共享相同的源。

我们将创建两个独立的应用程序——一个 UI 应用程序和一个简单的 REST API，我们将使用 UI 应用程序中的 Zuul 代理来代理对 REST API 的调用。

Zuul 是网飞基于 JVM 的路由器和服务器端负载均衡器。Spring Cloud 与嵌入式 Zuul 代理进行了很好的集成，这就是我们在这里要使用的。

## 延伸阅读:

## [使用 Zuul 和 Eureka 进行负载平衡的示例](/web/20221117030334/https://www.baeldung.com/zuul-load-balancing)

See how load-balancing with Netflix Zuul looks like.[Read more](/web/20221117030334/https://www.baeldung.com/zuul-load-balancing) →

## [使用 Springfox 用 Spring REST API 设置 Swagger 2](/web/20221117030334/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)

Learn how to document a Spring REST API using Swagger 2.[Read more](/web/20221117030334/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api) →

## [春假单据介绍](/web/20221117030334/https://www.baeldung.com/spring-rest-docs)

This article introduces Spring REST Docs, a test-driven mechanism to generate documentation for RESTful services that is both accurate and readable.[Read more](/web/20221117030334/https://www.baeldung.com/spring-rest-docs) →

## 2。Maven 配置

首先，我们需要从 Spring Cloud 添加一个对 zuul 支持的依赖项到我们的 UI 应用程序的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221117030334/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-starter-netflix-zuul)

## 3。Zuul 属性

接下来，我们需要配置 Zuul，因为我们使用的是 Spring Boot，所以我们将在`application.yml`中完成:

```java
zuul:
  routes:
    foos:
      path: /foos/**
      url: http://localhost:8081/spring-zuul-foos-resource/foos
```

请注意:

*   我们正在代理我们的资源服务器`Foos.`
*   来自 UI 的所有以“`/foos/`”开头的请求都将被路由到我们位于`http://loclahost:8081/spring-zuul-foos-resource/foos/`的`Foos`资源服务器

## 4。API

我们的 API 应用程序是一个简单的 Spring Boot 应用程序。

在本文中，我们将考虑在端口 8081 上运行**的服务器中部署的 API。**

让我们首先为将要使用的资源定义基本 DTO:

```java
public class Foo {
    private long id;
    private String name;

    // standard getters and setters
}
```

和一个简单的控制器:

```java
@RestController
public class FooController {

    @GetMapping("/foos/{id}")
    public Foo findById(
      @PathVariable long id, HttpServletRequest req, HttpServletResponse res) {
        return new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));
    }
}
```

## 5。用户界面应用程序

我们的 UI 应用程序也是一个简单的 Spring Boot 应用程序。

在本文中，我们将考虑在端口 8080 上运行**的服务器中部署的 API。**

让我们从主要的`index.html`开始——使用一些角度:

```java
<html>
<body ng-app="myApp" ng-controller="mainCtrl">
<script src="angular.min.js"></script>
<script src="angular-resource.min.js"></script>

<script>
var app = angular.module('myApp', ["ngResource"]);

app.controller('mainCtrl', function($scope,$resource,$http) {
    $scope.foo = {id:0 , name:"sample foo"};
    $scope.foos = $resource("/foos/:fooId",{fooId:'@id'});

    $scope.getFoo = function(){
        $scope.foo = $scope.foos.get({fooId:$scope.foo.id});
    }  
});
</script>

<div>
    <h1>Foo Details</h1>
    <span>{{foo.id}}</span>
    <span>{{foo.name}}</span>
    <a href="#" ng-click="getFoo()">New Foo</a>
</div>
</body>
</html>
```

这里最重要的方面是我们如何使用相对 URL 访问 API **!**

请记住，API 应用程序并没有和 UI 应用程序部署在同一个服务器上，**所以相对 URL 不应该工作**，并且在没有代理的情况下也不会工作。

然而，使用代理，我们通过 Zuul 代理访问`Foo`资源，当然，Zuul 代理被配置为将这些请求路由到实际部署 API 的任何地方。

最后，实际启动的应用程序:

```java
@EnableZuulProxy
@SpringBootApplication
public class UiApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(UiApplication.class, args);
    }
}
```

除了简单的引导注释，请注意，我们还为 Zuul 代理使用了 enable 风格的注释，这非常酷、干净和简洁。

## 6。测试路由

现在，让我们测试我们的 UI 应用程序，如下所示:

```java
@Test
public void whenSendRequestToFooResource_thenOK() {
    Response response = RestAssured.get("http://localhost:8080/foos/1");

    assertEquals(200, response.getStatusCode());
}
```

## 7。自定义 Zuul 过滤器

有多种 Zuul 过滤器可用，我们也可以创建自己的自定义过滤器:

```java
@Component
public class CustomZuulFilter extends ZuulFilter {

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        ctx.addZuulRequestHeader("Test", "TestSample");
        return null;
    }

    @Override
    public boolean shouldFilter() {
       return true;
    }
    // ...
}
```

这个简单的过滤器只是在请求中添加了一个名为“`Test`”的头——但是当然，我们可以根据需要增加请求的复杂程度。

## 8。测试自定义 Zuul 过滤器

最后，让我们测试一下，确保我们的定制过滤器正在工作——首先，我们将在 Foos 资源服务器上修改我们的`FooController`:

```java
@RestController
public class FooController {

    @GetMapping("/foos/{id}")
    public Foo findById(
      @PathVariable long id, HttpServletRequest req, HttpServletResponse res) {
        if (req.getHeader("Test") != null) {
            res.addHeader("Test", req.getHeader("Test"));
        }
        return new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));
    }
}
```

现在，让我们来测试一下:

```java
@Test
public void whenSendRequest_thenHeaderAdded() {
    Response response = RestAssured.get("http://localhost:8080/foos/1");

    assertEquals(200, response.getStatusCode());
    assertEquals("TestSample", response.getHeader("Test"));
}
```

## 9。结论

在这篇文章中，我们主要关注使用 Zuul 将请求从 UI 应用程序路由到 REST API。我们成功地解决了 CORS 和同源策略问题，还成功地定制和扩充了传输中的 HTTP 请求。

本教程的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20221117030334/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-zuul)中找到。