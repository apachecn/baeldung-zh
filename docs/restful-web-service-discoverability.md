# REST API 可发现性和 HATEOAS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/restful-web-service-discoverability>

## 1。概述

本文将关注 REST API 的**可发现性，HATEOAS** 和测试驱动的实际场景。

## 2。为什么要让 API 可被发现

API 的可发现性是一个没有得到足够重视的话题。因此，很少有 API 能做到这一点。如果做得正确，它还可以使 API 不仅是 RESTful 和可用的，而且是优雅的。

为了理解可发现性，我们需要将超媒体理解为应用程序状态(HATEOAS)约束的引擎。REST API 的这个约束是关于超媒体(实际上是超文本)资源上的动作/转换的完全可发现性，作为应用状态的唯一驱动。

如果交互是由 API 通过对话本身驱动的，具体来说是通过超文本，那么就没有文档。这将迫使客户做出实际上超出 API 上下文的假设。

总之，**服务器应该有足够的描述性来指导客户端如何仅通过超文本使用 API** 。在 HTTP 对话的情况下，我们可以通过`Link`头来实现这一点。

## 3。可发现性场景(由测试驱动)

那么，REST 服务可被发现意味着什么呢？

在本节中，我们将使用 Junit、[放心](https://web.archive.org/web/20221012100323/https://github.com/rest-assured/rest-assured "rest-assured - Java DSL for easy testing of REST services")和 [Hamcrest](https://web.archive.org/web/20221012100323/https://code.google.com/archive/p/hamcrest/ "Hamcrest") 来测试可发现性的个体特征。由于[REST 服务之前已经得到了](/web/20221012100323/https://www.baeldung.com/securing-a-restful-web-service-with-spring-security "Securing a REST Web Service with Spring Security 3")的保护，每次测试在使用 API 之前首先需要[认证](https://web.archive.org/web/20221012100323/https://gist.github.com/1341570 "Test Utilities for Authentication")。

### 3.1。发现有效的 HTTP 方法

当一个 REST 服务被一个无效的 HTTP 方法使用时，响应应该是一个不允许的 405 方法。

API 还应该帮助客户端发现特定资源所允许的有效 HTTP 方法。为此，**我们可以在响应中使用`Allow` HTTP 头:**

```java
@Test
public void
  whenInvalidPOSTIsSentToValidURIOfResource_thenAllowHeaderListsTheAllowedActions(){
    // Given
    String uriOfExistingResource = restTemplate.createResource();

    // When
    Response res = givenAuth().post(uriOfExistingResource);

    // Then
    String allowHeader = res.getHeader(HttpHeaders.ALLOW);
    assertThat( allowHeader, AnyOf.anyOf(
      containsString("GET"), containsString("PUT"), containsString("DELETE") ) );
}
```

### 3.2。发现新创建资源的 URI

**创建新资源的操作应该总是在响应中包含新创建资源的 URI。**为此，我们可以使用`Location` HTTP 头。

现在，如果客户端在 URI 上执行 GET 操作，资源应该是可用的:

```java
@Test
public void whenResourceIsCreated_thenUriOfTheNewlyCreatedResourceIsDiscoverable() {
    // When
    Foo newResource = new Foo(randomAlphabetic(6));
    Response createResp = givenAuth().contentType("application/json")
      .body(unpersistedResource).post(getFooURL());
    String uriOfNewResource= createResp.getHeader(HttpHeaders.LOCATION);

    // Then
    Response response = givenAuth().header(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
      .get(uriOfNewResource);

    Foo resourceFromServer = response.body().as(Foo.class);
    assertThat(newResource, equalTo(resourceFromServer));
}
```

测试遵循一个简单的场景:**创建一个新的`Foo`资源，然后使用 HTTP 响应来发现资源现在可用的 using】。然后，它还在 URI 上执行 GET 以检索资源，并将其与原始资源进行比较。这是为了确保它被正确保存。**

### 3.3。发现 URI 以获得该类型的所有资源

**当我们获得任何特定的`Foo`资源时，我们应该能够发现我们下一步可以做什么:我们可以列出所有可用的`Foo`资源。**因此，检索资源的操作应该总是在其响应中包括从哪里获取该类型所有资源的 URI。

为此，我们可以再次使用`Link`标题:

```java
@Test
public void whenResourceIsRetrieved_thenUriToGetAllResourcesIsDiscoverable() {
    // Given
    String uriOfExistingResource = createAsUri();

    // When
    Response getResponse = givenAuth().get(uriOfExistingResource);

    // Then
    String uriToAllResources = HTTPLinkHeaderUtil
      .extractURIByRel(getResponse.getHeader("Link"), "collection");

    Response getAllResponse = givenAuth().get(uriToAllResources);
    assertThat(getAllResponse.getStatusCode(), is(200));
}
```

请注意，`extractURIByRel`-负责通过`rel`关系[提取 URIs 的完整低级代码在此](https://web.archive.org/web/20221012100323/https://gist.github.com/eugenp/8269915)处显示。

这个测试涵盖了 REST 中链接关系的棘手问题:检索所有资源的 URI 使用了`rel=”collection”`语义。

这种类型的链接关系还没有被标准化，但是已经被一些微格式使用，并被提议标准化。非标准链接关系的使用开启了关于 RESTful web 服务中微格式和更丰富语义的讨论。

## 4。其他潜在的可发现 URIs 和微格式

**通过`Link`头**可以潜在地发现其他 URIs，但是现有的链接关系类型只允许这么多，而不需要转移到更丰富的语义标记，例如定义自定义链接关系的[、](https://web.archive.org/web/20221012100323/https://tools.ietf.org/html/rfc5988#section-6.2.1 "Web Links RFC") [Atom 发布协议](https://web.archive.org/web/20221012100323/https://datatracker.ietf.org/doc/html/rfc5023 "The Atom Publishing Protocol")或[微格式](https://web.archive.org/web/20221012100323/https://en.wikipedia.org/wiki/Microformat "Wikipedia on Microformats")，这将是另一篇文章的主题。

例如，在对特定资源执行`GET`操作时，客户端应该能够发现 URI 来创建新资源。不幸的是，没有到模型`create`语义的链接关系。

幸运的是，这是一个标准的实践，用于创建的 URI 与用于获取该类型的所有资源的 URI 是相同的，唯一的区别是 POST HTTP 方法。

## 5。结论

我们已经看到了**REST API 是如何完全从根目录发现的，并且没有任何先验知识**——这意味着客户端能够通过在根目录上执行 GET 来导航它。接下来，所有的状态变化都是由客户端使用 REST API 在表示中提供的可用和可发现的转换来驱动的(因此称为`Representational State Transfer`)。

本文涵盖了 REST web 服务上下文中可发现性的一些特征，讨论了 HTTP 方法发现、创建和获取之间的关系、发现 URI 以获取所有资源等。

GitHub 上的[提供了所有这些例子和代码片段的实现。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221012100323/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest "Github Project exemplifying the Discoverability tests")