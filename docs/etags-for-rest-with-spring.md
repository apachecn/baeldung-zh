# 带弹簧的休息标签

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/etags-for-rest-with-spring>

## 1。概述

本文将重点介绍在 Spring 中使用 ETags 的**，REST API 和消费场景与`curl`的集成测试。**

## 延伸阅读:

## [春假单据介绍](/web/20220707143826/https://www.baeldung.com/spring-rest-docs)

This article introduces Spring REST Docs, a test-driven mechanism to generate documentation for RESTful services that is both accurate and readable.[Read more](/web/20220707143826/https://www.baeldung.com/spring-rest-docs) →

## [Spring REST API 的自定义媒体类型](/web/20220707143826/https://www.baeldung.com/spring-rest-custom-media-type)

A quick intro to using a custom media type in a Spring REST API.[Read more](/web/20220707143826/https://www.baeldung.com/spring-rest-custom-media-type) →

## [用弹簧座和角度表分页](/web/20220707143826/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table)

An extensive look at how to implement a simple API with pagination with Spring and how to consume it with AngularJS and UI Grid.[Read more](/web/20220707143826/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table) →

## 2。休息和 ETags

来自官方的关于 ETag 支持的 Spring 文档:

An [ETag](https://web.archive.org/web/20220707143826/https://en.wikipedia.org/wiki/HTTP_ETag) (entity tag) is an HTTP response header returned by an HTTP/1.1 compliant web server used to determine change in content at a given URL.

我们可以将 ETags 用于两件事——缓存和条件请求。 **ETag 值可以被认为是从响应体的字节中计算出来的散列值**。因为服务很可能使用加密散列函数，所以即使对主体进行最小的修改也会极大地改变输出，从而改变 ETag 的值。这只适用于强 Etag——该协议也提供了一个[弱 Etag](https://web.archive.org/web/20220707143826/https://datatracker.ietf.org/doc/html/rfc2616#section-13.3.3 "Weak and Strong Validators") 。

**使用一个`If-*`头将一个标准的 GET 请求转换成一个条件 GET。**与 ETags 一起使用的两个`If-*`头是“ [If-None-Match](https://web.archive.org/web/20220707143826/https://datatracker.ietf.org/doc/html/rfc2616#section-14.26 "If-None-Match in the Specification") ”和“[If-Match](https://web.archive.org/web/20220707143826/https://datatracker.ietf.org/doc/html/rfc2616#section-14.24 "If-Match in the Specification")”——每个头都有自己的语义，本文稍后将对此进行讨论。

## 3。客户端与`curl`服务器通信

我们可以将涉及 ETags 的简单客户端-服务器通信分解为以下步骤:

首先，客户端发出一个 REST API 调用–**响应包含 ETag 头**,它将被存储以备将来使用:

```java
curl -H "Accept: application/json" -i http://localhost:8080/spring-boot-rest/foos/1
```

```java
HTTP/1.1 200 OK
ETag: "f88dd058fe004909615a64f01be66a7"
Content-Type: application/json;charset=UTF-8
Content-Length: 52
```

**对于下一个请求，客户端将在`If-None-Match`请求头中包含上一步的 ETag 值。**如果服务器上的资源没有改变，响应将不包含正文和状态代码`of 304 – Not Modified`:

```java
curl -H "Accept: application/json" -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"'
 -i http://localhost:8080/spring-boot-rest/foos/1
```

```java
HTTP/1.1 304 Not Modified
ETag: "f88dd058fe004909615a64f01be66a7"
```

现在，在再次检索资源之前，让我们通过执行更新来更改它:

```java
curl -H "Content-Type: application/json" -i 
  -X PUT --data '{ "id":1, "name":"Transformers2"}' 
    http://localhost:8080/spring-boot-rest/foos/1
```

```java
HTTP/1.1 200 OK
ETag: "d41d8cd98f00b204e9800998ecf8427e" 
Content-Length: 0
```

最后，我们再次发出检索 Foo 的最后一个请求。请记住，自从我们上次请求它以来，我们已经对它进行了更新，所以以前的 ETag 值应该不再有效。该响应将包含新的数据和新的 ETag，其同样可以被存储以供进一步使用:

```java
curl -H "Accept: application/json" -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"' -i 
  http://localhost:8080/spring-boot-rest/foos/1
```

```java
HTTP/1.1 200 OK
ETag: "03cb37ca667706c68c0aad4cb04c3a211"
Content-Type: application/json;charset=UTF-8
Content-Length: 56
```

这就是你要的——野外的电子标签，节省带宽。

## 4。弹簧中的 ETag 支架

关于 Spring 支持:在 Spring 中使用 ETag 非常容易设置，并且对应用程序完全透明。**我们可以通过在`web.xml`中添加一个简单的`Filter`** 来启用支持:

```java
<filter>
   <filter-name>etagFilter</filter-name>
   <filter-class>org.springframework.web.filter.ShallowEtagHeaderFilter</filter-class>
</filter>
<filter-mapping>
   <filter-name>etagFilter</filter-name>
   <url-pattern>/foos/*</url-pattern>
</filter-mapping>
```

我们将过滤器映射到与 RESTful API 本身相同的 URI 模式上。过滤器本身是自 Spring 3.0 以来 ETag 功能的标准实现。

**实现很简单**–应用程序根据响应计算 ETag，这将节省带宽，但不会影响服务器性能。

因此，受益于 ETag 支持的请求仍将作为标准请求来处理，消耗它通常会消耗的任何资源(数据库连接等),并且只有在它的响应返回到客户端之前，ETag 支持才会起作用。

此时，ETag 将从响应体中计算出来，并在资源本身上设置；同样，如果在请求上设置了`If-None-Match`头，它也会被处理。

ETag 机制的更深层次的实现可能会提供更大的好处——比如从缓存中处理一些请求，而根本不需要执行计算——但是实现肯定不会像这里描述的浅层方法那样简单，也不像这里描述的浅层方法那样可插拔。

### 4.1.基于 Java 的配置

让我们通过在我们的 Spring 上下文中**声明一个`ShallowEtagHeaderFilter` bean 来看看基于 Java 的配置会是什么样子:**

```java
@Bean
public ShallowEtagHeaderFilter shallowEtagHeaderFilter() {
    return new ShallowEtagHeaderFilter();
}
```

请记住，如果我们需要提供进一步的过滤器配置，我们可以声明一个`FilterRegistrationBean`实例:

```java
@Bean
public FilterRegistrationBean<ShallowEtagHeaderFilter> shallowEtagHeaderFilter() {
    FilterRegistrationBean<ShallowEtagHeaderFilter> filterRegistrationBean
      = new FilterRegistrationBean<>( new ShallowEtagHeaderFilter());
    filterRegistrationBean.addUrlPatterns("/foos/*");
    filterRegistrationBean.setName("etagFilter");
    return filterRegistrationBean;
}
```

最后，如果我们不使用 Spring Boot，我们可以使用`AbstractAnnotationConfigDispatcherServletInitializer`的`getServletFilters `方法设置过滤器。

### 4.2.使用响应实体的`eTag()`方法

这个方法是在 Spring framework 4.1 中引入的，**我们可以用它来控制单个端点检索的 ETag 值**。

例如，假设我们使用版本化的实体作为乐观锁定机制来访问我们的数据库信息。

我们可以使用版本本身作为 ETag 来指示实体是否已经被修改:

```java
@GetMapping(value = "/{id}/custom-etag")
public ResponseEntity<Foo>
  findByIdWithCustomEtag(@PathVariable("id") final Long id) {

    // ...Foo foo = ...

    return ResponseEntity.ok()
      .eTag(Long.toString(foo.getVersion()))
      .body(foo);
}
```

如果请求的条件头与缓存数据匹配，服务将检索相应的`304-Not Modified`状态。

## 5。测试标签

让我们从简单的开始—**我们需要验证检索单个资源的简单请求的响应实际上会返回“`ETag”`头:**

```java
@Test
public void givenResourceExists_whenRetrievingResource_thenEtagIsAlsoReturned() {
    // Given
    String uriOfResource = createAsUri();

    // When
    Response findOneResponse = RestAssured.given().
      header("Accept", "application/json").get(uriOfResource);

    // Then
    assertNotNull(findOneResponse.getHeader("ETag"));
}
```

**接下来**，**我们验证一下 ETag 行为的快乐路径。**如果从服务器检索`Resource`的请求使用正确的`ETag`值，则服务器不检索资源:

```java
@Test
public void givenResourceWasRetrieved_whenRetrievingAgainWithEtag_thenNotModifiedReturned() {
    // Given
    String uriOfResource = createAsUri();
    Response findOneResponse = RestAssured.given().
      header("Accept", "application/json").get(uriOfResource);
    String etagValue = findOneResponse.getHeader(HttpHeaders.ETAG);

    // When
    Response secondFindOneResponse= RestAssured.given().
      header("Accept", "application/json").headers("If-None-Match", etagValue)
      .get(uriOfResource);

    // Then
    assertTrue(secondFindOneResponse.getStatusCode() == 304);
}
```

循序渐进:

*   我们创建并检索一个资源， `storing``ETag`值
*   发送一个新的检索请求，这次用“`If-None-Match`”头指定先前存储的`ETag`值
*   在第二次请求时，服务器简单地返回一个`304 Not Modified`，因为资源本身在两次检索操作之间确实没有被修改

**最后，我们验证资源在第一次和第二次检索请求之间改变的情况:**

```java
@Test
public void 
  givenResourceWasRetrievedThenModified_whenRetrievingAgainWithEtag_thenResourceIsReturned() {
    // Given
    String uriOfResource = createAsUri();
    Response findOneResponse = RestAssured.given().
      header("Accept", "application/json").get(uriOfResource);
    String etagValue = findOneResponse.getHeader(HttpHeaders.ETAG);

    existingResource.setName(randomAlphabetic(6));
    update(existingResource);

    // When
    Response secondFindOneResponse= RestAssured.given().
      header("Accept", "application/json").headers("If-None-Match", etagValue)
      .get(uriOfResource);

    // Then
    assertTrue(secondFindOneResponse.getStatusCode() == 200);
}
```

循序渐进:

*   我们首先创建并检索一个`Resource`，并存储`ETag`值以备将来使用
*   然后我们更新同样的`Resource`
*   发送一个新的 GET 请求，这次用“`If-None-Match`”头指定我们先前存储的`ETag`
*   在第二次请求时，服务器将返回一个`200 OK`和完整的资源，因为`ETag`值不再正确，因为我们同时更新了资源

最后，最后一个测试是**对`If-Match` HTTP 头:**的支持，这个测试不会起作用，因为这个功能[还没有在 Spring](https://web.archive.org/web/20220707143826/https://jira.springsource.org/browse/SPR-10164 " ShallowEtagHeaderFilter should deal with the If-Match HTTP Header ") 中实现

```java
@Test
public void givenResourceExists_whenRetrievedWithIfMatchIncorrectEtag_then412IsReceived() {
    // Given
    T existingResource = getApi().create(createNewEntity());

    // When
    String uriOfResource = baseUri + "/" + existingResource.getId();
    Response findOneResponse = RestAssured.given().header("Accept", "application/json").
      headers("If-Match", randomAlphabetic(8)).get(uriOfResource);

    // Then
    assertTrue(findOneResponse.getStatusCode() == 412);
}
```

循序渐进:

*   我们创造了一种资源
*   然后使用指定了不正确的`ETag`值的“`If-Match`”头检索它——这是一个有条件的 GET 请求
*   服务器应该返回一个`412 Precondition Failed`

## 6。ETags 很大

我们只在读操作中使用 ETags。一个 [RFC 存在](https://web.archive.org/web/20220707143826/http://greenbytes.de/tech/webdav/draft-reschke-http-etag-on-write-latest.html#rfc.section.1.1 "The Hypertext Transfer Protocol (HTTP) Entity Tag ("ETag") Response Header in Write Operations")试图阐明实现应该如何处理写操作上的 ETags 这不是标准的，但是是一个有趣的阅读。

当然，ETag 机制还有其他可能的用途，比如用于乐观锁定机制以及处理与[相关的“更新丢失问题”](https://web.archive.org/web/20220707143826/https://www.w3.org/1999/04/Editing/ "Detecting the Lost Update Problem Using Unreserved Checkout")。

使用 ETags 时，还有几个已知的[潜在陷阱和警告](https://web.archive.org/web/20220707143826/https://www.mnot.net/blog/2007/08/07/etags "ETags, ETags, ETags")需要注意。

## 7。结论

本文只是触及了 Spring 和 ETags 的一些皮毛。

对于支持 ETag 的 RESTful 服务的完整实现，以及验证 ETag 行为的集成测试，请查看 GitHub 项目。