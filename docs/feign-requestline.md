# 带有假装客户端的请求行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/feign-requestline>

## 1。概述

在本教程中，我们将演示如何在[假装](/web/20220811173852/https://www.baeldung.com/spring-cloud-openfeign)客户端中使用 [`@RequestLine`](https://web.archive.org/web/20220811173852/https://github.com/OpenFeign/feign/blob/master/core/src/main/java/feign/RequestLine.java) 注释。@ `RequestLine`是一个模板，用于定义连接 RESTful web 服务的 URI 和查询参数。

## 2。Maven 依赖关系

首先，让我们创建一个 [Spring Boot](/web/20220811173852/https://www.baeldung.com/spring-boot) web 项目，并将 [`spring-cloud-starter-openfeign`](https://web.archive.org/web/20220811173852/https://search.maven.org/search?q=a:spring-cloud-starter-openfeign) 或 [`feign-core`](https://web.archive.org/web/20220811173852/https://search.maven.org/artifact/io.github.openfeign/feign-core) 依赖项包含到我们的`pom.xml`文件中。`spring-cloud-starter-openfeign`中包含`feign-core`依赖关系；

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>3.1.2</version>
</dependency>
```

或者

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-core</artifactId>
    <version>11.8</version>
</dependency>
```

## 3。`@RequestLine`在中**佯托**

`@RequestLine` Feign 注释指定 HTTP 动词、路径和请求参数作为 Feign 客户机中的参数。使用`@Param`注释指定路径和请求参数。

通常在 Spring Boot 应用程序中，我们会使用`@FeignClient`，但是如果我们不想使用`spring-cloud-starter-openfeign` 依赖关系，我们也可以使用`@RequestLine` 。如果我们将`@RequestLine`和`@FeignClient`一起使用，那么使用这个依赖关系将会给我们一个`IllegalStateException`。

`@FeignClient`注释的`String`值是一个任意的名称，用于创建 Spring Cloud LoadBalancer 客户端。我们可能会根据需要额外指定一个 URL 和其他参数。

让我们创建一个使用`@RequestLine`的界面:

```java
public interface EmployeeClient {
    @RequestLine("GET /empployee/{id}?active={isActive}")
    @Headers("Content-Type: application/json")
    Employee getEmployee(@Param long id, @Param boolean isActive);
}
```

我们还应该提供`@Headers`，在这里我们定义其余`API`所需的头。

现在，我们将调用这样创建的接口来调用实际的`API`:

```java
EmployeeClient employeeResource = Feign.builder().encoder(new SpringFormEncoder())
  .target(EmployeeClient.class, "http://localhost:8081");
Employee employee = employeeResource.getEmployee(id, true);
```

## 4。结论

在本文中，我们展示了@ `RequestLine`注释是如何以及何时在 Feign Client 中使用的。

按照惯例，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220811173852/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign)