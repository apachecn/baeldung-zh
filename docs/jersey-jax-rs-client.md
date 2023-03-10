# JAX-新泽西的客户

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jersey-jax-rs-client>

## 1。概述

Jersey 是一个用于开发 RESTFul Web 服务的开源框架。它还具有强大的内置客户端功能。

在这个快速教程中，我们将探索使用 [Jersey 2](https://web.archive.org/web/20221129003653/https://jersey.java.net/) 创建 JAX-RS 客户端。

关于使用 Jersey 创建 RESTful Web 服务的讨论，请参考本文。

## 延伸阅读:

## [带运动衫和弹簧的 REST API](/web/20221129003653/https://www.baeldung.com/jersey-rest-api-with-spring)

Building Restful Web Services using Jersey 2 and Spring.[Read more](/web/20221129003653/https://www.baeldung.com/jersey-rest-api-with-spring) →

## [CORS 在 JAX-RS](/web/20221129003653/https://www.baeldung.com/cors-in-jax-rs)

Learn how to implement Cross-Origin Resource Sharing (CORS) mechanism in JAX-RS based applications.[Read more](/web/20221129003653/https://www.baeldung.com/cors-in-jax-rs) →

## [泽西过滤器和拦截器](/web/20221129003653/https://www.baeldung.com/jersey-filters-interceptors)

Take a look at how filters and interceptors work in the Jersey framework.[Read more](/web/20221129003653/https://www.baeldung.com/jersey-filters-interceptors) →

## 2。Maven 依赖关系

让我们首先在`pom.xml`中添加所需的依赖项(针对泽西 JAX-RS 客户端):

```java
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>2.25.1</version>
</dependency>
```

要使用 Jackson 2.x 作为 JSON 提供者:

```java
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.25.1</version>
</dependency>
```

这些依赖关系的最新版本可以在 [jersey-client](https://web.archive.org/web/20221129003653/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.core%22%20AND%20a%3A%22jersey-client%22) 和[jersey-media-JSON-Jackson](https://web.archive.org/web/20221129003653/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.media%22%20AND%20a%3A%22jersey-media-json-jackson%22)找到。

## 3。新泽西的 RESTFul 客户端

我们将开发一个 JAX-RS 客户端来使用我们在这里开发的 JSON 和 XML REST APIs(我们需要确保服务已经部署并且 URL 是可访问的)。

### 3.1。资源表示类

让我们来看看资源表示类:

```java
@XmlRootElement
public class Employee {
    private int id;
    private String firstName;

    // standard getters and setters
}
```

只有在需要 XML 支持时，才需要像`@XmlRootElement`这样的 JAXB 注释。

### 3.2。创建一个`Client` 的实例

我们首先需要的是一个`Client`的实例:

```java
Client client = ClientBuilder.newClient();
```

### 3.3。创造一个`WebTarget`

一旦我们有了`Client`实例，我们就可以使用目标 web 资源的 URI 创建一个`WebTarget`:

```java
WebTarget webTarget 
  = client.target("http://localhost:8082/spring-jersey");
```

使用`WebTarget`，我们可以定义一个特定资源的路径:

```java
WebTarget employeeWebTarget 
  = webTarget.path("resources/employees");
```

### 3.4。构建 HTTP 请求调用

调用生成器实例是通过`WebTarget.request()`方法之一创建的:

```java
Invocation.Builder invocationBuilder 
  = employeeWebTarget.request(MediaType.APPLICATION_JSON);
```

对于 XML 格式，可以使用`MediaType.APPLICATION_XML`。

### 3.5。调用 HTTP 请求

调用 HTTP GET:

```java
Response response 
  = invocationBuilder.get(Employee.class);
```

调用 HTTP POST:

```java
Response response 
  = invocationBuilder
  .post(Entity.entity(employee, MediaType.APPLICATION_JSON);
```

### 3.6。样本剩余客户端

让我们开始编写一个简单的 REST 客户端。`getJsonEmployee()`方法基于雇员`id`检索一个`Employee`对象。由 [REST Web 服务](/web/20221129003653/https://www.baeldung.com/jersey-rest-api-with-spring)返回的 JSON 在返回之前被反序列化为`Employee`对象。

流畅地使用 JAX-RS API 创建 web 目标，调用生成器并调用一个 GET HTTP 请求:

```java
public class RestClient {

    private static final String REST_URI 
      = "http://localhost:8082/spring-jersey/resources/employees";

    private Client client = ClientBuilder.newClient();

    public Employee getJsonEmployee(int id) {
        return client
          .target(REST_URI)
          .path(String.valueOf(id))
          .request(MediaType.APPLICATION_JSON)
          .get(Employee.class);
    }
    //...
}
```

现在让我们为 POST HTTP 请求添加一个方法。`createJsonEmployee()`方法通过调用用于`Employee`创建的 [REST Web 服务](/web/20221129003653/https://www.baeldung.com/jersey-rest-api-with-spring)来创建一个`Employee`。在调用 HTTP POST 方法之前，客户端 API 在内部将`Employee`对象序列化为 JSON:

```java
public Response createJsonEmployee(Employee emp) {
    return client
      .target(REST_URI)
      .request(MediaType.APPLICATION_JSON)
      .post(Entity.entity(emp, MediaType.APPLICATION_JSON));
}
```

### 4。测试客户端

让我们用 JUnit 测试我们的客户机:

```java
public class JerseyClientLiveTest {

    public static final int HTTP_CREATED = 201;
    private RestClient client = new RestClient();

    @Test
    public void givenCorrectObject_whenCorrectJsonRequest_thenResponseCodeCreated() {
        Employee emp = new Employee(6, "Johny");

        Response response = client.createJsonEmployee(emp);

        assertEquals(response.getStatus(), HTTP_CREATED);
    }
}
```

## 5。结论

在本文中，我们介绍了使用 Jersey 2 的 JAX-RS 客户端，并开发了一个简单的 RESTFul Java 客户端。

和往常一样，完整的源代码可以在 Github 项目中获得。