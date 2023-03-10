# 春天休息分页

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-pagination-in-spring>

## 1。概述

本教程将关注使用 Spring MVC 和 Spring 数据在 REST API 中实现分页。

## 延伸阅读:

## [用弹簧座和角度表分页](/web/20220727020632/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table)

An extensive look at how to implement a simple API with pagination with Spring and how to consume it with AngularJS and UI Grid.[Read more](/web/20220727020632/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table) →

## [JPA 分页](/web/20220727020632/https://www.baeldung.com/jpa-pagination)

Pagination in JPA - how to use JQL and the Criteria API to do pagination correctly.[Read more](/web/20220727020632/https://www.baeldung.com/jpa-pagination) →

## [REST API 可发现性和 HATEOAS](/web/20220727020632/https://www.baeldung.com/restful-web-service-discoverability)

HATEOAS and Discoverability of a REST Service - driven by tests.[Read more](/web/20220727020632/https://www.baeldung.com/restful-web-service-discoverability) →

## 2。页面作为资源 vs 页面作为表示

在 RESTful 架构的上下文中设计分页的第一个问题是，是将**页面视为实际的资源还是仅仅是资源**的表示。

将页面本身视为资源会带来许多问题，比如不再能够唯一地标识调用之间的资源。这一点，再加上在持久层中，页面不是一个真正的实体，而是一个在必要时构造的容器，使得选择变得简单明了；页面是表现形式的一部分。

REST 上下文中分页设计的下一个问题是**在哪里包含分页信息**:

*   在 URI 路径:`/foo/page/1`
*   URI 查询:`/foo?page=1`

记住**页面不是资源**，在 URI 中编码页面信息是不可行的。

我们将使用标准的方法来解决这个问题，即在 URI 查询中对分页信息进行编码。

## 3。控制器

现在来看实现。用于分页的 Spring MVC 控制器很简单:

```java
@GetMapping(params = { "page", "size" })
public List<Foo> findPaginated(@RequestParam("page") int page, 
  @RequestParam("size") int size, UriComponentsBuilder uriBuilder,
  HttpServletResponse response) {
    Page<Foo> resultPage = service.findPaginated(page, size);
    if (page > resultPage.getTotalPages()) {
        throw new MyResourceNotFoundException();
    }
    eventPublisher.publishEvent(new PaginatedResultsRetrievedEvent<Foo>(
      Foo.class, uriBuilder, response, page, resultPage.getTotalPages(), size));

    return resultPage.getContent();
}
```

在这个例子中，我们通过`@RequestParam.`在控制器方法中注入两个查询参数`size`和`page,`

**或者，我们可以使用一个`Pageable`对象，它自动映射`page`、`size`和`sort`参数。**此外，`PagingAndSortingRepository` 实体提供了现成的方法，支持使用`Pageable`作为参数。

我们还注入了 Http 响应和`UriComponentsBuilder`来帮助实现可发现性，我们通过一个定制事件来解耦。如果这不是 API 的目标，我们可以简单地删除自定义事件。

最后注意，本文的重点只是 REST 和 web 层；为了更深入地了解分页的数据访问部分，我们可以[看看这篇关于 Spring 数据分页的文章](https://web.archive.org/web/20220727020632/http://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-part-seven-pagination/ "Data Acceess for Pagination")。

## 4。REST 分页的可发现性

在分页范围内，满足 REST 的 **HATEOAS 约束意味着使 API 的客户端能够基于导航中的当前页面发现`next`和`previous`页面。为此，**我们将使用`Link` HTTP 头，加上“`next,`”“`prev,`”“`first,`”和“`last`”链接关系类型**。**

总之，**可发现性是一个跨领域的问题**，不仅适用于特定的操作，也适用于操作类型。例如，每次创建资源时，客户端都应该可以发现该资源的 URI。因为这个需求与任何资源的创建都相关，所以我们将单独处理它。

正如我们在关注 REST 服务的可发现性的[上一篇文章中所讨论的，我们将使用事件来分离这些问题。在分页的情况下，事件`PaginatedResultsRetrievedEvent,`在控制器层被触发。然后，我们将为该事件实现一个定制监听器的可发现性。](/web/20220727020632/https://www.baeldung.com/rest-api-discoverability-with-spring "How to make REST Discoverable")

简而言之，收听者将检查导航是否允许`next`、`previous`、`first`和`last`页面。如果是，它会**将相关的 URIs 作为“链接”HTTP 头**添加到响应中。

现在我们一步一步来。从控制器传递的`UriComponentsBuilder`只包含基本 URL(主机、端口和上下文路径)。因此，我们必须添加剩余的部分:

```java
void addLinkHeaderOnPagedResourceRetrieval(
 UriComponentsBuilder uriBuilder, HttpServletResponse response,
 Class clazz, int page, int totalPages, int size ){

   String resourceName = clazz.getSimpleName().toString().toLowerCase();
   uriBuilder.path( "/admin/" + resourceName );

    // ...

}
```

接下来，我们将使用一个`StringJoiner`来连接每个链接。我们将使用`uriBuilder`来生成 URIs。让我们看看如何链接到`next` 页面:

```java
StringJoiner linkHeader = new StringJoiner(", ");
if (hasNextPage(page, totalPages)){
    String uriForNextPage = constructNextPageUri(uriBuilder, page, size);
    linkHeader.add(createLinkHeader(uriForNextPage, "next"));
}
```

让我们来看看`constructNextPageUri`方法的逻辑:

```java
String constructNextPageUri(UriComponentsBuilder uriBuilder, int page, int size) {
    return uriBuilder.replaceQueryParam(PAGE, page + 1)
      .replaceQueryParam("size", size)
      .build()
      .encode()
      .toUriString();
}
```

对于我们想要包含的 URIs 的其余部分，我们将进行类似的操作。

最后，我们将输出添加为响应头:

```java
response.addHeader("Link", linkHeader.toString());
```

注意，为了简洁起见，这里只包含了部分代码示例，完整的代码在这里。

## 5。试驾分页

分页和可发现性的主要逻辑都包含在小型的集中集成测试中。正如在[上一篇文章](/web/20220727020632/https://www.baeldung.com/restful-web-service-discoverability "Testing REST Dsicoverability")中，我们将使用[放心库](https://web.archive.org/web/20220727020632/https://github.com/rest-assured/rest-assured "rest-assured official project page")来消费 REST 服务并验证结果。

这些是分页集成测试的几个例子；要获得完整的测试套件，请查看 GitHub 项目(本文末尾的链接):

```java
@Test
public void whenResourcesAreRetrievedPaged_then200IsReceived(){
    Response response = RestAssured.get(paths.getFooURL() + "?page=0&size;=2");

    assertThat(response.getStatusCode(), is(200));
}
@Test
public void whenPageOfResourcesAreRetrievedOutOfBounds_then404IsReceived(){
    String url = getFooURL() + "?page=" + randomNumeric(5) + "&size;=2";
    Response response = RestAssured.get.get(url);

    assertThat(response.getStatusCode(), is(404));
}
@Test
public void givenResourcesExist_whenFirstPageIsRetrieved_thenPageContainsResources(){
   createResource();
   Response response = RestAssured.get(paths.getFooURL() + "?page=0&size;=2");

   assertFalse(response.body().as(List.class).isEmpty());
}
```

## 6。测试分页可发现性

测试分页是否能被客户端发现相对来说比较简单，尽管还有很多内容需要讨论。

**测试将集中在当前页面在导航中的位置，**以及从每个位置可以发现的不同 URIs:

```java
@Test
public void whenFirstPageOfResourcesAreRetrieved_thenSecondPageIsNext(){
   Response response = RestAssured.get(getFooURL()+"?page=0&size;=2");

   String uriToNextPage = extractURIByRel(response.getHeader("Link"), "next");
   assertEquals(getFooURL()+"?page=1&size;=2", uriToNextPage);
}
@Test
public void whenFirstPageOfResourcesAreRetrieved_thenNoPreviousPage(){
   Response response = RestAssured.get(getFooURL()+"?page=0&size;=2");

   String uriToPrevPage = extractURIByRel(response.getHeader("Link"), "prev");
   assertNull(uriToPrevPage );
}
@Test
public void whenSecondPageOfResourcesAreRetrieved_thenFirstPageIsPrevious(){
   Response response = RestAssured.get(getFooURL()+"?page=1&size;=2");

   String uriToPrevPage = extractURIByRel(response.getHeader("Link"), "prev");
   assertEquals(getFooURL()+"?page=0&size;=2", uriToPrevPage);
}
@Test
public void whenLastPageOfResourcesIsRetrieved_thenNoNextPageIsDiscoverable(){
   Response first = RestAssured.get(getFooURL()+"?page=0&size;=2");
   String uriToLastPage = extractURIByRel(first.getHeader("Link"), "last");

   Response response = RestAssured.get(uriToLastPage);

   String uriToNextPage = extractURIByRel(response.getHeader("Link"), "next");
   assertNull(uriToNextPage);
}
```

注意负责通过`rel`关系提取 URIs 的`extractURIByRel,`、[的完整低级代码在这里](https://web.archive.org/web/20220727020632/https://gist.github.com/eugenp/8269915)。

## 7 .**。获取所有资源**

关于分页和可发现性的相同主题，必须做出选择，是允许客户机一次检索系统中的所有资源，还是客户机必须请求分页的资源。

如果确定客户机不能通过一个请求检索所有资源，并且需要分页，那么响应可以有几个选项来获取请求。一种选择是返回 404 ( `Not Found`)并使用`Link`标题使第一页可被发现:

> `Link=<http://localhost:8080/rest/api/admin/foo?page=0&size;=2>; rel=”first”, <http://localhost:8080/rest/api/admin/foo?page=103&size;=2>; rel=”last”`

另一个选项是返回一个重定向，303 `(See Other`，到第一页。更保守的方法是简单地返回客户端 a 405 ( `Method Not Allowed)`)请求 GET。

## 8。使用`Range` HTTP 头进行分页

实现分页的一种相对不同的方式是使用 **HTTP `Range`头、** `Range`、`Content-Range`、`If-Range`、`Accept-Ranges,`和 **HTTP 状态代码、** 206 ( `Partial Content`)、413 ( `Request Entity Too Large`)和 416 ( `Requested Range Not Satisfiable`)。

这种方法的一种观点是，HTTP 范围扩展并不用于分页，它们应该由服务器管理，而不是由应用程序管理。基于 HTTP Range 头扩展实现分页在技术上是可能的，尽管不如本文中讨论的实现那样常见。

## 9.Spring 数据 REST 分页

在 Spring Data 中，如果我们需要从完整的数据集中返回几个结果，我们可以使用任何`Pageable`存储库方法，因为它总是会返回一个`Page.`结果将基于页码、页面大小和排序方向返回。

**[Spring Data REST](/web/20220727020632/https://www.baeldung.com/spring-data-rest-intro) 自动识别`page, size, sort`等 URL 参数。**

要使用任何存储库的分页方法，我们需要扩展`PagingAndSortingRepository:`

```java
public interface SubjectRepository extends PagingAndSortingRepository<Subject, Long>{}
```

如果我们调用`http://localhost:8080/subjects,` ，Spring 会自动用 API 添加`page, size, sort`参数建议:

```java
"_links" : {
  "self" : {
    "href" : "http://localhost:8080/subjects{?page,size,sort}",
    "templated" : true
  }
}
```

默认情况下，页面大小是 20，但是我们可以通过调用类似`http://localhost:8080/subjects?page=10.`的函数来改变它

如果我们想将分页实现到我们自己的定制存储库 API 中，我们需要传递一个额外的`Pageable` 参数，并确保 API 返回一个`Page:`

```java
@RestResource(path = "nameContains")
public Page<Subject> findByNameContaining(@Param("name") String name, Pageable p);
```

每当我们添加一个定制的 API，一个`/search`端点就会被添加到生成的链接中。因此，如果我们调用`http://localhost:8080/subjects/search,` ，我们将看到一个支持分页的端点:

```java
"findByNameContaining" : {
  "href" : "http://localhost:8080/subjects/search/nameContains{?name,page,size,sort}",
  "templated" : true
}
```

如果我们需要从`Page,`返回结果列表，所有实现`PagingAndSortingRepository`的 API 都会返回一个`Page.`。`Page`的`getContent() ` API 提供了作为 Spring Data REST API 结果的记录列表。

## 10.将一个`List`转换成一个`Page`

假设我们有一个`Pageable`对象作为输入，但是我们需要检索的信息包含在一个列表中，而不是一个`PagingAndSortingRepository`。在这些情况下，我们可能需要**将一个`List`转换成一个`Page`** 。

例如，假设我们有一个来自 [SOAP](/web/20220727020632/https://www.baeldung.com/spring-boot-soap-web-service) 服务的结果列表:

```java
List<Foo> list = getListOfFooFromSoapService();
```

我们需要访问发送给我们的`Pageable`对象指定的特定位置的列表。让我们定义开始索引:

```java
int start = (int) pageable.getOffset();
```

和结束索引:

```java
int end = (int) ((start + pageable.getPageSize()) > fooList.size() ? fooList.size()
  : (start + pageable.getPageSize()));
```

有了这两个元素，我们可以创建一个`Page`来获取它们之间的元素列表:

```java
Page<Foo> page 
  = new PageImpl<Foo>(fooList.subList(start, end), pageable, fooList.size());
```

就是这样！我们现在可以返回`page`作为有效结果。

注意，如果我们也想支持对列表进行排序，我们需要在对列表进行子列表之前对列表进行排序。

## 11。结论

本文展示了如何使用 Spring 在 REST API 中实现分页，并讨论了如何设置和测试可发现性。

如果我们想在持久性层次上深入研究分页，我们可以查看一下 [JPA](/web/20220727020632/https://www.baeldung.com/jpa-pagination "JPA Pagination") 或 [Hibernate](/web/20220727020632/https://www.baeldung.com/hibernate-pagination "Hibernate Pagination") 分页教程。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest "Github Project exemplifying REST Pagination")中找到——这是一个基于 Maven 的项目，因此它应该很容易导入和运行。