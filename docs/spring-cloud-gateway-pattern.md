# 春云系列——门户模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-pattern>

## 1。概述

[到目前为止](/web/20221117030334/https://www.baeldung.com/spring-cloud-series)，在我们的云应用中，我们已经使用网关模式来支持两个主要特性。

首先，我们将客户与每个服务隔离开来，消除了对跨来源支持的需求。接下来，我们使用 Eureka 实现了服务实例的定位。

在本文中，我们将研究如何使用网关模式通过单个请求从多个服务中检索数据。为此，我们将在我们的网关中引入 Feign 来帮助编写对我们的服务的 API 调用。

要了解如何使用虚拟客户端，请阅读本文。

Spring Cloud 现在也提供了实现这种模式的 [Spring Cloud Gateway](/web/20221117030334/https://www.baeldung.com/spring-cloud-gateway) 项目。

## 2。设置

让我们打开我们的`gateway`服务器的`pom.xml`,并添加对 Feign 的依赖:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

作为参考——我们可以在`Maven Central` ( [春-云-启动器-佯](https://web.archive.org/web/20221117030334/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-feign%22))上找到最新版本。

现在我们已经支持构建一个虚拟客户端，让我们在`GatewayApplication.java`中启用它:

```java
@EnableFeignClients
public class GatewayApplication { ... }
```

现在让我们为图书和评级服务设置虚拟客户端。

## 3。假装客户

### 3.1。预订客户端

让我们创建一个名为`BooksClient.java`的新界面:

```java
@FeignClient("book-service")
public interface BooksClient {

    @RequestMapping(value = "/books/{bookId}", method = RequestMethod.GET)
    Book getBookById(@PathVariable("bookId") Long bookId);
}
```

通过这个接口，我们指示 Spring 创建一个将访问“`/books/{bookId` }”端点的假造客户端。当被调用时，`getBookById`方法将对端点进行 HTTP 调用，并使用`bookId`参数。

为了完成这项工作，我们需要添加一个`Book.java` DTO:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class Book {

    private Long id;
    private String author;
    private String title;
    private List<Rating> ratings;

    // getters and setters
}
```

让我们继续进行`RatingsClient`。

### 3.2。评级客户

让我们创建一个名为`RatingsClient`的接口:

```java
@FeignClient("rating-service")
public interface RatingsClient {

    @RequestMapping(value = "/ratings", method = RequestMethod.GET)
    List<Rating> getRatingsByBookId(
      @RequestParam("bookId") Long bookId, 
      @RequestHeader("Cookie") String session);

}
```

与`BookClient`一样，这里公开的方法将对我们的评级服务进行 rest 调用，并返回一本书的评级列表。

但是，此端点是安全的。为了能够正确地访问这个端点，我们需要将用户的会话传递给请求。

我们使用`@RequestHeader`注释来完成这项工作。这将指示 Feign 将该变量的值写入请求的头部。在我们的例子中，我们正在写入`Cookie`头，因为 Spring Session 将在 cookie 中寻找我们的会话。

在我们的例子中，我们正在写入`Cookie`头，因为 Spring Session 将在 cookie 中寻找我们的会话。

最后，我们来补充一个`Rating.java` DTO:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class Rating {
    private Long id;
    private Long bookId;
    private int stars;
}
```

现在，两个客户端都完成了。让我们把它们派上用场吧！

## 4。组合请求

网关模式的一个常见用例是让端点封装通常被调用的服务。这可以通过减少客户端请求的数量来提高性能。

为此，让我们创建一个控制器，并将其命名为`CombinedController.java`:

```java
@RestController
@RequestMapping("/combined")
public class CombinedController { ... }
```

接下来，让我们连接新创建的虚拟客户端:

```java
private BooksClient booksClient;
private RatingsClient ratingsClient;

@Autowired
public CombinedController(
  BooksClient booksClient, 
  RatingsClient ratingsClient) {

    this.booksClient = booksClient;
    this.ratingsClient = ratingsClient;
}
```

最后，让我们创建一个 GET 请求，该请求将这两个端点结合起来，并返回一本书及其加载的评级:

```java
@GetMapping
public Book getCombinedResponse(
  @RequestParam Long bookId,
  @CookieValue("SESSION") String session) {

    Book book = booksClient.getBookById(bookId);
    List<Rating> ratings = ratingsClient.getRatingsByBookId(bookId, "SESSION="+session);
    book.setRatings(ratings);
    return book;
}
```

注意，我们使用从请求中提取会话值的`@CookieValue`注释来设置会话值。

在那里！我们的网关中有一个组合端点，可以减少客户端和系统之间的网络呼叫！

## 5。测试

让我们确保我们的新端点正在工作。

导航到`LiveTest.java`，让我们为我们的组合端点添加一个测试:

```java
@Test
public void accessCombinedEndpoint() {
    Response response = RestAssured.given()
      .auth()
      .form("user", "password", formConfig)
      .get(ROOT_URI + "/combined?bookId=1");

    assertEquals(HttpStatus.OK.value(), response.getStatusCode());
    assertNotNull(response.getBody());

    Book result = response.as(Book.class);

    assertEquals(new Long(1), result.getId());
    assertNotNull(result.getRatings());
    assertTrue(result.getRatings().size() > 0);
}
```

启动 Redis，然后运行应用程序中的每个服务:`config, discovery, zipkin,` `gateway`、`book`和`rating`服务。

一旦一切就绪，运行新的测试来确认它正在工作。

## 6。结论

我们已经看到了如何将 Feign 集成到我们的网关中来构建一个专门化的端点。我们可以利用这些信息来构建我们需要支持的任何 API。最重要的是，我们看到我们没有被一个只公开单个资源的通用 API 所束缚。

使用网关模式，我们可以根据每个客户的独特需求来设置网关服务。这就产生了解耦，使我们的服务可以根据需要自由发展，保持精简并专注于应用程序的一个领域。

和往常一样，代码片段可以在 GitHub 上找到。