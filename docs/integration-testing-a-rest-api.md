# 用 Java 测试 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/integration-testing-a-rest-api>

## 1。概述

在本教程中，我们将重点关注用实时集成测试(使用 JSON 有效负载)测试 REST API 的基本原理和机制。

我们的主要目标是提供测试 API 基本正确性的介绍，我们将使用最新版本的 [GitHub REST API](https://web.archive.org/web/20220727020632/https://docs.github.com/en/rest "GitHub REST API") 作为例子。

对于内部应用程序，这种测试通常在持续集成过程的后期运行，在部署 REST API 后消耗它。

当测试一个 REST 资源时，通常有一些测试应该关注的正交职责:

*   HTTP **响应码**
*   响应中的其他 HTTP **头**
*   **有效载荷** (JSON，XML)

每个测试应该只关注一个责任，并包含一个断言。关注清晰的分离总是有好处的，但是当进行这种黑盒测试时，它甚至更重要，因为一般的趋势是在最开始就编写复杂的测试场景。

集成测试的另一个重要方面是坚持`Single Level of Abstraction Principle;` 我们应该在高层次上编写测试中的逻辑。创建请求、向服务器发送 HTTP 请求、处理 IO 等细节。，不应该内联完成，而是通过实用方法。

## 延伸阅读:

## [春季集成测试](/web/20220727020632/https://www.baeldung.com/integration-testing-in-spring)

A quick guide to writing integration tests for a Spring Web application.[Read more](/web/20220727020632/https://www.baeldung.com/integration-testing-in-spring) →

## [在 Spring Boot 测试](/web/20220727020632/https://www.baeldung.com/spring-boot-testing)

Learn about how the Spring Boot supports testing, to write unit tests efficiently.[Read more](/web/20220727020632/https://www.baeldung.com/spring-boot-testing) →

## [放心指南](/web/20220727020632/https://www.baeldung.com/rest-assured-tutorial)

Explore the basics of REST-assured - a library that simplifies the testing and validation of REST APIs.[Read more](/web/20220727020632/https://www.baeldung.com/rest-assured-tutorial) →

## 2。测试状态代码

```java
@Test
public void givenUserDoesNotExists_whenUserInfoIsRetrieved_then404IsReceived()
  throws ClientProtocolException, IOException {

    // Given
    String name = RandomStringUtils.randomAlphabetic( 8 );
    HttpUriRequest request = new HttpGet( "https://api.github.com/users/" + name );

    // When
    HttpResponse httpResponse = HttpClientBuilder.create().build().execute( request );

    // Then
    assertThat(
      httpResponse.getStatusLine().getStatusCode(),
      equalTo(HttpStatus.SC_NOT_FOUND));
}
```

这是一个相当简单的测试。**它验证了一个基本的快乐路径正在工作**，没有给测试套件增加太多的复杂性。

如果，不管什么原因，它失败了，那么我们不需要查看这个 URL 的任何其他测试，直到我们修复它。

## 3。测试媒体类型

```java
@Test
public void 
givenRequestWithNoAcceptHeader_whenRequestIsExecuted_thenDefaultResponseContentTypeIsJson()
  throws ClientProtocolException, IOException {

   // Given
   String jsonMimeType = "application/json";
   HttpUriRequest request = new HttpGet( "https://api.github.com/users/eugenp" );

   // When
   HttpResponse response = HttpClientBuilder.create().build().execute( request );

   // Then
   String mimeType = ContentType.getOrDefault(response.getEntity()).getMimeType();
   assertEquals( jsonMimeType, mimeType );
}
```

这确保了响应确实包含 JSON 数据。

正如我们所看到的，我们正在进行一系列合乎逻辑的测试。首先是响应状态代码(确保请求正常)，然后是响应的媒体类型。只有在下一个测试中，我们才会看到实际的 JSON 负载。

## 4。测试 JSON 负载

```java
@Test
public void 
  givenUserExists_whenUserInformationIsRetrieved_thenRetrievedResourceIsCorrect()
  throws ClientProtocolException, IOException {

    // Given
    HttpUriRequest request = new HttpGet( "https://api.github.com/users/eugenp" );

    // When
    HttpResponse response = HttpClientBuilder.create().build().execute( request );

    // Then
    GitHubUser resource = RetrieveUtil.retrieveResourceFromResponse(
      response, GitHubUser.class);
    assertThat( "eugenp", Matchers.is( resource.getLogin() ) );
}
```

在这种情况下，GitHub 资源的默认表示是 JSON，但是通常，响应的`Content-Type`头应该和请求的`Accept`头一起测试。客户端通过`Accept`请求特定类型的表示，服务器应该接受。

## 5。测试实用程序

我们将使用 Jackson 2 将原始 JSON 字符串解组成一个类型安全的 Java 实体:

```java
public class GitHubUser {

    private String login;

    // standard getters and setters
}
```

我们只使用一个简单的实用程序来保持测试的整洁、易读和高抽象水平:

```java
public static <T> T retrieveResourceFromResponse(HttpResponse response, Class<T> clazz) 
  throws IOException {

    String jsonFromResponse = EntityUtils.toString(response.getEntity());
    ObjectMapper mapper = new ObjectMapper()
      .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    return mapper.readValue(jsonFromResponse, clazz);
}
```

注意 [Jackson 忽略了 GitHub API 发送给我们的未知属性](/web/20220727020632/https://www.baeldung.com/jackson-deserialize-json-unknown-properties "How to Ignore Uknown Properties")。这仅仅是因为 GitHub 上用户资源的表示变得非常复杂，我们在这里不需要任何这种信息。

## 6。依赖性

这些实用程序和测试使用了以下库，所有这些库都可以在 Maven central 中找到:

*   [http 客户端](https://web.archive.org/web/20220727020632/https://hc.apache.org/httpcomponents-client-ga/index.html "Apache HttpClient")
*   [杰克逊 2](https://web.archive.org/web/20220727020632/https://github.com/FasterXML/jackson)
*   [Hamcrest](https://web.archive.org/web/20220727020632/https://code.google.com/archive/p/hamcrest/ "Hamcrest") (可选)

## 7。结论

这只是完整集成测试套件的一部分。测试的重点是确保 REST API 的基本正确性，而不是进入更复杂的场景。

例如，我们没有涵盖以下内容:API 的可发现性、同一资源的不同表示的消费，等等。

所有这些例子和代码片段的实现都可以在 Github 的[中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)